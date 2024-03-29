# API приложения

Во Vue 3 все API, которые глобально изменяют поведение Vue, были вынесены в экземпляр приложения, создаваемого с помощью нового метода `createApp`. Теперь их эффекты ограничены конкретным экземпляром приложения:

```js
import { createApp } from 'vue'

const app = createApp({})
```

Вызов `createApp` возвращает экземпляр приложения. Этот экземпляр предоставляет доступ к контексту приложения. Всё дерево компонентов, смонтированных экземпляром приложения, имеет один и тот же контекст, конфигурации которого были «глобальными» во Vue 2.x.

Так как метод `createApp` возвращает экземпляр приложения, то теперь можно составлять цепочки вызовов из других методов. Информация о них содержится в следующих разделах.

## component

- **Аргументы:**

  - `{string} name`
  - `{Function | Object} definition (опционально)`

- **Возвращает:**

  - Экземпляр приложения, если аргумент `definition` передавался
  - Экземпляр компонента, если аргумент `definition` не передавался

- **Использование:**

  Регистрация или получение глобального компонента. Регистрация также автоматически устанавливает `name` компонента в соответствии с полученным параметром `name`.

- **Пример:**

```js
import { createApp } from 'vue'

const app = createApp({})

// регистрация с объектом настроек
app.component('my-component', {
  /* ... */
})

// получение зарегистрированного компонента
const MyComponent = app.component('my-component')
```

- **См. также:** [Компоненты](../guide/component-basics.md)

## config

- **Использование:**

  Объект с конфигурацией приложения.

- **Пример:**

```js
import { createApp } from 'vue'
const app = createApp({})

app.config = {...}
```

- **См. также:** [Конфигурация приложения](application-config.md)

## directive

- **Аргументы:**

  - `{string} name`
  - `{Function | Object} definition (опционально)`

- **Возвращает:**

  - Экземпляр приложения, если аргумент `definition` передавался
  - Экземпляр директивы, если аргумент `definition` не передавался

- **Использование:**

  Регистрация или получение глобальной директивы.

- **Пример:**

```js
import { createApp } from 'vue'
const app = createApp({})

// регистрация
app.directive('my-directive', {
  // Директива имеет набор хуков жизненного цикла:
  // вызывается до привязки атрибутов или слушателей событий к элементу
  created() {},
  // вызывается перед монтированием компонента, в котором находится элемент с директивой
  beforeMount() {},
  // вызывается после монтирования компонента, в котором находится элемент с директивой
  mounted() {},
  // вызывается перед обновлением VNode родительского компонента
  beforeUpdate() {},
  // вызывается после обновления VNode родительского компонента и VNodes его дочерних элементов
  updated() {},
  // вызывается перед тем, как будет размонтирован родительский компонент
  beforeUnmount() {},
  // вызывается после того, как будет размонтирован родительский компонент
  unmounted() {}
})

// регистрация (определение директивы с помощью функции)
app.directive('my-directive', () => {
  // будет вызвана для хуков `mounted` и `updated`
})

// получение определения зарегистрированной директивы
const myDirective = app.directive('my-directive')
```

Хуки директивы принимают следующие аргументы:

#### el

Элемент, к которому привязывается директива. Можно использовать для прямого манипулирования DOM.

#### binding

Объект, содержащий следующие свойства:

- `instance`: Экземпляр компонента, где используется директива.
- `value`: Значение, переданное в директиву. Например, для `v-my-directive="1 + 1"` значением будет `2`.
- `oldValue`: Предыдущее значение доступно только в хуках `beforeUpdate` и `updated`. Свойство доступно независимо от того, изменялось ли значение.
- `arg`: Аргумент, передаваемый с директивой (если указан). Например, для `v-my-directive:foo` аргументом будет `"foo"`.
- `modifiers`: Объект, содержащий модификаторы (если указаны). Например, для `v-my-directive.foo.bar` объект будет таким: `{ foo: true, bar: true }`.
- `dir`: Объект, который передавался в качестве параметра при регистрации директивы. Например, для директивы:

  ```js
  app.directive('focus', {
    mounted(el) {
      el.focus()
    }
  })
  ```

  в `dir` будет передаваться объект:

  ```js
  {
    mounted(el) {
      el.focus()
    }
  }
  ```

#### vnode

Схема реального DOM-элемента, полученного выше в качестве аргумента `el`.

#### prevNode

Предыдущая виртуальная нода. Доступна только в хуках `beforeUpdate` и `updated`.

:::tip Примечание
Ко всем этим аргументам, кроме `el`, следует относиться как доступным «только для чтения» и никогда не изменять их. Если нужно передать данные между хуками, рекомендуем использовать [dataset](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/dataset) элемента.
:::

