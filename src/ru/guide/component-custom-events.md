# Пользовательские события

> Подразумевается, что уже изучили и разобрались с разделом [Основы компонентов](component-basics.md). Если нет — прочитайте его сначала.

## Стиль именования событий

Также как для компонентов и входных параметров, для имён событий выполняется автоматическое преобразование регистра. Если событие генерируется в дочернем компоненте в camelCase, то в родительском потребуется прослушивать в kebab-case:

```js
this.$emit('myEvent')
```

```html
<my-component @my-event="doSomething"></my-component>
```

Как и в случае [с регистром входных параметров](component-props.md#регистр-в-именах-входных-параметров), рекомендуется использовать kebab-case при использовании DOM-шаблонов. Для строковых шаблонов этого ограничения не будет.

## Определение пользовательских событий

<VideoLesson href="https://vueschool.io/lessons/defining-custom-events-emits?friend=vuejs" title="Посмотрите бесплатное видео об определении пользовательских событий на Vue School">Посмотрите бесплатное видео об определении пользовательских событий на Vue School</VideoLesson>

События генерируемые компонентом можно объявить с помощью опции `emits`.

```js
app.component('custom-form', {
  emits: ['inFocus', 'submit']
})
```

Если в опции `emits` объявить нативное событие (например, `click`), то будет использоваться событие компонента **вместо** отслеживания нативного события.

:::tip Совет
Рекомендуется объявлять все генерируемые события, чтобы лучше задокументировать как должен работать компонент.
:::

### Валидация сгенерированных событий

Аналогично валидации входных параметров, также можно валидировать и генерируемые события, если они объявлены с помощью объектного синтаксиса или синтаксиса массива.

Для добавления валидации события нужно указать функцию, которая получит те аргументы, с которыми был вызван `$emit`, и возвращает булево, чтобы сообщить является ли событие корректным или нет.

```js
app.component('custom-form', {
  emits: {
    // Без валидации
    click: null,

    // Валидация события submit
    submit: ({ email, password }) => {
      if (email && password) {
        return true
      } else {
        console.warn('Некорректные данные для генерации события submit!')
        return false
      }
    }
  },
  methods: {
    submitForm(email, password) {
      this.$emit('submit', { email, password })
    }
  }
})
```

## Аргументы `v-model`

При использовании `v-model` на компоненте по умолчанию используется входной параметр `modelValue` и событие `update:modelValue`. Их можно изменить через аргумент `v-model`:

```html
<my-component v-model:title="bookTitle"></my-component>
```

Для такой записи дочерний компонент будет ожидать входной параметр `title` и должен будет генерировать событие `update:title` для синхронизации значения:

```js
app.component('my-component', {
  props: {
    title: String
  },
  emits: ['update:title'],
  template: `
    <input
      type="text"
      :value="title"
      @input="$emit('update:title', $event.target.value)">
  `
})
```

```html
<my-component v-model:title="bookTitle"></my-component>
```

## Использование нескольких `v-model`

Развивая потенциал возможности настройки используемого входного параметра и события, как изучили выше с помощью [аргумента `v-model`](#аргументы-v-model), теперь стало возможным одновременно использовать несколько привязок `v-model` на одном экземпляре компонента.

Каждая `v-model` будет синхронизироваться со своим входным параметром, не требуя дополнительных настроек в компоненте:

```html
<user-name
  v-model:first-name="firstName"
  v-model:last-name="lastName"
></user-name>
```

```js
app.component('user-name', {
  props: {
    firstName: String,
    lastName: String
  },
  emits: ['update:firstName', 'update:lastName'],
  template: `
    <input
      type="text"
      :value="firstName"
      @input="$emit('update:firstName', $event.target.value)">

    <input
      type="text"
      :value="lastName"
      @input="$emit('update:lastName', $event.target.value)">
  `
})
```

<common-codepen-snippet title="Использование нескольких v-model" slug="GRoPPrM" tab="html,result" />

## Обработка модификаторов `v-model`

Изучая работу с формами, можно заметить, что у `v-model` есть [встроенные модификаторы](forms.md#модификаторы) — `.trim`, `.number` и `.lazy`. Но в некоторых случаях может потребоваться добавить пользовательские модификаторы.

Создадим для примера пользовательский модификатор `capitalize`, который станет делать заглавной первую букву строки, привязанной с помощью `v-model`.

Модификаторы, которые будут использоваться в `v-model` компонента нужно указывать через входной параметр `modelModifiers`. В примере ниже, у компонента есть входной параметр `modelModifiers`, который по умолчанию будет пустым объектом.

Обратите внимание, в хуке жизненного цикла `created` входной параметр `modelModifiers` будет содержать `capitalize` со значением `true` — потому что он используется на привязке `v-model` компонента `v-model.capitalize="myText"`.

```html
<my-component v-model.capitalize="myText"></my-component>
```

```js
app.component('my-component', {
  props: {
    modelValue: String,
    modelModifiers: {
      default: () => ({})
    }
  },
  emits: ['update:modelValue'],
  template: `
    <input
      type="text"
      :value="modelValue"
      @input="$emit('update:modelValue', $event.target.value)">
  `,
  created() {
    console.log(this.modelModifiers) // { capitalize: true }
  }
})
```

После настройки входного параметра теперь можно проверять ключи `modelModifiers` и реализовать обработчик для модификации значения. В примере ниже обработчик будет запускаться каждый раз, когда элемент `<input />` генерирует событие `input`.

```html
<div id="app">
  <my-component v-model.capitalize="myText"></my-component>
  {{ myText }}
</div>
```

```js
const app = Vue.createApp({
  data() {
    return {
      myText: ''
    }
  }
})

app.component('my-component', {
  props: {
    modelValue: String,
    modelModifiers: {
      default: () => ({})
    }
  },
  emits: ['update:modelValue'],
  methods: {
    emitValue(e) {
      let value = e.target.value
      if (this.modelModifiers.capitalize) {
        value = value.charAt(0).toUpperCase() + value.slice(1)
      }
      this.$emit('update:modelValue', value)
    }
  },
  template: `<input
    type="text"
    :value="modelValue"
    @input="emitValue">`
})

app.mount('#app')
```

Если используется привязка `v-model` с аргументом, то имя входного параметра будет генерироваться таким образом `arg + "модификатор"`:

```html
<my-component v-model:description.capitalize="myText"></my-component>
```

```js
app.component('my-component', {
  props: ['description', 'descriptionModifiers'],
  emits: ['update:description'],
  template: `
    <input
      type="text"
      :value="description"
      @input="$emit('update:description', $event.target.value)">
  `,
  created() {
    console.log(this.descriptionModifiers) // { capitalize: true }
  }
})
```
