# Render-функции

Vue рекомендует использовать шаблоны для создания приложений в большинстве случаев. Однако бывают ситуации, когда необходима полная программная мощь JavaScript. В таких случаях можно использовать **render-функции**.

Возьмём пример, где функция `render()` оказалась бы практичнее. Допустим, требуется сгенерировать якорные заголовки:

```html
<h1>
  <a name="hello-world" href="#hello-world">
    Hello world!
  </a>
</h1>
```

Якорные заголовки используются часто, поэтому стоит создать компонент:

```vue
<anchored-heading :level="1">Hello world!</anchored-heading>
```

Компонент должен генерировать заголовок, основываясь на входном параметре `level`, что приводит к следующему решению:

```js
const { createApp } = Vue

const app = createApp({})

app.component('anchored-heading', {
  template: `
    <h1 v-if="level === 1">
      <slot></slot>
    </h1>
    <h2 v-else-if="level === 2">
      <slot></slot>
    </h2>
    <h3 v-else-if="level === 3">
      <slot></slot>
    </h3>
    <h4 v-else-if="level === 4">
      <slot></slot>
    </h4>
    <h5 v-else-if="level === 5">
      <slot></slot>
    </h5>
    <h6 v-else-if="level === 6">
      <slot></slot>
    </h6>
  `,
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```

Такой шаблон выглядит не очень. Он не только многословен, но и дублирует `<slot></slot>` для каждого уровня заголовка. И когда добавляем новый элемент якоря, снова приходится дублировать его в каждой ветке `v-if/v-else-if`.

Хотя шаблоны отлично работают в большинстве компонентов, в данном случае очевидно, что это не один из них. Давайте перепишем его с помощью функции `render()`:

```js
const { createApp, h } = Vue

const app = createApp({})

app.component('anchored-heading', {
  render() {
    return h(
      'h' + this.level, // имя тега
      {}, // входные параметры/атрибуты
      this.$slots.default() // массив дочерних элементов
    )
  },
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```

Реализация с функцией `render()` гораздо проще, но требует больших знаний о свойствах экземпляра компонента. В этом случае необходимо знать, что когда передаются дочерние элементы без директивы `v-slot` в компонент, например `Hello world!` внутрь `anchored-heading`, то они хранятся в экземпляре компонента в `$slots.default()`. Если это ещё непонятно, **рекомендуем изучить раздел [API свойств экземпляра](../api/instance-properties.md) перед углублением в render-функции.**

## DOM-дерево

Прежде чем погрузиться в render-функции, важно немного подробнее узнать о том, как работают браузеры. Возьмём, к примеру, этот HTML:

```html
<div>
  <h1>My title</h1>
  Some text content
  <!-- TODO: Add tagline -->
</div>
```

