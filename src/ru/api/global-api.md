---
sidebarDepth: 1
---

# Глобальное API

## createApp

Возвращает экземпляр приложения, который предоставляет контекст приложения. Всё дерево компонентов, смонтированных экземпляром приложения, имеет один и тот же контекст.

```js
const app = Vue.createApp({})
```

После `createApp` можно цепочкой вызывать другие методы, их перечень можно найти в [API приложения](application-api.md)

### Аргументы

Первым аргументом функция получает объект настроек корневого компонента:

```js
const app = Vue.createApp({
  data() {
    return {
      ...
    }
  },
  methods: {...},
  computed: {...}
  ...
})
```

Вторым аргументом можно передать корневые входные параметры приложения:

```js
const app = Vue.createApp(
  {
    props: ['username']
  },
  { username: 'Evan' }
)
```

```html
<div id="app">
  <!-- Отобразит 'Evan' -->
  {{ username }}
</div>
```

### Типы

```ts
interface Data {
  [key: string]: unknown
}

export type CreateAppFunction<HostElement> = (
  rootComponent: PublicAPIComponent,
  rootProps?: Data | null
) => App<HostElement>
```

## h

Возвращает «виртуальную ноду», для сокращения называемую как **VNode**: простой объект, содержащий информацию для Vue какой узел должен отобразиться на странице, включая описания любых дочерних узлов. Предназначается для написания [render-функций](../guide/render-function.md) вручную:

```js
render() {
  return Vue.h('h1', {}, 'Какой-то заголовок')
}
```

### Аргументы

Принимает три аргумента: `type`, `props` и `children`

#### type

- **Тип:** `String | Object | Function`

- **Подробности:**

  Имя HTML-тега, компонента или асинхронного компонента. Использование функции, возвращающей `null` будет отрисовывать комментарий. Этот параметр обязателен.

#### props

- **Тип:** `Object`

- **Подробности:**

  Объект, с соответствующими атрибутами, входными параметрами и событиями, которые будут использоваться в шаблоне. Опционально.

#### children

- **Тип:** `String | Array | Object`

- **Подробности:**

  Дочерние VNode, созданные с помощью `h()`, или использование строк для получения «текстовых VNode», или объект со слотами. Опционально.

  ```js
  h('div', {}, [
    'Some text comes first.',
    h('h1', 'A headline'),
    h(MyComponent, {
      someProp: 'foobar'
    })
  ])
  ```

## defineComponent

Реализация `defineComponent` ничего не делает, но возвращает переданный ему объект. Однако, с точки зрения типизации, возвращаемое значение имеет синтетический тип конструктора для создаваемых render-функций, TSX и поддержки инструментария IDE.

### Аргументы

Объект с опциями компонента

```js
import { defineComponent } from 'vue'

const MyComponent = defineComponent({
  data() {
    return { count: 1 }
  },
  methods: {
    increment() {
      this.count++
    }
  }
})
```

Или функция `setup`, имя которой будет использовано в качестве имени компонента

```js
import { defineComponent, ref } from 'vue'

const HelloWorld = defineComponent(function HelloWorld() {
  const count = ref(0)
  return { count }
})
```

## defineAsyncComponent

Создаёт асинхронный компонент, который будет загружен только при необходимости.

### Аргументы

Обычно, в `defineAsyncComponent` передают функцию-фабрику, возвращающую `Promise`. Его коллбэк `resolve` должен вызываться при получении описания компонента с сервера. Также можно вызываться `reject(reason)` для обозначения неудачи при загрузке.

```js
import { defineAsyncComponent } from 'vue'

const AsyncComp = defineAsyncComponent(() =>
  import('./components/AsyncComponent.vue')
)

app.component('async-component', AsyncComp)
```

