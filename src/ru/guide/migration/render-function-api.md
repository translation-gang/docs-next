---
badges:
  - breaking
---

# Изменён API render-функций <MigrationBadges :badges="$frontmatter.badges" />

## Обзор

Это изменение не затрагивает пользователей, определяющих шаблоны в `<template>`.

Краткое описание изменений:

- Теперь `h` импортируется глобально, а не передаётся в render-функцию аргументом
- Изменены аргументы render-функции для консистентности с обычными и функциональными компонентами
- VNode теперь имеют плоскую структуру входных параметров

Для получения дополнительной информации читайте дальше!

## Аргумент render-функции

### Синтаксис в 2.x

В версии 2.x, функция `render` автоматически получает аргументом функцию `h` (которая является общепринятым сокращением для `createElement`):

```js{3}
// Пример render-функции во Vue 2
export default {
  render(h) {
    return h('div')
  }
}
```

### Синтаксис в 3.x

В версии 3.x, `h` импортируется глобально, а не передаётся аргументом автоматически.

```js{2,5}
// Пример render-функции во Vue 3
import { h } from 'vue'

export default {
  render() {
    return h('div')
  }
}
```

## Изменения сигнатуры render-функции

### Синтаксис в 2.x

В версии 2.x, функция `render` автоматические получала такие аргументы, как `h`.

```js
// Пример render-функции во Vue 2
export default {
  render(h) {
    return h('div')
  }
}
```

### Синтаксис в 3.x

В версии 3.x, поскольку функция `render` больше не получает никаких аргументов, то в основном будет использоваться внутри функции `setup()`. В этом случае, дополнительное преимущество в том, что есть возможность получения реактивного состояния и функций, объявленных в области видимости, а также к аргументам переданным в `setup()`.

```js
import { h, reactive } from 'vue'

export default {
  setup(props, { slots, attrs, emit }) {
    const state = reactive({
      count: 0
    })

    function increment() {
      state.count++
    }

    // возвращаем render-функцию
    return () =>
      h(
        'div',
        {
          onClick: increment
        },
        state.count
      )
  }
}
```

Подробнее о работе `setup()` можно узнать в [руководстве Composition API](../composition-api-introduction.md).

## Формат входных параметров VNode

### Синтаксис в 2.x

В версии 2.x, `domProps` содержал вложенный список внутри входных параметров VNode:

```js
// Синтаксис 2.x
{
  staticClass: 'button',
  class: {'is-outlined': isOutlined },
  staticStyle: { color: '#34495E' },
  style: { backgroundColor: buttonColor },
  attrs: { id: 'submit' },
  domProps: { innerHTML: '' },
  on: { click: submitForm },
  key: 'submit-button'
}
```

### Синтаксис в 3.x

В версии 3.x, вся структура входных параметров VNode имеет плоскую структуру. Пример выше в таком случае выглядел бы сейчас так:

```js
// Синтаксис в 3.x
{
  class: ['button', { 'is-outlined': isOutlined }],
  style: [{ color: '#34495E' }, { backgroundColor: buttonColor }],
  id: 'submit',
  innerHTML: '',
  onClick: submitForm,
  key: 'submit-button'
}
```

## Зарегистрированные компоненты

### Синтаксис в 2.x

В версии 2.x, когда компонент зарегистрирован, render-функция будет работать если передать имя компонента строкой первым аргументом:

```js{2,17}
// Синтаксис 2.x
Vue.component('button-counter', {
  data() {
    return {
      count: 0
    }
  }
  template: `
    <button @click="count++">
      Счётчик кликов — {{ count }}.
    </button>
  `
})

export default {
  render(h) {
    return h('button-counter')
  }
}
```

### Синтаксис в 3.x

В версии 3.x, когда VNode не имеют контекста, уже не получится использовать строковый ID для неявного указания зарегистрированных компонентов. Вместо этого потребуется использовать импортированный метод `resolveComponent` и передавать уже компонент:

```js{2,6,7}
// Синтаксис в 3.x
import { h, resolveComponent } from 'vue'

export default {
  setup() {
    const ButtonCounter = resolveComponent('button-counter')
    return () => h(ButtonCounter)
  }
}
```

Для получения дополнительной информации см. [RFC по изменению API render-функций](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0008-render-function-api-change.md#context-free-vnodes).

## Стратегия миграции

[Флаг сборки для миграции: `RENDER_FUNCTION`](migration-build.md#конфигурация-совместимости)

### Разработчикам библиотек

Необходимость глобально импортировать `h` означает, что любая библиотека, содержащая компоненты Vue, где-нибудь будет использовать `import { h } from 'vue'`. Это создаст небольшие накладные расходы, потому что разработчикам библиотек потребуется корректно настроить экстернализацию Vue при конфигурации сборки:

- Vue не должен добавляться в сборку библиотеки
- В сборках модулей импорт должен оставаться как есть и обрабатываться конечным сборщиком пользователя
- В сборках UMD / для браузера сначала должно быть обращение к глобальному `Vue.h`, а при недоступности использовать вызов `require`

## Дальнейшие шаги

Подробную документацию см. в [руководстве по render-функциям](../render-function.md)!