Когда браузер читает этот код, он строит [дерево «DOM узлов»](https://javascript.info/dom-nodes) для помощи в отслеживании за всем.

Дерево DOM-узлов для HTML выше будет выглядеть так:

![Визуализация дерева DOM](/images/dom-tree.png)

Каждый элемент является узлом. Каждый текст является узлом. Каждый комментарий является узлом! Каждый узел может иметь дочерние элементы (т.е. каждый узел может содержать другие узлы).

Обновлять все эти узлы эффективно непростая задача, но, к счастью, это не потребуется делать вручную. Требуется лишь сообщать Vue какой HTML нужен на странице в шаблоне:

```html
<h1>{{ blogTitle }}</h1>
```

Или в render-функции:

```js
render() {
  return h('h1', {}, this.blogTitle)
}
```

И в обоих случаях Vue автоматически поддерживает страницу в обновлённом состоянии, даже при изменениях значения `blogTitle`.

## Виртуальное DOM-дерево

Vue обновляет страницу, создавая **виртуальный DOM**, чтобы отслеживать изменения, которые необходимо внести в реальный DOM. Взглянем внимательнее на эту строку:

```js
return h('h1', {}, this.blogTitle)
```

Что возвращает функция `h()`? Это _не совсем_ настоящий DOM-элемент. Возвращается обычный объект с информацией для Vue, какой узел должен отобразиться на страницы, включая описание любых дочерних элементов. Это описание называют «виртуальным узлом» или «виртуальной нодой», обычно сокращая до **VNode**. «Виртуальный DOM» — это всё дерево из VNode, созданных по дереву компонентов Vue.

## Аргументы `h()`

Функция `h()` является утилитой для создания VNode. Возможно, её стоило назвать `createVNode()` для точности, но она называется `h()` ввиду частого использования и для краткости. Она принимает три аргумента:

```js
// @returns {VNode}
h(
  // {String | Object | Function } тег
  // Имя HTML-тега, компонента или асинхронного компонента.
  // Использование функции, возвращающей null, будет отрисовывать комментарий.
  //
  // Обязательный параметр.
  'div',

  // {Object} входные параметры
  // Объект, соответствующий атрибутам, входным параметрам
  // и событиям, которые использовали бы в шаблоне.
  //
  // Опционально.
  {},

  // {String | Array | Object} дочерние элементы
  // Дочерние VNode, созданные с помощью `h()`,
  // или строки для получения 'текстовых VNode' или
  // объект со слотами.
  //
  // Опционально.
  [
    'Какой-то текст в начале.',
    h('h1', 'Заголовок'),
    h(MyComponent, {
      someProp: 'foobar'
    })
  ]
)
```

Если нет входных параметров, то дочерние элементы можно передать вторым аргументом. В случаях, когда это может добавить путаницы, можно указать `null` вторым аргументом, чтобы явно передавать дочерние элементы третьим аргументом.

## Полный пример

С полученными знаниями теперь можем завершить начатый компонент:

```js
const { createApp, h } = Vue

const app = createApp({})

/** Рекурсивно получаем текст дочерних узлов */
function getChildrenTextContent(children) {
  return children
    .map(node => {
      return typeof node.children === 'string'
        ? node.children
        : Array.isArray(node.children)
        ? getChildrenTextContent(node.children)
        : ''
    })
    .join('')
}

app.component('anchored-heading', {
  render() {
    // создаём ID в kebab-case из текстового содержимого дочерних узлов
    const headingId = getChildrenTextContent(this.$slots.default())
      .toLowerCase()
      .replace(/\W+/g, '-') // заменяем не-буквенные символы на тире
      .replace(/(^-|-$)/g, '') // удаляем тире в начале и конце

    return h('h' + this.level, [
      h(
        'a',
        {
          name: headingId,
          href: '#' + headingId
        },
        this.$slots.default()
      )
    ])
  },
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```

## Ограничения

### VNode должны быть уникальными

Все VNode в дереве компонентов должны быть уникальными. Это означает, что следующая render-функция некорректна:

```js
render() {
  const myParagraphVNode = h('p', 'hi')
  return h('div', [
    // НЕПРАВИЛЬНО - одинаковые VNode!
    myParagraphVNode, myParagraphVNode
  ])
}
```

Если нужно многократно дублировать один и тот же элемент/компонент, то реализовать это можно с помощью функции-фабрики. Например, следующая render-функция является абсолютно корректным способом по отрисовке 20 идентичных параграфов:

```js
render() {
  return h('div',
    Array.from({ length: 20 }).map(() => {
      return h('p', 'hi')
    })
  )
}
```

## Создание VNode компонентов

Для создания VNode для компонента, первым аргументом `h` должен быть сам компонент:

```js
render() {
  return h(ButtonCounter)
}
```

Если необходимо разрешить компонент по имени, можно использовать `resolveComponent`:

```js
const { h, resolveComponent } = Vue

// ...

render() {
  const ButtonCounter = resolveComponent('ButtonCounter')
  return h(ButtonCounter)
}
```

`resolveComponent` — та же самая функция, которую шаблоны под капотом используют для разрешения компонентов по имени.

Функции `render` обычно требуется использовать `resolveComponent` для компонентов [зарегистрированных глобально](component-registration.md#global-registration). При [локальной регистрации компонентов](component-registration.md#local-registration) обычно можно обойтись без неё. Рассмотрим следующий пример:

```js
// Можно упростить это
components: {
  ButtonCounter
},
render() {
  return h(resolveComponent('ButtonCounter'))
}
```

Вместо того, чтобы регистрировать компонент по имени, а затем искать его, можно сразу использовать его напрямую:

```js
render() {
  return h(ButtonCounter)
}
```

## Замена возможностей шаблона обычным JavaScript

### `v-if` и `v-for`

Всё что используется, может быть легко реализовано на простом JavaScript, render-функции Vue не создаёт никакой проприетарной альтернативы. Например, шаблон с использованием `v-if` и `v-for`:

```html
<ul v-if="items.length">
  <li v-for="item in items">{{ item.name }}</li>
</ul>
<p v-else>Элементов не найдено.</p>
```

Можно переписать с помощью `if`/`else` JavaScript и `map()` в render-функции:

```js
props: ['items'],
render() {
  if (this.items.length) {
    return h('ul', this.items.map((item) => {
      return h('li', item.name)
    }))
  } else {
    return h('p', 'Элементов не найдено.')
  }
}
```

В шаблоне иногда удобно использовать тег `<template>`, для указания директив `v-if` или `v-for`. При миграции на `render`-функции, тег `<template>` можно просто опустить.

### `v-model`

Директива `v-model` на этапе компиляции шаблона раскладывается на входные параметры `modelValue` и `onUpdate:modelValue` — потребуется указать их самостоятельно:

```js
props: ['modelValue'],
emits: ['update:modelValue'],
render() {
  return h(SomeComponent, {
    modelValue: this.modelValue,
    'onUpdate:modelValue': value => this.$emit('update:modelValue', value)
  })
}
```

### `v-on`

Необходимо предоставить правильное имя входного параметра для обработчика события, например, для обработки событий `click` имя входного параметра должно быть `onClick`.

```js
render() {
  return h('div', {
    onClick: $event => console.log('кликнули!', $event.target)
  })
}
```

#### Модификаторы событий

Для модификаторов событий `.passive`, `.capture` и `.once` можно указать их после имени события в camelCase.

Например:

```js
render() {
  return h('input', {
    onClickCapture: this.doThisInCapturingMode,
    onKeyupOnce: this.doThisOnce,
    onMouseoverOnceCapture: this.doThisOnceInCapturingMode
  })
}
```

Для всех остальных событий и модификаторов клавиш не требуется специального API, так как в обработчике событий можно использовать нативные свойства и методы:

| Модификатор(ы)                                        | Эквивалент в обработчике                                                                                                |
| ----------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| `.stop`                                               | `event.stopPropagation()`                                                                                            |
| `.prevent`                                            | `event.preventDefault()`                                                                                             |
| `.self`                                               | `if (event.target !== event.currentTarget) return`                                                                   |
| Клавиши:<br>`.enter`, `.13`                              | `if (event.keyCode !== 13) return` (замените `13` на [другой код клавиши](http://keycode.info/) для других модификаторов клавиш) |
| Клавиши модификаторов:<br>`.ctrl`, `.alt`, `.shift`, `.meta` | `if (!event.ctrlKey) return` (замените `ctrlKey` на `altKey`, `shiftKey` или `metaKey`, соответственно)                  |

Вот пример со всеми этими модификаторами, используемыми вместе:

```js
render() {
  return h('input', {
    onKeyUp: event => {
      // Отменяем обработку, если элемент вызвавший событие
      // не является элементом, к которому событие было привязано
      if (event.target !== event.currentTarget) return
      // Отменяем обработку, если код клавиши не соответствовал enter
      // key (13) и клавиша shift не была нажата в то же время
      if (!event.shiftKey || event.keyCode !== 13) return
      // Останавливаем всплытие события
      event.stopPropagation()
      // Останавливаем поведение по умолчанию для этого элемента
      event.preventDefault()
      // ...
    }
  })
}
```

### Слоты

Доступ к содержимому слотов в виде массива VNode можно получить через [`this.$slots`](../api/instance-properties.md#slots):

```js
render() {
  // `<div><slot></slot></div>`
  return h('div', this.$slots.default())
}
```

```js
props: ['message'],
render() {
  // `<div><slot :text="message"></slot></div>`
  return h('div', this.$slots.default({
    text: this.message
  }))
}
```

Для VNode компонента необходимо передать дочерние элементы в `h` в виде объекта, а не массива. Каждое свойство будет использовано для заполнения одноимённого слота:

```js
render() {
  // `<div><child v-slot="props"><span>{{ props.text }}</span></child></div>`
  return h('div', [
    h(
      resolveComponent('child'),
      null,
      // передаём `slots` как дочерний объект
      // в виде { slotName: props => VNode | Array<VNode> }
      {
        default: (props) => h('span', props.text)
      }
    )
  ])
}
```

Слоты передаются в виде функций, чтобы позволить дочернему компоненту управлять созданием содержимого каждого слота. Любые реактивные данные должны быть доступны внутри функции слота, чтобы гарантировать, что они зарегистрированы как зависимость дочернего компонента, а не родительского. И наоборот, обращения к `resolveComponent` должны производиться вне функции слота, иначе они будут разрешаться относительно неправильного компонента:

```js
// `<MyButton><MyIcon :name="icon" />{{ text }}</MyButton>`
render() {
  // Вызовы resolveComponent должны находиться вне функции слота
  const Button = resolveComponent('MyButton')
  const Icon = resolveComponent('MyIcon')

  return h(
    Button,
    null,
    {
      // Использование стрелочной функции для сохранения значения `this`
      default: (props) => {
        // Реактивные свойства должны считываться внутри функции слота,
        // чтобы они стали зависимостями для отрисовки дочернего компонента
        return [
          h(Icon, { name: this.icon }),
          this.text
        ]
      }
    }
  )
}
```

Если компонент получает слоты из родителя, то их можно передать напрямую дочернему компоненту:

```js
render() {
  return h(Panel, null, this.$slots)
}
```

Их также можно передавать по-отдельности или оборачивать при необходимости:

```js
render() {
  return h(
    Panel,
    null,
    {
      // Если хотим передать функцию слота, то можем это сделать
      header: this.$slots.header,
      
      // Если нужно как-то управлять слотом,
      // тогда нужно обернуть его в новую функцию
      default: (props) => {
        const children = this.$slots.default ? this.$slots.default(props) : []
        
        return children.concat(h('div', 'Extra child'))
      }
    } 
  )
}
```

### `<component>` и `is`

Под капотом шаблоны используют `resolveDynamicComponent` для реализации атрибута `is`. Можно воспользоваться этой же функции, если требуется вся гибкость, предоставляемая `is`, в создаваемой `render`-функции:

```js
const { h, resolveDynamicComponent } = Vue

// ...

// `<component :is="name"></component>`
render() {
  const Component = resolveDynamicComponent(this.name)
  return h(Component)
}
```

Аналогично `is`, `resolveDynamicComponent` поддерживает передачу имени компонента, имени HTML-элемента или объект с опциями компонента.

Однако, такой уровень гибкости обычно и не требуется. Поэтому часто можно заменить `resolveDynamicComponent` на более конкретную альтернативу.

Например, если требуется поддерживать только имена компонентов, то можно использовать `resolveComponent`.

Если VNode всегда будет HTML-элементом, то можно передать имя непосредственно в `h`:

```js
// `<component :is="bold ? 'strong' : 'em'"></component>`
render() {
  return h(this.bold ? 'strong' : 'em')
}
```

Аналогично, если передаваемое значение в `is` будет объектом опций компонента, то нет необходимости в разрешении компонентов, его можно передать напрямую первым аргументом `h`.

Подобно тегу `<template>`, тег `<component>` требуется в шаблонах только в качестве заполнителя для синтаксиса и его требуется опустить при миграции на `render`-функции.

## JSX

При создании множества `render`-функций, может быть мучительно писать подобное:

```js
h(
  resolveComponent('anchored-heading'),
  {
    level: 1
  },
  {
    default: () => [h('span', 'Hello'), ' world!']
  }
)
```

Особенно, когда версия с шаблоном выглядит очень лаконично:

```vue
<anchored-heading :level="1"> <span>Hello</span> world! </anchored-heading>
```

Вот почему существует [плагин Babel](https://github.com/vuejs/jsx-next) для использования JSX во Vue, что возвращает близкий к шаблонам синтаксис:

```jsx
import AnchoredHeading from './AnchoredHeading.vue'

const app = createApp({
  render() {
    return (
      <AnchoredHeading level={1}>
        <span>Hello</span> world!
      </AnchoredHeading>
    )
  }
})

app.mount('#demo')
```

Подробнее о том, как JSX преобразуется в JavaScript смотрите в [документации](https://github.com/vuejs/jsx-next#installation).

## Компиляция шаблона

Возможно, будет интересно узнать, что шаблоны Vue в действительности компилируются в render-функции. Обычно нет необходимости знать такие детали реализации, но может быть любопытно посмотреть каким образом компилируются те или иные возможности шаблона. Ниже приведена небольшая демонстрация использования метода `Vue.compile` для компиляции строковых шаблонов на лету:

<iframe src="https://vue-next-template-explorer.netlify.app/" width="100%" height="420"></iframe>
