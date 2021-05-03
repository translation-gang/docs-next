# Composition API

> Этот раздел использует синтаксис [однофайловых компонентов](../guide/single-file-component.md) в примерах кода

## `setup`

Опция компонента, которая выполняется **перед созданием** компонента, после разрешения входных параметров `props`, и служит точкой входа для Composition API.

- **Аргументы:**

  - `{Data} props` — входные параметры
  - `{SetupContext} context` — контекст

- **Типы**:

```ts
interface Data {
  [key: string]: unknown
}

interface SetupContext {
  attrs: Data
  slots: Slots
  emit: (event: string, ...args: unknown[]) => void
}

function setup(props: Data, context: SetupContext): Data
```

:::tip Совет
Для получения автодополнения типов для аргументов, передаваемых в `setup()`, необходимо использовать [defineComponent](global-api.md#definecomponent).
:::

- **Пример**

  С использованием шаблона:

  ```vue
  <!-- MyBook.vue -->
  <template>
    <div>{{ readersNumber }} {{ book.title }}</div>
  </template>

  <script>
    import { ref, reactive } from 'vue'

    export default {
      setup() {
        const readersNumber = ref(0)
        const book = reactive({ title: 'Vue 3 Guide' })

        // объявление для шаблона
        return {
          readersNumber,
          book
        }
      }
    }
  </script>
  ```

  С использованием render-функции:

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

- **См. также**: [Composition API `setup`](../guide/composition-api-setup.md)

## Хуки жизненного цикла

Хуки жизненного цикла можно регистрировать через явно импортируемые `onX` функции:

```js
import { onMounted, onUpdated, onUnmounted } from 'vue'

const MyComponent = {
  setup() {
    onMounted(() => {
      console.log('примонтирован!')
    })
    onUpdated(() => {
      console.log('обновлён!')
    })
    onUnmounted(() => {
      console.log('размонтирован!')
    })
  }
}
```

Функции хуков жизненного цикла могут использоваться только синхронно в [`setup()`](#setup), так как они полагаются на внутреннее локальное состояние для обнаружения текущего активного экземпляра (экземпляра компонента чей `setup()` сейчас вызывается). Вызов их без текущего активного экземпляра будет приводить к ошибке.

Контекст экземпляра компонента также устанавливается во время синхронного выполнения хуков жизненного цикла. В результате, методы-наблюдатели и вычисляемые свойства созданные синхронно внутри хуков жизненного цикла будут также автоматически уничтожаться при размонтировании компонента.

- **Сопоставление между Options API и Composition API**

  - ~~`beforeCreate`~~ -> используйте `setup()`
  - ~~`created`~~ -> используйте `setup()`
  - `beforeMount` -> `onBeforeMount`
  - `mounted` -> `onMounted`
  - `beforeUpdate` -> `onBeforeUpdate`
  - `updated` -> `onUpdated`
  - `beforeUnmount` -> `onBeforeUnmount`
  - `unmounted` -> `onUnmounted`
  - `errorCaptured` -> `onErrorCaptured`
  - `renderTracked` -> `onRenderTracked`
  - `renderTriggered` -> `onRenderTriggered`

- **См. также**: [Composition API. Хуки жизненного цикла](../guide/composition-api-lifecycle-hooks.md)

## Provide / Inject

Функции `provide` и `inject` предоставляют возможность инъекции зависимостей. Могут быть вызваны только во время [`setup()`](#setup) с текущим активным экземпляром.

- **Типы**:

```ts
interface InjectionKey<T> extends Symbol {}

function provide<T>(key: InjectionKey<T> | string, value: T): void

// без значения по умолчанию
function inject<T>(key: InjectionKey<T> | string): T | undefined

// со значением по умолчанию
function inject<T>(key: InjectionKey<T> | string, defaultValue: T): T

// с функцией-фабрикой
function inject<T>(
  key: InjectionKey<T> | string,
  defaultValue: () => T,
  treatDefaultAsFactory: true
): T
```

Vue предоставляет интерфейс `InjectionKey` — общий тип, расширяющий `Symbol`. Может использоваться для синхронизации типа внедряемого значения между провайдером и потребителем:

```ts
import { InjectionKey, provide, inject } from 'vue'

const key: InjectionKey<string> = Symbol()

provide(key, 'foo') // указание не-строчного значения приведёт к ошибке

const foo = inject(key) // тип foo: string | undefined
```

При использовании строковых ключей или не-типизированных Symbol, тип внедряемого значения необходимо явно объявить:

```ts
const foo = inject<string>('foo') // string | undefined
```

- **См. также**:
  - [Provide / Inject](../guide/component-provide-inject.md)
  - [Composition API. Provide / Inject](../guide/composition-api-provide-inject.md)

## `getCurrentInstance`

Метод `getCurrentInstance` позволяет получить доступ к внутреннему экземпляру компонента, что может быть полезно для продвинутых пользователей или разработчиков библиотек.

```ts
import { getCurrentInstance } from 'vue'

const MyComponent = {
  setup() {
    const internalInstance = getCurrentInstance()

    internalInstance.appContext.config.globalProperties // доступ к globalProperties
  }
}
```

Метод `getCurrentInstance` **работает только** во время [setup](#setup) или [хуков жизненного цикла](#lifecycle-hooks)

> При использовании вне [setup](#setup) или [хуков жизненного цикла](#lifecycle-hooks) вызывайте `getCurrentInstance()` в `setup` и используйте вместо него экземпляр.

```ts
const MyComponent = {
  setup() {
    const internalInstance = getCurrentInstance() // работает

    const id = useComponentId() // работает

    const handleClick = () => {
      getCurrentInstance() // НЕ БУДЕТ РАБОТАТЬ
      useComponentId() // НЕ БУДЕТ РАБОТАТЬ

      internalInstance // работает
    }

    onMounted(() => {
      getCurrentInstance() // работает
    })

    return () =>
      h(
        'button',
        {
          onClick: handleClick
        },
        `uid: ${id}`
      )
  }
}

// также работает при вызове в функции композиции
function useComponentId() {
  return getCurrentInstance().uid
}
```
