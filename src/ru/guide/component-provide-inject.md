# Provide / inject

> Подразумевается, что вы уже изучили и разобрались с разделом [Основы компонентов](component-basics.md). Если нет — прочитайте его сначала.

Обычно для передачи данных от родительского компонента в дочерний используются [входные параметры](component-props.md). Представьте структуру, в которой будет несколько глубоко вложенных компонентов и потребуется что-то от родительского компонента в глубоко вложенном дочернем. В таком случае необходимо передавать входные параметры вниз по всей цепочке компонентов, что может быть очень неудобным.

В таких случаях можно использовать пару `provide` и `inject`. Родительские компоненты могут служить провайдерами зависимостей для всех своих потомков, независимо от того, насколько глубокая иерархия компонентов. Работа этой возможности строится из двух частей: родительский компонент имеет опцию `provide` для предоставления данных, а дочерний компонент имеет опцию `inject` для использования этих данных.

![Схема provide/inject](/images/components_provide.png)

Например, если у нас есть такая иерархия:

```
Root
└─ TodoList
   ├─ TodoItem
   └─ TodoListFooter
      ├─ ClearTodosButton
      └─ TodoListStatistics
```

Если нужно передать длину массива элементов todo-списка в `TodoListStatistics`, то необходимо передать входной параметр вниз по иерархии: `TodoList` -> `TodoListFooter` -> `TodoListStatistics`. С помощью подхода provide/inject, можно сделать это напрямую:

```js
const app = Vue.createApp({})

app.component('todo-list', {
  data() {
    return {
      todos: ['Feed a cat', 'Buy tickets']
    }
  },
  provide: {
    user: 'John Doe'
  },
  template: `
    <div>
      {{ todos.length }}
      <!-- остальной шаблон компонента -->
    </div>
  `
})

app.component('todo-list-statistics', {
  inject: ['user'],
  created() {
    console.log(`Внедрённое свойство: ${this.user}`) // > Внедрённое свойство: John Doe
  }
})
```

Однако, это не сработает если попытаемся указать здесь какое-нибудь свойство экземпляра компонента:

```js
app.component('todo-list', {
  data() {
    return {
      todos: ['Feed a cat', 'Buy tickets']
    }
  },
  provide: {
    todoLength: this.todos.length // это приведёт к ошибке 'Cannot read property 'length' of undefined`
  },
  template: `
    ...
  `
})
```

Для доступа к свойствам экземпляра компонента необходимо преобразовать `provide` в функцию, вощвращающую объект:

```js
app.component('todo-list', {
  data() {
    return {
      todos: ['Feed a cat', 'Buy tickets']
    }
  },
  provide() {
    return {
      todoLength: this.todos.length
    }
  },
  template: `
    ...
  `
})
```

Это позволяет безопаснее дорабатывать этот компонент, не опасаясь, что можем изменить/удалить что-нибудь, что опирается на дочерний компонент. Интерфейс между этими компонентами остаётся чётко определённым, как и в случае с входными параметрами.

На самом деле, можно считать это инъекцией зависимости как вид «входного параметра дальнего действия», за исключением:

- родительские компоненты не должны знать, какие потомки используют свойства, которые они предоставляют
- дочерним компонентам не нужно знать откуда приходят внедряемые свойства

## Работа с реактивностью

В примере выше, если изменить список `todos`, то это не будет отражено во внедряемом свойстве `todoLength`. Это связано с тем, что привязки `provide/inject` _не реактивны_ по умолчанию. Можно изменить это поведение, передав свойство `ref` или объект `reactive` в `provide`. В данном случае, если требуется реагировать на изменения в компоненте предке, необходимо присвоить свойство `computed`  Composition API во внедряемое `todoLength`:

```js
app.component('todo-list', {
  // ...
  provide() {
    return {
      todoLength: Vue.computed(() => this.todos.length)
    }
  }
})

app.component('todo-list-statistics', {
  inject: ['todoLength'],
  created() {
    console.log(`Внедряемое свойство: ${this.todoLength.value}`) // > Внедряемое свойство: 5
  }
})
```

Теперь любое изменение `todos.length` будет корректно отражаться в компонентах, куда внедряется `todoLength`. Подробнее `computed` можно изучить в [разделе Computed и Watch](reactivity-computed-watchers.md#computed-values) и `reactive` provide/inject в [разделе Composition API](composition-api-provide-inject.md#reactivity).
