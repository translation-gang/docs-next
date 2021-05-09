# Основное API реактивности

> В этом разделе в примерах кода используется синтаксис [однофайловых компонентов](../guide/single-file-component.md)

## `reactive`

Возвращает реактивную копию объекта.

```js
const obj = reactive({ count: 0 })
```

Реактивное преобразование «глубокое» — оно будет затрагивать все вложенные свойства. В реализации, основанной на [ES2015 Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy), возвращаемый прокси **НЕ БУДЕТ РАВЕН** оригинальному объекту. Рекомендуется работать исключительно с реактивным прокси и не полагаться на оригинальный объект.

**Типизация:**

```ts
function reactive<T extends object>(target: T): UnwrapNestedRefs<T>
```

:::tip Примечание
`reactive` разворачивает вложенные [ref-ссылки](refs-api.md#ref) и при этом сохраняет их реактивность

```ts
const count = ref(1)
const obj = reactive({ count })

// ссылка будет развёрнута
console.log(obj.count === count.value) // true

// изменения будут обновлять `obj.count`
count.value++
console.log(count.value) // 2
console.log(obj.count) // 2

// изменения будут влиять и на ссылку `count`
obj.count++
console.log(obj.count) // 3
console.log(count.value) // 3
```
:::

:::warning Важно
При присвоении [ref-ссылки](refs-api.md#ref) в свойство `reactive` она разворачивается автоматически.

```ts
const count = ref(1)
const obj = reactive({})

obj.count = count

console.log(obj.count) // 1
console.log(obj.count === count.value) // true
```
:::

## `readonly`

Принимает объект (реактивный или обычный) или [ref-ссылку](refs-api.md#ref) и возвращает доступную только для чтения прокси к оригиналу. Прокси только для чтения «глубокие»: любое вложенное свойство также доступно только для чтения.

```js
const original = reactive({ count: 0 })

const copy = readonly(original)

watchEffect(() => {
  // работает для отслеживания реактивности
  console.log(copy.count)
})

// изменение оригинала вызывает методы-наблюдатели, которые отслеживают копию
original.count++

// изменение копии не сработает и приведёт к предупреждению
copy.count++ // ПРЕДУПРЕЖДЕНИЕ, потому что только для чтения!
```

Как и в случае с [`reactive`](#reactive), если какое-либо свойство использует `ref`, то оно будет автоматически разворачиваться при доступе через прокси:

```js
const raw = {
  count: ref(123)
}

const copy = readonly(raw)

console.log(raw.count.value) // 123
console.log(copy.count) // 123
```

## `isProxy`

Проверяет, является ли объект прокси, созданной с помощью [`reactive`](#reactive) или [`readonly`](#readonly).

## `isReactive`

Проверяет, является ли объект реактивной прокси, созданной с помощью [`reactive`](#reactive).

```js
import { reactive, isReactive } from 'vue'
export default {
  setup() {
    const state = reactive({
      name: 'John'
    })

    console.log(isReactive(state)) // -> true
  }
}
```

Возвращает `true`, если прокси создавалась с помощью [`readonly`](#readonly), но оборачивается другой прокси, созданной с помощью [`reactive`](#reactive).

```js{9-17}
import { reactive, isReactive, readonly } from 'vue'

export default {
  setup() {
    const state = reactive({
      name: 'John'
    })

    // прокси только для чтения, созданная из обычного объекта
    const plain = readonly({
      name: 'Mary'
    })
    console.log(isReactive(plain)) // -> false

    // прокси только для чтения, созданная из реактивной прокси
    const stateCopy = readonly(state)
    console.log(isReactive(stateCopy)) // -> true
  }
}
```

## `isReadonly`

Проверяет, является ли объект прокси только для чтения, созданной с помощью [`readonly`](#readonly).

## `toRaw`

Возвращает исходный оригинальный объект из [`reactive`](#reactive) или [`readonly`](#readonly) прокси. Применяется в крайнем случае, когда нужно только чтение без доступа/отслеживания или запись без инициирования изменений. **Не рекомендуется** хранить постоянную ссылку на оригинальный объект. Используйте с осторожностью.

```js
const foo = {}
const reactiveFoo = reactive(foo)

console.log(toRaw(reactiveFoo) === foo) // true
```

## `markRaw`

Отмечает объект, чтобы он никогда не преобразовывался в прокси. Возвращает сам объект.

```js
const foo = markRaw({})
console.log(isReactive(reactive(foo))) // false

// работает даже если вложен в другие реактивные объекты
const bar = reactive({ foo })
console.log(isReactive(bar.foo)) // false
```

:::warning ВНИМАНИЕ
API `markRaw` и `shallowXXX` (см.ниже) позволяют выборочно отказаться от глубокого реактивного/только для чтения преобразования по умолчанию и встраивать исходные, не-проксированные объекты в диаграмму состояния. Причины, по которым это может использоваться:

- Некоторые значение не должны быть реактивными. Например, сторонний комплексный экземпляр класса или объект компонента Vue.

- Пропуск преобразования в прокси может улучшить производительность при отрисовке больших списков с иммутабельными (неизменяемыми) данными.

Пропуск преобразования — продвинутая техника, потому что опциональное отключение доступно только на корневом уровне. Если установить вложенный неотмеченный исходный объект в реактивный объект и получить к нему доступ, то вернётся его проксированная версия. Это может привести к **опасности идентификации**, то есть к выполнению операции, которая основывается на идентификации объекта, но использует как исходную, так и проксированную версию одного и того же объекта:

```js
const foo = markRaw({
  nested: {}
})

const bar = reactive({
  // хотя `foo` отмечен как raw, foo.nested не будет таким.
  nested: foo.nested
})

console.log(foo.nested === bar.nested) // false
```

С опасностью идентификации сталкиваются редко. Однако, чтобы безопасно использовать API `markRaw` и `shallowXXX`, нужно твёрдо понимать, как работает система реактивности.
:::

## `shallowReactive`

Создаёт реактивный прокси, который отслеживает реактивность собственных свойств. Не выполняет глубокое реактивное преобразование вложенных объектов, предоставляя доступ к сырым значениям.

```js
const state = shallowReactive({
  foo: 1,
  nested: {
    bar: 2
  }
})

// изменение состояния собственных свойств реактивно
state.foo++
// ...но вложенные объекты не преобразуются
isReactive(state.nested) // false
state.nested.bar++ // НЕ-РЕАКТИВНО
```

В отличие от [`reactive`](#reactive), любое свойство, использующее [`ref`](refs-api.md#ref), **не будет** автоматически разворачиваться прокси.

## `shallowReadonly`

Создаёт прокси, который делает собственные свойства доступными только для чтения. При этом вложенные объекты не преобразуются, предоставляя доступ к сырым значениям.

```js
const state = shallowReadonly({
  foo: 1,
  nested: {
    bar: 2
  }
})

// изменение состояния собственных свойств не сработает
state.foo++
// ...но сработает на вложенных объектах
isReadonly(state.nested) // false
state.nested.bar++ // сработает
```

В отличие от [`readonly`](#readonly), любое свойство, которое использует [`ref`](refs-api.md#ref), **не будет** автоматически разворачиваться с помощью прокси.
