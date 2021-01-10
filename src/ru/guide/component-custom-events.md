# Пользовательские события

> Подразумевается, что вы уже изучили и разобрались с разделом [Основы компонентов](component-basics.md). Если нет — прочитайте его сначала.

## Стиль именования событий

Аналогично компонентам и входным параметрам, имена событий также предоставляют автоматическое преобразование регистра. Если событие генерируется в дочернем компоненте в camelCase, то в родительском можно использовать слушатель в kebab-case:

```js
this.$emit('myEvent')
```

```html
<my-component @my-event="doSomething"></my-component>
```

Как и в случае [с регистром входных параметров](component-props.md#prop-casing-camelcase-vs-kebab-case), рекомендуется использовать kebab-case при использовании DOM-шаблонов. Для строковых шаблонов этого ограничения не будет.

## Определение пользовательских событий

<VideoLesson href="https://vueschool.io/lessons/defining-custom-events-emits?friend=vuejs" title="Посмотрите бесплатное видео об определении пользовательских событий на Vue School">Посмотрите бесплатное видео об определении пользовательских событий на Vue School</VideoLesson>

Генерируемые компонентом события можно определить с помощью опции `emits`.

```js
app.component('custom-form', {
  emits: ['inFocus', 'submit']
})
```

Если в опции `emits` указано нативное событие (например, `click`), то событие компонента будет использовано **вместо** отслеживания нативного события.

:::tip Совет
Рекомендуется указывать все генерируемые компонентом события для лучшей документированности того, как компонент должен работать.
:::

### Валидация сгенерированных событий

Аналогично валидации входных параметров, генерируемые события также могут быть валидированы, если это определено с помощью объектного синтаксиса или синтаксиса массива.

Для добавления валидации событию необходимо указать функцию, которая получает аргументы, с которыми вызывался `$emit` и возвращает булево, определяющее является ли событие корректным или нет.

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
    submitForm() {
      this.$emit('submit', { email, password })
    }
  }
})
```

## Аргументы `v-model`

По умолчанию, `v-model` на компоненте использует входной параметр `modelValue` и событие `update:modelValue`. Их можно изменить с помощью аргумента `v-model`:

```html
<my-component v-model:title="bookTitle"></my-component>
```

В таком случае, ожидается что дочерний компонент будет использовать входной параметр `title` и генерировать событие `update:title` для синхронизации значения:

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

Развивая потенциал возможности определять конкретный входной параметр и событие, как мы изучили ранее с помощью [аргумента `v-model`](#v-model-arguments), теперь стало возможным создавать несколько привязок v-model на одном экземпляре компонента.

Каждая v-model синхронизирует свой входной параметр, без необходимости дополнительных опций в компоненте:

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

<common-codepen-snippet title="Multiple v-models" slug="GRoPPrM" tab="html,result" />

## Обработка модификаторов `v-model`

Изучая как работать с формами, мы столкнулись с тем, что `v-model` имеет [встроенные модификаторы](forms.md#modifiers) — `.trim`, `.number` и `.lazy`. В некоторых случах может пригодиться создавать собственные модификаторы.

Создадим для примера пользовательский модификатор `capitalize`, который будет делать заглавной первую букву строки, привязанной с помощью `v-model`.

Модификаторы, которые будут использоваться в `v-model` компонента, должны указываться через входной параметр `modelModifiers`. В примере ниже, компонент содержит входной параметр `modelModifiers`, который по умолчанию будет пустым объектом.

Обратите внимание, в хуке жизненного цикла `created` входной параметр `modelModifiers` содержит `capitalize` со значением `true` — потому что он указан на привязке `v-model` компонента `v-model.capitalize="myText"`.

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

Теперь, после настройки входного параметра, можно проверять ключи `modelModifiers` и создать обработчик для изменения значения. Например, будем запускать этот обработчик каждый раз, когда элемент `<input />` генерирует событие `input`.

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

При использовании привязки `v-model` с аргументом, имя входного параметра будет генерироваться как `arg + "Modifiers"`:

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
