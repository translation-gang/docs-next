# Функция `setup`

> Этот раздел использует синтаксис [однофайловых компонентов](single-file-component.md) для примеров кода

> Подразумевается, что уже изучили и разобрались с разделами [Введение в Composition API](composition-api-introduction.md) и [Основы реактивности](reactivity-fundamentals.md). Если нет — прочитайте их сначала.

## Аргументы

При использовании функции `setup` можно передавать два аргумента:

1. `props` — входные параметры
2. `context` — контекст

Давайте разберёмся с тем, как можно использовать каждый из них.

### Входные параметры

Первым аргументом `setup` передаются входные параметры `props`. Как и следует ожидать в обычном компоненте, `props` внутри функции `setup` реактивны и будут обновляться при передаче новых значений.

```js
// MyBook.vue

export default {
  props: {
    title: String
  },
  setup(props) {
    console.log(props.title)
  }
}
```

:::warning ВНИМАНИЕ
Поскольку `props` реактивны, то **использовать деструктуризацию ES6 нельзя**, потому что это уберёт реактивность со входных параметров.
:::

Если необходимо использовать деструктуризацию входных параметров, это можно сделать используя [toRefs](reactivity-fundamentals.md#destructuring-reactive-state) внутри функции `setup`:

```js
// MyBook.vue

import { toRefs } from 'vue'

setup(props) {
  const { title } = toRefs(props)

  console.log(title.value)
}
```

Если `title` является необязательным входным параметром — он может отсутствовать в `props`. В таком случае  `toRefs` не создаст ссылку для `title`. Вместо этого потребуется использовать `toRef`:

```js
// MyBook.vue

import { toRef } from 'vue'

setup(props) {
  const title = toRef(props, 'title')

  console.log(title.value)
}
```

### Контекст

Вторым аргументом, передаваемым в функцию `setup`, будет контекст `context`. Это обычный объект JavaScript, который предоставляет доступ к трём свойствам компонента:

```js
// MyBook.vue

export default {
  setup(props, context) {
    // Атрибуты (нереактивный объект)
    console.log(context.attrs)

    // Слоты (нереактивный объект)
    console.log(context.slots)

    // Генерация событий (метод)
    console.log(context.emit)
  }
}
```

Поскольку объект `context` будет обычным объектом JavaScript, т.е. он нереактивный, а значит можно спокойно использовать деструктуризацию ES6 для `context`.

```js
// MyBook.vue
export default {
  setup(props, { attrs, slots, emit }) {
    ...
  }
}
```

Свойства `attrs` и `slots` — объекты с состоянием, которые будут всегда обновляться при обновлении самого компонента. Это значит, что следует избегать деструктуризации для них и всегда ссылаться на свойства как `attrs.x` или `slots.x`. Также обратите внимание, что в отличие от `props`, свойства `attrs` и `slots` **НЕ РЕАКТИВНЫ**. Если необходимо применить побочные эффекты, основанные на изменениях `attrs` или `slots`, то следует делать это внутри жизненного цикла `onUpdated`.

## Доступ к свойствам компонента

При выполнении `setup` экземпляр компонента ещё не будет создан. Поэтому возможно получить доступ только к следующим свойствам:

- `props`
- `attrs`
- `slots`
- `emit`

Другими словами, **не будет доступа** к следующим опциям компонента:

- `data`
- `computed`
- `methods`

## Использование в шаблонах

Если функция `setup` возвращает объект, то его свойства будут также доступны в шаблоне компонента, как и свойства `props`, переданные в `setup`:

```vue
<!-- MyBook.vue -->
<template>
  <div>{{ collectionName }}: {{ readersNumber }} {{ book.title }}</div>
</template>

<script>
  import { ref, reactive } from 'vue'

  export default {
    props: {
      collectionName: String
    },
    setup(props) {
      const readersNumber = ref(0)
      const book = reactive({ title: 'Vue 3 Guide' })

      // всё объявленное будет доступно в шаблоне
      return {
        readersNumber,
        book
      }
    }
  }
</script>
```

Обратите внимание, [refs](../api/refs-api.md#ref) возвращаемые из `setup` будут [автоматически неглубоко разворачиваться](reactivity-fundamentals.md#ref-unwrapping) при обращениях в шаблоне, поэтому в шаблонах указывать `.value` не потребуется.

## Использование в render-функциях

Функция `setup` также может возвращать render-функцию, которая может напрямую использовать реактивное состояние, объявленное в той же области видимости:

```js
// MyBook.vue

import { h, ref, reactive } from 'vue'

export default {
  setup() {
    const readersNumber = ref(0)
    const book = reactive({ title: 'Vue 3 Guide' })
    // Обратите внимание, здесь потребуется явно объявлять значение ref
    return () => h('div', [readersNumber.value, book.title])
  }
}
```

## Использование `this`

**Внутри `setup()` использование `this` не будет ссылкой на текущий активный экземпляр.** Поскольку `setup()` вызывается до разрешения других опций компонента, то `this` внутри `setup()` будет вести себя несколько иначе, чем `this` в других опциях. Это может привести к путанице при использовании `setup()` вместе с другими Options API.