- **См. также:** [Пользовательские директивы](../guide/custom-directive.md)

## mixin

- **Аргументы:**

  - `{Object} mixin`

- **Возвращает:**

  - Экземпляр приложения

- **Использование:**

  Добавляет примесь (mixin) ко всей области приложения. Зарегистрированные примеси можно использовать в шаблоне любого компонента. Это будет полезно разработчикам плагинов для добавления собственного пользовательского поведения во все компоненты. **Не рекомендуется использовать в коде приложения**.

- **См. также:** [Глобальные примеси](../guide/mixins.md#глобальные-примеси)

## mount

- **Аргументы:**

  - `{Element | string} rootContainer`
  - `{boolean} isHydrate (опционально)`

- **Возвращает:**

  - Экземпляр корневого компонента

- **Использование:**

  Заменяет `innerHTML` указанного DOM-элемента на отрисованный шаблон корневого компонента приложения.

- **Пример:**

```html
<body>
  <div id="my-app"></div>
</body>
```

```js
import { createApp } from 'vue'

const app = createApp({})
// необходимые приготовления
app.mount('#my-app')
```

- **См. также:** [Диаграмма жизненного цикла](../guide/instance.md#диаграмма-жизненного-цикла)

## provide

- **Аргументы:**

  - `{string | Symbol} key`
  - `value`

- **Возвращает:**

  - Экземпляр приложения

- **Использование:**

  Устанавливает значение, которое будет внедрено во все компоненты внутри приложения. Компоненты должны использовать `inject` для получения этих установленных значений.

  С точки зрения `provide`/`inject`, приложение может рассматриваться как предок на корневом уровне, а корневой компонент — как его единственный дочерний элемент.

  Не следует путать этот метод с предоставлением [опций компоненту](options-composition.md#provide-inject) или [функции](composition-api.md#provide-inject) в Composition API. Хотя они также являются частью того же механизма `provide`/`inject`, они используются для конфигурирования значений, предоставляемых компонентом, а не приложением.

  Предоставление значений через приложение полезно при разработке плагинов, так как плагины обычно не могут предоставлять значения с помощью компонентов. Это альтернатива использованию [globalProperties](application-config.md#globalproperties).

  :::tip Примечание
  Привязки `provide` и `inject` **НЕ ЯВЛЯЮТСЯ РЕАКТИВНЫМИ**. Так и задумано. Однако, при передаче реактивного объекта, его свойства останутся реактивными.
  :::

- **Пример:**

  Внедрение свойства в корневой компонент со значением, предоставленным приложением:

```js
import { createApp } from 'vue'

const app = createApp({
  inject: ['user'],
  template: `
    <div>
      {{ user }}
    </div>
  `
})

app.provide('user', 'administrator')
```

- **См. также:** [Provide / Inject](../guide/component-provide-inject.md)

## unmount

- **Использование:**

  Демонтирует корневой компонент экземпляра приложения.

- **Пример:**

```html
<body>
  <div id="my-app"></div>
</body>
```

```js
import { createApp } from 'vue'

const app = createApp({})
// необходимые приготовления
app.mount('#my-app')

// приложение будет размонтировано через 5 секунд после монтирования
setTimeout(() => app.unmount(), 5000)
```

## use

- **Аргументы:**

  - `{Object | Function} plugin`
  - `...options (опционально)`

- **Возвращает:**

  - Экземпляр приложения

- **Использование:**

  Установка плагина Vue.js. При передаче объекта он должен содержать метод `install`. Если передаётся функция, то она рассматривается как метод `install` для установки плагина.

  Экземпляр приложения передаётся первым аргументом при вызове метода `install`. Любые настройки `options`, переданные в `use`, будут в последующих аргументах.

  При многократном вызове метода установки одного и того же плагина установка будет выполнена только один раз.

- **Пример:**

  ```js
  import { createApp } from 'vue'
  import MyPlugin from './plugins/MyPlugin'

  const app = createApp({})

  app.use(MyPlugin)
  app.mount('#app')
  ```

- **См. также:** [Плагины](../guide/plugins.md)

## version

- **Использование:**

  Возвращает номер установленной версии Vue в виде строки. Это полезно для сторонних [плагинов](../guide/plugins.md), в которых могут использоваться разные стратегии для разных версий.

- **Пример:**

  ```js
  export default {
    install(app) {
      const version = Number(app.version.split('.')[0])

      if (version < 3) {
        console.warn('Для этого плагина требуется Vue 3')
      }

      // ...
    }
  }
  ```

- **См. также**: [Глобальное API — version](global-api.md#version)
