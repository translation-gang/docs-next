---
sidebarDepth: 1
---

# Глобальное API

При использовании сборки с CDN доступ к функциям глобального API можно получить через глобальный объект `Vue`, например:

```js
const { createApp, h, nextTick } = Vue
```

При использовании ES-модулей, их можно импортировать напрямую:

```js
import { createApp, h, nextTick } from 'vue'
```

Глобальные функции для работы с реактивностью, такие как `reactive` и `ref`, задокументированы отдельно. Подробнее о них — в разделе [API реактивности](reactivity-api.md).

## createApp

Возвращает экземпляр приложения, который предоставляет контекст приложения. Всё дерево компонентов, смонтированное экземпляром приложения, имеет один и тот же контекст.

```js
const app = createApp({})
```

После `createApp` можно цепочкой вызывать другие методы, список которых представлен в [API приложения](application-api.md)

### Аргументы

Первым аргументом функция получает объект настроек корневого компонента:

```js
const app = createApp({
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
const app = createApp(
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

Возвращает «виртуальную ноду» **VNode**: простой объект с информацией о том, какой узел должен отобразиться на странице, включая описание дочерних узлов. Предназначается для написания [render-функций](../guide/render-function.md) вручную:

```js
render() {
  return h('h1', {}, 'Какой-то заголовок')
}
```

### Аргументы

Принимает три аргумента: `type`, `props` и `children`

#### type

- **Тип:** `String | Object | Function`

- **Подробности:**

  Имя HTML-тега, компонента, асинхронного или функционального компонента. Если указать функцию, возвращающую `null` — будет отрисовываться комментарий. Это обязательный параметр.

#### props

- **Тип:** `Object`

- **Подробности:**

  Объект с соответствующими атрибутами, входными параметрами и событиями, которые будут использоваться в шаблоне. Опциональный параметр.

#### children

- **Тип:** `String | Array | Object`

- **Подробности:**

  Дочерние VNode, созданные с помощью `h()` или с использованием строк (для получения «текстовых VNode») или объект со слотами. Опциональный параметр.

  ```js
  h('div', {}, [
    'Какой-то текст идёт первым.',
    h('h1', 'Заголовок'),
    h(MyComponent, {
      someProp: 'foobar'
    })
  ])
  ```

## defineComponent

С точки зрения реализации, `defineComponent` лишь возвращает переданный объект. Однако, с точки зрения типизации, возвращаемое значение имеет синтетический тип конструктора для создаваемых render-функций, TSX и поддержки инструментария IDE.

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

Обычно в `defineAsyncComponent` передают функцию-фабрику, возвращающую `Promise`. Его коллбэк `resolve` должен вызываться при получении компонента с сервера. Также может быть вызван `reject(reason)` для обозначения неудачи при загрузке.

```js
import { defineAsyncComponent } from 'vue'

const AsyncComp = defineAsyncComponent(() =>
  import('./components/AsyncComponent.vue')
)

app.component('async-component', AsyncComp)
```

При [локальной регистрации](../guide/component-registration.md#локальная-регистрация) компонента можно также указать функцию напрямую, которая вернёт `Promise`:

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

Для продвинутого использования `defineAsyncComponent` может принимать объект:

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
  // Определение, является ли компонент suspensible. По умолчанию: true.
  suspensible: false,
  /**
   * Обработчик ошибки
   * @param {*} error — объект ошибки с сообщением
   * @param {*} retry — функция, определяющая повторные попытки загрузки при promise rejects
   * @param {*} fail — конец обработки ошибок
   * @param {*} attempts — максимальное число повторных попыток
   */
  onError(error, retry, fail, attempts) {
    if (error.message.match(/fetch/) && attempts <= 3) {
      // повтор при ошибках загрузки, максимум 3 попытки
      retry()
    } else {
      // обратите внимание, что retry/fail аналогичны resolve/reject для promise:
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

Позволяет разрешить компонент по его имени, если он доступен в текущем экземпляре приложения.

Возвращает `Component` или аргумент `name`, если не был найден.

```js
const app = createApp({})
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

Позволяет разрешить компонент по аналогичному механизму, как в `<component :is="">`.

Возвращает `Component` или новую созданную `VNode` с именем компонента в качестве тега. Отобразит предупреждение, если `Component` не был найден.

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

  Подробнее можно прочитать в разделе [динамические компоненты](../guide/component-dynamic-async.md).

