# Поддержка TypeScript

> [Vue CLI](https://cli.vuejs.org/ru/) предоставляет встроенную поддержку TypeScript из коробки.

## Официальные декларации типов в npm-пакетах

Статическая система типов может помочь предотвратить многие потенциальные ошибки по мере роста приложения, поэтому Vue 3 написан на TypeScript. А значит, для использования TypeScript во Vue не нужны никакие дополнительные инструменты — он уже поставляется с отличной поддержкой.

## Рекомендуемая конфигурация

```js
// tsconfig.json
{
  "compilerOptions": {
    "target": "esnext",
    "module": "esnext",
    // использование более строгого вывода типов для свойств данных в `this`
    "strict": true,
    "jsx": "preserve",
    "moduleResolution": "node"
  }
}
```

Обратите внимание, требуется включать `strict: true` (или хотя бы `noImplicitThis: true`, который является частью флага `strict`), чтобы проверять `this` в методах компонента, иначе он всегда будет интерпретироваться как тип `any`.

Подробнее можно прочитать в [документации настроек компилятора TypeScript](https://www.typescriptlang.org/docs/handbook/compiler-options.html).

## Конфигурация Webpack

При использовании пользовательской конфигурации Webpack `ts-loader` необходимо настроить для парсинга блоков `<script lang="ts">` во `.vue` файлах:

```js{10}
// webpack.config.js
module.exports = {
  ...
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        loader: 'ts-loader',
        options: {
          appendTsSuffixTo: [/\.vue$/],
        },
        exclude: /node_modules/,
      },
      {
        test: /\.vue$/,
        loader: 'vue-loader',
      }
      ...
```

## Инструменты для разработки

### Создание нового проекта

[Vue CLI](https://github.com/vuejs/vue-cli) умеет генерировать новые проекты, которые будут использовать TypeScript:

```bash
# 1. Устанавливаем Vue CLI, если ещё не установлен
npm install --global @vue/cli

# 2. Создаём новый проект, затем выбираем опцию "Manually select features"
vue create my-project-name

# В существующий проект Vue CLI без TypeScript
# можно добавить его поддержку с помощью плагина для Vue CLI:
vue add typescript
```

Убедитесь, что для секции `script` компонента в качестве языка указан TypeScript:

```html
<script lang="ts">
  ...
</script>
```

Или при желании совмещать использование TypeScript с [JSX `render`-функцией](render-function.md#jsx):

```html
<script lang="tsx">
  ...
</script>
```

### Поддержка в редакторах

Для разработки приложений Vue на TypeScript настоятельно рекомендуем использовать [Visual Studio Code](https://code.visualstudio.com/), обеспечивающее отличную поддержку TypeScript из коробки. При использовании [однофайловых компонентов](single-file-component.md) (SFC) также установите [расширение Volar](https://github.com/johnsoncodehk/volar), которое добавит вывод типов TypeScript в них и множество других отличных возможностей.

[WebStorm](https://www.jetbrains.com/webstorm/) также предоставляет встроенную поддержку как для TypeScript, так и для Vue.

## Объявление компонентов Vue

Чтобы TypeScript правильно определял типы внутри опций компонентов Vue необходимо объявлять компоненты с помощью глобального метода `defineComponent`:

```ts
import { defineComponent } from 'vue'

const Component = defineComponent({
  // вывод типов будет работать
})
```

При использовании [однофайловых компонентов](single-file-component.md) это будет выглядеть так:

```vue
<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
  // вывод типов будет работать
})
</script>
```

## Использование с Options API

TypeScript умеет выводить большинство типов без их явного определения. Например, если есть компонент со свойством `count`, то будет выведена ошибка при попытке вызывать строковый метод на этом свойстве:

```ts
const Component = defineComponent({
  data() {
    return {
      count: 0
    }
  },
  mounted() {
    const result = this.count.split('') // => Property 'split' does not exist on type 'number'
  }
})
```

К сложным типам или интерфейсам можно приводить с использованием [type assertion](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-assertions):

```ts
interface Book {
  title: string
  author: string
  year: number
}

const Component = defineComponent({
  data() {
    return {
      book: {
        title: 'Руководство по Vue 3',
        author: 'Команда Vue',
        year: 2021
      } as Book
    }
  }
})
```

### Расширение типов для `globalProperties`

Для добавления глобальных свойств, доступных в любом экземпляре компонента, Vue 3 предоставляет [объект `globalProperties`](../api/application-config.md#globalproperties). Например, для [плагина](plugins.md#создание-плагина) может потребоваться внедрить глобальный объект или функцию.

```ts{7,12}
import axios from 'axios'
import { createApp } from 'vue'

const app = createApp({})

// Пользовательское объявление глобального свойства
app.config.globalProperties.$http = axios

// Плагин для валидации некоторых данных
export default {
  install(app, options) {
    app.config.globalProperties.$validate = (data: object, rule: object) => {
      // проверка, что объект соответствует определённым правилам
    }
  }
}
```

Сообщить TypeScript об этих новых свойствах можно используя [расширение модуля](https://www.typescriptlang.org/docs/handbook/declaration-merging.html#module-augmentation).

Для примера выше потребовалось бы добавить следующее объявление типа:

```ts
import axios from 'axios'

declare module '@vue/runtime-core' {
  export interface ComponentCustomProperties {
    $http: typeof axios
    $validate: (data: object, rule: object) => boolean
  }
}
```

Объявление типа можно указать в этом же файле или в общем файле `*.d.ts` всего проекта (например, в `src/typings`, чтобы он автоматически загружался TypeScript). Разработчикам библиотек и плагинов нужно указать этот файл в файле `package.json` в свойстве `types`.

:::warning Убедитесь, что файл декларации является модулем TypeScript
Чтобы воспользоваться преимуществами расширения модуля, нужно убедиться что в файле будет хотя бы один `import` или `export` корневого уровня, даже если это будет просто `export {}`.

Любой файл с `import` или `export` корневого уровня рассматривается как модуль в [TypeScript](https://www.typescriptlang.org/docs/handbook/modules.html). Если объявление типов сделано вне модуля, то они перезапишут исходные типы, а не расширят их.
:::

Подробнее о типе `ComponentCustomProperties` смотрите в его [определении в `@vue/runtime-core`](https://github.com/vuejs/vue-next/blob/2587f36fe311359e2e34f40e8e47d2eebfab7f42/packages/runtime-core/src/componentOptions.ts#L64-L80) и [модульных тестах TypeScript](https://github.com/vuejs/vue-next/blob/master/test-dts/componentTypeExtensions.test-d.tsx).

### Аннотация возвращаемых типов

Из-за цикличной природы файлов деклараций Vue, TypeScript может испытывать трудности с выводом типа вычисляемых свойств. По этой причине может потребоваться аннотировать возвращаемый тип вычисляемых свойств.

```ts
import { defineComponent } from 'vue'

const Component = defineComponent({
  data() {
    return {
      message: 'Привет'
    }
  },
  computed: {
    // требуется аннотация типа
    greeting(): string {
      return this.message + '!'
    },

    // при использовании сеттера, нужна аннотация для геттера
    greetingUppercased: {
      get(): string {
        return this.greeting.toUpperCase()
      },
      set(newValue: string) {
        this.message = newValue.toUpperCase()
      }
    }
  }
})
```

### Аннотация входных параметров

Входные параметры по указанному `type` проверяются во время выполнения. Чтобы передать эти типы в TypeScript потребуется приводить конструктор с помощью `PropType`:

```ts
import { defineComponent, PropType } from 'vue'

interface Book {
  title: string
  author: string
  year: number
}

const Component = defineComponent({
  props: {
    name: String,
    id: [Number, String],
    success: { type: String },
    callback: {
      type: Function as PropType<() => void>
    },
    book: {
      type: Object as PropType<Book>,
      required: true
    },
    metadata: {
      type: null // metadata будет с типом any
    }
  }
})
```

:::warning ВНИМАНИЕ
Ввиду [ограничений](https://github.com/microsoft/TypeScript/issues/38845) TypeScript, когда дело доходит до вывода типов выражений функций, необходимо быть осторожным со значениями `validator` и `default` для объектов и массивов:
:::

```ts
import { defineComponent, PropType } from 'vue'

interface Book {
  title: string
  year?: number
}

const Component = defineComponent({
  props: {
    bookA: {
      type: Object as PropType<Book>,
      // Убедитесь, что используете стрелочную функцию
      default: () => ({
        title: 'Выражение со стрелочной функцией'
      }),
      validator: (book: Book) => !!book.title
    },
    bookB: {
      type: Object as PropType<Book>,
      // Или явно указывайте this параметром
      default(this: void) {
        return {
          title: 'Выражение с функцией'
        }
      },
      validator(this: void, book: Book) {
        return !!book.title
      }
    }
  }
})
```

### Аннотация событий

Можно объявить тип данных, передаваемых вместе с событием. Кроме того, все необъявленные в `emits` события при вызове будут выбрасывать ошибку:

```ts
const Component = defineComponent({
  emits: {
    addBook(payload: { bookName: string }) {
      // валидации во время выполнения
      return payload.bookName.length > 0
    }
  },
  methods: {
    onSubmit() {
      this.$emit('addBook', {
        bookName: 123 // Ошибка типа!
      })

      this.$emit('non-declared-event') // Ошибка типа!
    }
  }
})
```

## Использование с Composition API

В функции `setup()` не требуется указывать типы для параметра `props`, так как они будут выводиться из опции `props` компонента.

```ts
import { defineComponent } from 'vue'

const Component = defineComponent({
  props: {
    message: {
      type: String,
      required: true
    }
  },

  setup(props) {
    const result = props.message.split('') // ОК, потому что 'message' строка
    const filtered = props.message.filter(p => p.value) // ОШИБКА: Property 'filter' does not exist on type 'string'
  }
})
```

### Типизация ссылок на элементы шаблона

Иногда может потребоваться аннотировать ссылку в шаблоне на дочерний компонент, чтобы вызвать его публичный метод. Например, есть дочерний компонент `MyModal` с методом, который открывает модальное окно:

```ts
import { defineComponent, ref } from 'vue'

const MyModal = defineComponent({
  setup() {
    const isContentShown = ref(false)
    const open = () => (isContentShown.value = true)

    return {
      isContentShown,
      open
    }
  }
})
```

И вызвать этот метод из родительского компонента через ссылку на элемент шаблона:

```ts
import { defineComponent, ref } from 'vue'

const MyModal = defineComponent({
  setup() {
    const isContentShown = ref(false)
    const open = () => (isContentShown.value = true)

    return {
      isContentShown,
      open
    }
  }
})

const app = defineComponent({
  components: {
    MyModal
  },
  template: `
    <button @click="openModal">Открыть из родительского</button>
    <my-modal ref="modal" />
  `,
  setup() {
    const modal = ref()
    const openModal = () => {
      modal.value.open()
    }

    return { modal, openModal }
  }
})
```

Хоть это будет работать, но не будет никакой информации о типе `MyModal` и его доступных методах. Чтобы исправить это, нужно при создании ref-ссылки использовать `InstanceType`:

```ts
setup() {
  const modal = ref<InstanceType<typeof MyModal>>()
  const openModal = () => {
    modal.value?.open()
  }

  return { modal, openModal }
}
```

Обратите внимание, что необходимо воспользоваться оператором [optional chaining](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining) или любым другим способом, чтобы проверять, что `modal.value` не будет undefined.

### Типизация `refs`

Ref-ссылки выводят тип из исходного значения:

```ts
import { defineComponent, ref } from 'vue'

const Component = defineComponent({
  setup() {
    const year = ref(2021)

    const result = year.value.split('') // => Property 'split' does not exist on type 'number'
  }
})
```

Иногда может потребоваться указать сложный тип для внутреннего значения ref-ссылки. Это можно сделать передав общий аргумент при вызове ссылки для переопределения вывода типа по умолчанию:

```ts
const year = ref<string | number>('2021') // тип year: Ref<string | number>

year.value = 2021 // ОК!
```

:::tip Примечание
Если generic тип неизвестен, рекомендуется приводить `ref` к `Ref<T>`.
:::

### Типизация `reactive`

При типизации свойства `reactive` можно использовать интерфейсы:

```ts
import { defineComponent, reactive } from 'vue'

interface Book {
  title: string
  year?: number
}

export default defineComponent({
  name: 'HelloWorld',
  setup() {
    const book = reactive<Book>({ title: 'Руководство по Vue 3' })
    // ИЛИ
    const book: Book = reactive({ title: 'Руководство по Vue 3' })
    // ИЛИ
    const book = reactive({ title: 'Руководство по Vue 3' }) as Book
  }
})
```

### Типизация `computed`

Вычисляемые свойства автоматически выводят тип из возвращаемого значения:

```ts
import { defineComponent, ref, computed } from 'vue'

export default defineComponent({
  name: 'CounterButton',
  setup() {
    let count = ref(0)

    // только для чтения
    const doubleCount = computed(() => count.value * 2)

    const result = doubleCount.value.split('') // => Property 'split' does not exist on type 'number'
  }
})
```

### Типизация обработчиков событий

Работая с нативными событиями DOM может быть полезным типизировать аргумент, передаваемый в обработчик события. Например:

```vue
<template>
  <input type="text" @change="handleChange" />
</template>

<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
  setup() {
    // `evt` будет с типом `any`
    const handleChange = evt => {
      console.log(evt.target.value) // TypeScript здесь выбросит ошибку
    }

    return { handleChange }
  }
})
</script>
```

Как можно увидеть, без правильной типизации аргумента `evt`, TypeScript станет выбрасывать ошибку при попытке получить доступ к значению элемента `<input>`. Решением этой проблемы будет приведение к правильному типу `target` события:

```ts
const handleChange = (evt: Event) => {
  console.log((evt.target as HTMLInputElement).value)
}
```
