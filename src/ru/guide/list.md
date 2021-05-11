# Отрисовка списков

## Отображение элементов массива через `v-for`

Используйте директиву `v-for` для отрисовки списка элементов на основе массива данных. У директивы `v-for` специальный синтаксис: `item in items`, где `items` — исходный массив, а `item` — **ссылка** на итерируемый элемент массива:

```html
<ul id="array-rendering">
  <li v-for="item in items">
    {{ item.message }}
  </li>
</ul>
```

```js
Vue.createApp({
  data() {
    return {
      items: [
        { message: 'Foo' },
        { message: 'Bar' }
      ]
    }
  }
}).mount('#array-rendering')
```

Результат:

<common-codepen-snippet title="v-for с массивом" slug="VwLGbwa" tab="js,result" :preview="false" />

Внутри блока `v-for` доступны все свойства из области видимости родителя. Также может быть второй опциональный параметр у `v-for` с индексом текущего элемента.

```html
<ul id="array-with-index">
  <li v-for="(item, index) in items">
    {{ parentMessage }} - {{ index }} - {{ item.message }}
  </li>
</ul>
```

```js
Vue.createApp({
  data() {
    return {
      parentMessage: 'Родитель',
      items: [
        { message: 'Foo' },
        { message: 'Bar' }
      ]
    }
  }
}).mount('#array-with-index')
```

Результат:

<common-codepen-snippet title="v-for с массивом и индексом" slug="wvaEdBP" tab="js,result" :preview="false" />

Можно использовать `of` в качестве разделителя вместо `in`, как в итераторах JavaScript:

```html
<div v-for="item of items"></div>
```

## Отображение свойств объекта через `v-for`

Также можно использовать `v-for` для итерирования по свойствам объекта:

```html
<ul id="v-for-object" class="demo">
  <li v-for="value in myObject">
    {{ value }}
  </li>
</ul>
```

```js
Vue.createApp({
  data() {
    return {
      myObject: {
        title: 'How to do lists in Vue',
        author: 'Jane Doe',
        publishedAt: '2016-04-10'
      }
    }
  }
}).mount('#v-for-object')
```

Результат:

<common-codepen-snippet title="v-for с объектом" slug="NWqLjqy" tab="js,result" :preview="false" />

Можно указать второй аргумент для получения имени свойства (ключа объекта):

```html
<li v-for="(value, name) in myObject">
  {{ name }}: {{ value }}
</li>
```

<common-codepen-snippet title="v-for с объектом и отображением ключа" slug="poJOPjx" tab="js,result" :preview="false" />

И третий — для индекса:

```html
<li v-for="(value, name, index) in myObject">
  {{ index }}. {{ name }}: {{ value }}
</li>
```

<common-codepen-snippet title="v-for с объектом, ключом и индексом" slug="abOaWdo" tab="js,result" :preview="false" />

:::tip Примечание
При итерации по объекту порядок обхода свойств будет как и в `Object.keys()`. Консистентность **не гарантируется** для различных реализаций JavaScript-движков.
:::

## Сохранение состояния

При обновлении Vue списка элементов, отрисованного директивой `v-for`, по умолчанию используется стратегия обновления «на месте». Если порядок элементов массива или объекта изменился, Vue не станет перемещать элементы DOM, а просто обновит каждый элемент «на месте», чтобы он отображал новые данные по соответствующему индексу.

Режим по умолчанию эффективен, но **применим только в случаях, когда результат отрисовки списка не полагается на состояние дочерних компонентов или временное состояние DOM (например, значения в полях форм)**.

Чтобы подсказать Vue, как определять идентичность каждого элемента, и, таким образом, переиспользовать и упорядочивать существующие элементы, необходимо указать уникальный атрибут `key` для каждого элемента:

```html
<div v-for="item in items" :key="item.id">
  <!-- содержимое -->
</div>
```

Рекомендуется всегда указывать атрибут `key` с `v-for`, кроме случаев когда итерируемое содержимое DOM простое, или когда сознательно полагаетесь на стратегию обновления по умолчанию для улучшения производительности.

Так как это общий механизм Vue для идентификации узлов, то использование `key` имеет и другие области применения, не связанные с `v-for`, которые изучим дальше в руководстве.

:::tip Совет
Не используйте в качестве ключей `v-for` непримитивные значения, такие как объекты и массивы. Вместо этого указывайте строковые или числовые значения.
:::