## resolveDirective

:::warning ВНИМАНИЕ
`resolveDirective` можно использовать только внутри функций `render` или `setup`.
:::

Позволяет разрешить директиву по её имени, если она доступна в текущем экземпляре приложения.

Возвращает `Directive` или `undefined`, если директива не найдена.

```js
const app = createApp({})
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

  Каждая директива сама по себе является массивом, что позволяет объявлять до 4 индексов, как показано в следующих примерах.

  - `[directive]` — Директива сама по себе. Обязательно.

  ```js
  const MyDirective = resolveDirective('MyDirective')
  const nodeWithDirectives = withDirectives(h('div'), [[MyDirective]])
  ```

  - `[directive, value]` — То же, что и выше, плюс значение с типом `any`, которое присваивается директиве.

  ```js
  const MyDirective = resolveDirective('MyDirective')
  const nodeWithDirectives = withDirectives(h('div'), [[MyDirective, 100]])
  ```

  - `[directive, value, arg]` — То же, что и выше, плюс аргумент с типом `String`, т.е. `click` в `v-on:click`.

  ```js
  const MyDirective = resolveDirective('MyDirective')
  const nodeWithDirectives = withDirectives(h('div'), [
    [MyDirective, 100, 'click']
  ])
  ```

  - `[directive, value, arg, modifiers]` — То же, что и выше, плюс пары `key: value` с типом `Object`, определяющие любые модификаторы.

  ```js
  const MyDirective = resolveDirective('MyDirective')
  const nodeWithDirectives = withDirectives(h('div'), [
    [MyDirective, 100, 'click', { prevent: true }]
  ])
  ```

## createRenderer

Функция `createRenderer` принимает 2 общих аргумента: `HostNode` и `HostElement`, соответствующие типам `Node` и `Element` в окружении.

Например, для runtime-DOM `HostNode` будет интерфейсом DOM `Node`, а `HostElement` будет интерфейсом DOM `Element`.

Пользовательские рендеры могут передавать специфичные для платформы типы следующим образом:

```ts
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

  Узел в окружении.

#### HostElement

- **Тип:** `Element`

- **Подробности:**

  Элемент в окружении.

## nextTick

Предоставляет возможность вызвать коллбэк после следующего цикла обновления DOM. Используйте его после изменения некоторых данных, чтобы дождаться обновлённого DOM.

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

## mergeProps

Объединяет несколько объектов, содержащих входные параметры VNode. Возвращает новый объект — все передаваемые аргументами объекты останутся без изменений.

Можно передавать любое количество объектов, при этом приоритет будет у свойств, идущих позднее. Слушатели событий будут обрабатываться специальным образом, как `class` или `style`, значения этих свойств будут объединяться, а не перезаписываются.

```js
import { h, mergeProps } from 'vue'

export default {
  inheritAttrs: false,

  render() {
    const props = mergeProps({
      // Класс будет объединён с любым другим классом из $attrs
      class: 'active'
    }, this.$attrs)

    return h('div', props)
  }
}
```

## useCssModule

:::warning Предупреждение
`useCssModule` можно использовать только внутри функций `render` или `setup`.
:::

Позволяет [в функции `setup`](composition-api.md#setup) получить доступ к CSS-модулям [однофайлового компонента](../guide/single-file-component.md):

```vue
<script>
import { h, useCssModule } from 'vue'

export default {
  setup () {
    const style = useCssModule()

    return () => h('div', {
      class: style.success
    }, 'Задача выполнена!')
  }
}
</script>

<style module>
.success {
  color: #090;
}
</style>
```

Дополнительную информацию об использовании CSS-модулей можно изучить в разделе [Vue Loader — CSS модули](https://vue-loader.vuejs.org/ru/guide/css-modules.html).

### Аргументы

Принимает один аргумент: `name`

#### name

- **Тип:** `String`

- **Подробности:**

  Имя CSS-модуля. По умолчанию `'$style'`.

## version

Возвращает номер установленной версии Vue в виде строки.

```js
const version = Number(Vue.version.split('.')[0])

if (version === 3) {
  // Vue 3
} else if (version === 2) {
  // Vue 2
} else {
  // Неподдерживаемая версия Vue
}
```

**См. также**: [API приложения — version](application-api.md#version)