При использовании [локальной регистрации](../guide/component-registration.md#local-registration) компонента также можно напрямую указывать функцию, которая возвращает `Promise`:

```js
import { createApp, defineAsyncComponent } from 'vue'

createApp({
  // ...
  components: {
    AsyncComponent: defineAsyncComponent(() =>
      import('./components/AsyncComponent.vue')
    )
  }
})
```

Для продвинутого использования `defineAsyncComponent` может принимать объект следующего формата:

```js
import { defineAsyncComponent } from 'vue'

const AsyncComp = defineAsyncComponent({
  // Функция-фабрика
  loader: () => import('./Foo.vue')
  // Компонент загрузки, используемый во время загрузки асинхронного компонента
  loadingComponent: LoadingComponent,
  // Компонент ошибки, используемый в случае неудачи при загрузке
  errorComponent: ErrorComponent,
  // Ожидание перед показом компонента загрузки. По умолчанию: 200ms.
  delay: 200,
  // Компонент ошибки будет отображаться, если был указан таймаут
  // и время ожидания превышено. По умолчанию: Infinity.
  timeout: 3000,
  // Определение является ли компонент suspensible. По умолчанию: true.
  suspensible: false,
  /**
   * Обработчик ошибки
   * @param {*} error Объект ошибки с сообщением
   * @param {*} retry Функция, определяющая повторные попытки загрузки, при promise rejects
   * @param {*} fail  Конец обработки ошибок
   * @param {*} attempts Максимальное число повторных попыток
   */
  onError(error, retry, fail, attempts) {
    if (error.message.match(/fetch/) && attempts <= 3) {
      // повтор при ошибках загрузки, максимум 3 попытки
      retry()
    } else {
      // обратите внимание, retry/fail аналогичны resolve/reject для promise:
      // один из них должен быть вызван для продолжения обработки ошибок.
      fail()
    }
  },
})
```

**См. также**: [Динамические и асинхронные компоненты](../guide/component-dynamic-async.md)

## resolveComponent

:::warning ВНИМАНИЕ
`resolveComponent` можно использовать только внутри функций `render` или `setup`.
:::

Позволяет разрешить `component` по его имени, если он доступен в текущем экземпляре приложения.

Возвращает `Component` или `undefined` если не был найден.

```js
const app = Vue.createApp({})
app.component('MyComponent', {
  /* ... */
})
```

```js
import { resolveComponent } from 'vue'
render() {
  const MyComponent = resolveComponent('MyComponent')
}
```

### Аргументы

Принимает один аргумент: `name`

#### name

- **Тип:** `String`

- **Подробности:**

  Имя загружаемого компонента.

## resolveDynamicComponent

:::warning ВНИМАНИЕ
`resolveDynamicComponent` можно использовать только внутри функций `render` или `setup`.
:::

Позволяет разрешить `component` по аналогичному механизму как в `<component :is="">`.

Возвращает `Component` или новую созданную `VNode` с именем компонента в качестве тега. Будет отображать предупреждение, если `Component` не был найден.

```js
import { resolveDynamicComponent } from 'vue'
render () {
  const MyComponent = resolveDynamicComponent('MyComponent')
}
```

### Аргументы

Принимает один аргумент: `component`

#### component

- **Тип:** `String | Object (объект настроек компонента)`

- **Подробности:**

  Подробности можно узнать в разделе [динамические компоненты](../guide/component-dynamic-async.md).

## resolveDirective

:::warning ВНИМАНИЕ
`resolveDirective` можно использовать только внутри функций `render` или `setup`.
:::

Позволяет разрешить `directive` по её имени, если она доступна в текущем экземпляре приложения.

Возвращает `Directive` или `undefined` если не была найдена.

```js
const app = Vue.createApp({})
app.directive('highlight', {})
```

```js
import { resolveDirective } from 'vue'
render () {
  const highlightDirective = resolveDirective('highlight')
}
```

### Аргументы

Принимает один аргумент: `name`

#### name

- **Тип:** `String`

- **Подробности:**

  Имя загружаемой директивы.

## withDirectives

:::warning ВНИМАНИЕ
`withDirectives` можно использовать только внутри функций `render` или `setup`.
:::

Позволяет применять директивы к **VNode**. Возвращает VNode с применёнными директивами.

```js
import { withDirectives, resolveDirective } from 'vue'
const foo = resolveDirective('foo')
const bar = resolveDirective('bar')

return withDirectives(h('div'), [
  [foo, this.x],
  [bar, this.y]
])
```

### Аргументы

Принимает два аргумента: `vnode` и `directives`.

#### vnode

- **Тип:** `vnode`

- **Подробности:**

  Виртуальная нода, обычно созданная с помощью `h()`.

#### directives

- **Тип:** `Array`

- **Подробности:**

  Массив директив.

  Each directive itself is an array, which allows for up to 4 indexes to be defined as seen in the following examples.

  - `[directive]` - The directive by itself. Required.

  ```js
  const MyDirective = resolveDirective('MyDirective')
  const nodeWithDirectives = withDirectives(h('div'), [[MyDirective]])
  ```

  - `[directive, value]` - The above, plus a value of type `any` to be assigned to the directive

  ```js
  const MyDirective = resolveDirective('MyDirective')
  const nodeWithDirectives = withDirectives(h('div'), [[MyDirective, 100]])
  ```

  - `[directive, value, arg]` - The above, plus a `String` argument, ie. `click` in `v-on:click`

  ```js
  const MyDirective = resolveDirective('MyDirective')
  const nodeWithDirectives = withDirectives(h('div'), [
    [MyDirective, 100, 'click']
  ])
  ```

  - `[directive, value, arg, modifiers]` - The above, plus a `key: value` pair `Object` defining any modifiers.

  ```js
  const MyDirective = resolveDirective('MyDirective')
  const nodeWithDirectives = withDirectives(h('div'), [
    [MyDirective, 100, 'click', { prevent: true }]
  ])
  ```

## createRenderer

The createRenderer function accepts two generic arguments:
`HostNode` and `HostElement`, corresponding to Node and Element types in the
host environment.

For example, for runtime-dom, HostNode would be the DOM
`Node` interface and HostElement would be the DOM `Element` interface.

Custom renderers can pass in the platform specific types like this:

```js
import { createRenderer } from 'vue'
const { render, createApp } = createRenderer<Node, Element>({
  patchProp,
  ...nodeOps
})
```

### Аргументы

Принимает два аргумента: `HostNode` и `HostElement`

#### HostNode

- **Тип:** `Node`

- **Подробности:**

  The node in the host environment.

#### HostElement

- **Тип:** `Element`

- **Подробности:**

  The element in the host environment.

## nextTick

Defer the callback to be executed after the next DOM update cycle. Use it immediately after you’ve changed some data to wait for the DOM update.

```js
import { createApp, nextTick } from 'vue'

const app = createApp({
  setup() {
    const message = ref('Привет!')
    const changeMessage = async newMessage => {
      message.value = newMessage
      await nextTick()
      console.log('Теперь DOM обновлён')
    }
  }
})
```

**См. также**: [метод экземпляра `$nextTick`](instance-methods.md#nexttick)