Подробнее об использовании атрибута `key` можно прочитать в [документации API](../api/special-attributes.md#key).

## Отслеживание изменений в массивах

### Методы изменения, мутирующие массив

У наблюдаемого массива Vue оборачивает мутирующие методы, чтобы вызывалось обновление представления при их использовании. Оборачиваются следующие методы:

- `push()`
- `pop()`
- `shift()`
- `unshift()`
- `splice()`
- `sort()`
- `reverse()`

Можно попробовать в предыдущем примере изменять массив `items`, вызывая методы которые будут его мутировать, например: `example1.items.push({ message: 'Baz' })`.

### Методы изменения с заменой массива

Методы, мутирующие массив, как следует из названия, будут изменять исходный массив, на котором они вызваны. Но существуют и другие методы, например `filter()`, `concat()` и `slice()`, которые не мутируют исходный массив, а **всегда возвращают новый массив**. При их использовании можно просто заменять старый массив на новый:

```js
example1.items = example1.items.filter(item => item.message.match(/Foo/))
```

Может показаться, что в таких случаях Vue придётся выбросить существующий DOM и заново отрисовать весь список — к счастью, это не так. Во Vue есть умные эвристики для максимизации переиспользования элементов DOM, поэтому замена одного массива другим, в случае совпадения части элементов этих массивов, будет очень эффективной операцией.

## Отображение отфильтрованных/отсортированных результатов

Иногда может потребоваться отображать отфильтрованную или отсортированную версию массива, сохранив оригинальные данные. В таком случае можно создать вычисляемое свойство, которое будет возвращать отфильтрованный или отсортированный массив.

Например:

```html
<li v-for="n in evenNumbers" :key="n">{{ n }}</li>
```

```js
data() {
  return {
    numbers: [ 1, 2, 3, 4, 5 ]
  }
},
computed: {
  evenNumbers() {
    return this.numbers.filter(number => number % 2 === 0)
  }
}
```

В ситуациях, когда вычисляемые свойства невозможно применить (например, внутри вложенных циклов `v-for`), можно использовать метод:

```html
<ul v-for="numbers in sets">
  <li v-for="n in even(numbers)" :key="n">{{ n }}</li>
</ul>
```

```js
data() {
  return {
    sets: [[ 1, 2, 3, 4, 5 ], [6, 7, 8, 9, 10]]
  }
},
methods: {
  even(numbers) {
    return numbers.filter(number => number % 2 === 0)
  }
}
```

### `v-for` и диапазоны

Можно передавать целое число в `v-for` — шаблон будет повторяться указанное число раз.

```html
<div id="range" class="demo">
  <span v-for="n in 10" :key="n">{{ n }}</span>
</div>
```

Результат:

<common-codepen-snippet title="v-for with a range" slug="NWqLjNY" tab="html,result" />

### `v-for` и `<template>`

Аналогично использованию с `v-if`, также можно использовать тег `<template>` с директивой `v-for` для отрисовки блоков из нескольких элементов. Например:

```html
<ul>
  <template v-for="item in items" :key="item.msg">
    <li>{{ item.msg }}</li>
    <li class="divider" role="presentation"></li>
  </template>
</ul>
```

### `v-for` и `v-if`

:::tip Совет
Обратите внимание, **не рекомендуется** использовать вместе `v-if` и `v-for`. Подробнее можно прочитать в разделе [рекомендаций](../style-guide/#избегаите-использования-v-if-с-v-for-важно).
:::

Когда они указаны вместе на одном узле, у `v-if` будет больший приоритет, чем у `v-for`. И поэтому в условии `v-if` не будет доступа к переменным из области видимости `v-for`:

```html
<!-- Будет ошибка, так как свойство "todo" не объявлено в экземпляре. -->

<li v-for="todo in todos" v-if="!todo.isComplete">
  {{ todo.name }}
</li>
```

Это можно исправить переместив `v-for` на тег-обёртку `<template>`:

```html
<template v-for="todo in todos" :key="todo.name">
  <li v-if="!todo.isComplete">
    {{ todo.name }}
  </li>
</template>
```

### `v-for` и компоненты

> Эта секция подразумевает, что уже знакомы с [компонентами](component-basics.md). Не стесняйтесь пропустить её сейчас и вернуться потом.

Можно использовать `v-for` на пользовательских компонентах, как на обычных элементах:

```html
<my-component v-for="item in items" :key="item.id"></my-component>
```

Однако в компонент никакие данные автоматически передаваться не будут, поскольку у каждого будет своя изолированная область видимости. Чтобы передать итерируемые данные в компонент потребуется явно использовать входные параметры:

```html
<my-component
  v-for="(item, index) in items"
  :item="item"
  :index="index"
  :key="item.id"
></my-component>
```

Причина, почему не происходит автоматической передачи `item`  в компонент заключается в том, что это сделает компонент жёстко связанным с тем, как работает `v-for`. Но явное указание на то, откуда поступают данные, позволит переиспользовать и в других ситуациях.

Небольшой полноценный пример реализации простого списка задач:

```html
<div id="todo-list-example">
  <form v-on:submit.prevent="addNewTodo">
    <label for="new-todo">Добавить задачу</label>
    <input
      v-model="newTodoText"
      id="new-todo"
      placeholder="Например, покормить кота"
    />
    <button>Добавить</button>
  </form>
  <ul>
    <todo-item
      v-for="(todo, index) in todos"
      :key="todo.id"
      :title="todo.title"
      @remove="todos.splice(index, 1)"
    ></todo-item>
  </ul>
</div>
```

```js
const app = Vue.createApp({
  data() {
    return {
      newTodoText: '',
      todos: [
        {
          id: 1,
          title: 'Do the dishes'
        },
        {
          id: 2,
          title: 'Take out the trash'
        },
        {
          id: 3,
          title: 'Mow the lawn'
        }
      ],
      nextTodoId: 4
    }
  },
  methods: {
    addNewTodo() {
      this.todos.push({
        id: this.nextTodoId++,
        title: this.newTodoText
      })
      this.newTodoText = ''
    }
  }
})

app.component('todo-item', {
  template: `
    <li>
      {{ title }}
      <button @click="$emit('remove')">Удалить</button>
    </li>
  `,
  props: ['title'],
  emits: ['remove']
})

app.mount('#todo-list-example')
```

<common-codepen-snippet title="v-for с компонентами" slug="abOaWpz" tab="js,result" :preview="false" />
