# Основы реактивности

## Объявление реактивного состояния

Реактивное состояния из объекта JavaScript создаётся с помощью метода `reactive`:

```js
import { reactive } from 'vue'

// реактивное состояние
const state = reactive({
  count: 0
})
```

Метод `reactive` — эквивалент `Vue.observable()` API во Vue 2.x, переименованный чтобы избежать путаницы с observables RxJS. Здесь возвращаемое состояние будет являться реактивным объектом. Реактивное преобразование «глубокое» — оно будет влиять на все вложенные свойства переданного объекта.

Основной сценарий использования реактивного состояния во Vue — использование во время отрисовки. Благодаря отслеживанию зависимостей, представление автоматически будет обновляться при изменениях реактивного состояния.

Это сама суть системы реактивности Vue. Когда в компоненте возвращается объект из `data()`, то под капотом он уже становится реактивным с помощью `reactive()`. Шаблон компилируется в [render-функцию](render-function.md), которая использует эти реактивные свойства.

Подробнее о методе `reactive` можно узнать в разделе [Основы API реактивности](../api/basic-reactivity.md).

## Создание автономных ссылок на реактивные значения

Представьте случай, когда есть отдельное примитивное значение (например, строка) и необходимо сделать её реактивной. Конечно, можно сделать объект с одним свойством, значением которого будет эта строка, а затем передать его в `reactive`. Для этого у Vue уже есть метод, который сделает то же самое — `ref`:

```js
import { ref } from 'vue'

const count = ref(0)
```

`ref` вернёт реактивный объект, который можно изменять и который служит реактивной ссылкой (**ref** от слова reference) для внутреннего значения, которое он хранит — откуда и происходит его имя. Этот объект содержит только одно свойство с именем `value`:

```js
import { ref } from 'vue'

const count = ref(0)
console.log(count.value) // 0

count.value++
console.log(count.value) // 1
```

### Разворачивание ref-ссылок

Когда ref-ссылка возвращается в качестве свойства для контекста отрисовки (объект, возвращаемый из [setup()](composition-api-setup.md)) и доступ к свойству осуществляется в шаблоне, то ссылка будет  автоматически разворачиваться во внутреннее значение. Только для вложенных ref-ссылок потребуется указывать `.value` в шаблоне:

```vue
<template>
  <div>
    <span>{{ count }}</span>
    <button @click="count ++">Увеличить счётчик</button>
    <button @click="nested.count.value ++">Увеличить вложенный счётчик</button>
  </div>
</template>

<script>
  import { ref } from 'vue'
  export default {
    setup() {
      const count = ref(0)
      return {
        count,

        nested: {
          count
        }
      }
    }
  }
</script>
```

:::tip Совет
Если доступ к реальному экземпляру объекта не требуется, то можно обернуть его в метод `reactive`:

```js
nested: reactive({
  count
})
```
:::

### Доступ в реактивных объектах

При доступе к `ref` или мутировании как свойства реактивного объекта, автоматически она будет разворачиваться во внутреннее значение и вести себя как обычное свойство:

```js
const count = ref(0)
const state = reactive({
  count
})

console.log(state.count) // 0

state.count = 1
console.log(count.value) // 1
```

Если новая ссылка присваивается к свойству, связанному с существующей ссылкой, то она просто заменит собой старую ссылку:

```js
const otherCount = ref(2)

state.count = otherCount
console.log(state.count) // 2
console.log(count.value) // 1
```

Разворачивание ссылки происходит только в том случае, если она вложена в реактивный `Object`. Никакого разворачивания не будет выполняться, когда ссылка предоставляет доступ к `Array` или нативной коллекции, например [`Map`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map):

```js
const books = reactive([ref('Руководство по Vue 3')])
// здесь потребуется указывать .value
console.log(books[0].value)

const map = reactive(new Map([['count', ref(0)]]))
// здесь потребуется указывать .value
console.log(map.get('count').value)
```

## Деструктурирование реактивного состояния

При необходимости использовать лишь несколько свойств из большого реактивного объекта, может возникнуть соблазн использовать [деструктурирование из ES6](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) для получения нужных свойств:

```js
import { reactive } from 'vue'

const book = reactive({
  author: 'Vue Team',
  year: '2020',
  title: 'Vue 3 Guide',
  description: 'You are reading this book right now ;)',
  price: 'free'
})

let { author, title } = book
```

К сожалению, при таком деструктурировании реактивность обоих свойств будет потеряна. Для таких случаев необходимо сначала преобразовать реактивный объект в набор ссылок. Такие ссылки сохранят реактивную связь с исходным объектом:

```js
import { reactive, toRefs } from 'vue'

const book = reactive({
  author: 'Vue Team',
  year: '2020',
  title: 'Vue 3 Guide',
  description: 'You are reading this book right now ;)',
  price: 'free'
})

let { author, title } = toRefs(book)

title.value = 'Vue 3 Detailed Guide' // нужно .value потому что title теперь ссылка
console.log(book.title) // 'Vue 3 Detailed Guide'
```

Подробнее о `refs` можно узнать в разделе [API реактивных ссылок](../api/refs-api.md#ref).

## Предотвращение изменений реактивных объектов с помощью `readonly`

Иногда требуется отслеживать изменения реактивного объекта (`ref` или `reactive`), но необходимо запретить его изменения из определённого места приложения. Например, если используем [внедряемый через provide](component-provide-inject.md) реактивный объект и хотим предотвратить попытки изменений там, где он будет внедряться. Для таких случаев можно создать прокси «только для чтения» исходного объекта:

```js
import { reactive, readonly } from 'vue'

const original = reactive({ count: 0 })

const copy = readonly(original)

// изменение оригинала вызовет наблюдатели, полагающиеся на копию
original.count++

// изменение копии приведёт к неудаче и отображению предупреждения
copy.count++ // warning: "Set operation on key 'count' failed: target is readonly."
```
