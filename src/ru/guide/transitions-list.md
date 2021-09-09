# Анимация списков

На данный момент разобрались как управлять переходами для:

- Одиночных элементов
- Множества элементов, когда единовременно отображается только 1 элемент

Но что если есть целый список элементов, которые нужно отображать одновременно, например через `v-for`? В этом случае, надо использовать компонент `<transition-group>`. Прежде чем перейти к примеру, перечислим несколько важных моментов о нём:

- По умолчанию он не отрисовывает элемент-обёртку, но с помощью атрибута `tag` можно указать, какой элемент должен быть отрисован.
- [Режимы переходов](transitions-enterleave.md#режимы-переходов) недоступны, потому что больше не переключаются взаимоисключающие элементы.
- Каждый элемент внутри `<transition-group>` **всегда должен быть** с уникальным значением атрибута `key`.
- CSS-классы переходов будут применяться к внутренним элементам, а не к самой группе/контейнеру.

## Анимации добавления и удаления элементов списка

Рассмотрим небольшой пример с анимацией добавления и удаления элементов списка, используя те же CSS-классы, что и ранее:

```html
<div id="list-demo">
  <button @click="add">Добавить</button>
  <button @click="remove">Удалить</button>
  <transition-group name="list" tag="p">
    <span v-for="item in items" :key="item" class="list-item">
      {{ item }}
    </span>
  </transition-group>
</div>
```

```js
const Demo = {
  data() {
    return {
      items: [1, 2, 3, 4, 5, 6, 7, 8, 9],
      nextNum: 10
    }
  },
  methods: {
    randomIndex() {
      return Math.floor(Math.random() * this.items.length)
    },
    add() {
      this.items.splice(this.randomIndex(), 0, this.nextNum++)
    },
    remove() {
      this.items.splice(this.randomIndex(), 1)
    }
  }
}

Vue.createApp(Demo).mount('#list-demo')
```

```css
.list-item {
  display: inline-block;
  margin-right: 10px;
}
.list-enter-active,
.list-leave-active {
  transition: all 1s ease;
}
.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateY(30px);
}
```

<common-codepen-snippet title="Анимация списков" slug="e1cea580e91d6952eb0ae17bfb7c379d" tab="js,result" :editable="false" :preview="false" />

В примере есть одна проблема. При добавлении или удалении элемента, окружающие его элементы будут резко прыгать на новое место, вместо плавного перемещения. Исправим эту недоработку чуть позднее.

## Анимация перемещения элементов списка

У компонента `<transition-group>` есть ещё один козырь в рукаве. Он может анимировать не только появление и удаление элементов, но и их перемещение. Происходит это путём добавления **класса `v-move`**, который применяется при изменении позиции элементов. Как и с другими классами, его префикс определяется значением атрибута `name`, но его можно переопределить с помощью атрибута `move-class`.

Его удобно использовать для таймингов перехода и функции плавности, как показано ниже:

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.17.15/lodash.min.js"></script>

<div id="flip-list-demo">
  <button @click="shuffle">Перемешать</button>
  <transition-group name="flip-list" tag="ul">
    <li v-for="item in items" :key="item">
      {{ item }}
    </li>
  </transition-group>
</div>
```

```js
const Demo = {
  data() {
    return {
      items: [1, 2, 3, 4, 5, 6, 7, 8, 9]
    }
  },
  methods: {
    shuffle() {
      this.items = _.shuffle(this.items)
    }
  }
}

Vue.createApp(Demo).mount('#flip-list-demo')
```

```css
.flip-list-move {
  transition: transform 0.8s ease;
}
```

<common-codepen-snippet title="Пример использования Transition-group" slug="049211673d3c185fde6b6eceb8baebec" tab="html,result" :editable="false" :preview="false" />

Хоть это и выглядит как магия, но «под капотом» Vue использует анимационную технику под названием [FLIP](https://aerotwist.com/blog/flip-your-animations/), которая позволяет плавно перемещать элементы со старой позиции на новую с помощью CSS-трансформаций.

Можно теперь совместить эту технику с предыдущим примером, чтобы добавить анимации на любые изменения списка!

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.14.1/lodash.min.js"></script>

<div id="list-complete-demo" class="demo">
  <button @click="shuffle">Перемешать</button>
  <button @click="add">Добавить</button>
  <button @click="remove">Удалить</button>
  <transition-group name="list-complete" tag="p">
    <span v-for="item in items" :key="item" class="list-complete-item">
      {{ item }}
    </span>
  </transition-group>
</div>
```

```js
const Demo = {
  data() {
    return {
      items: [1, 2, 3, 4, 5, 6, 7, 8, 9],
      nextNum: 10
    }
  },
  methods: {
    randomIndex() {
      return Math.floor(Math.random() * this.items.length)
    },
    add() {
      this.items.splice(this.randomIndex(), 0, this.nextNum++)
    },
    remove() {
      this.items.splice(this.randomIndex(), 1)
    },
    shuffle() {
      this.items = _.shuffle(this.items)
    }
  }
}

Vue.createApp(Demo).mount('#list-complete-demo')
```

```css
.list-complete-item {
  transition: all 0.8s ease;
  display: inline-block;
  margin-right: 10px;
}

.list-complete-enter-from,
.list-complete-leave-to {
  opacity: 0;
  transform: translateY(30px);
}

.list-complete-leave-active {
  position: absolute;
}
```

<common-codepen-snippet title="Пример с Transition-group" slug="373b4429eb5769ae2e6d097fd954fd08" tab="js,result" :editable="false" :preview="false" />

:::tip Совет
Запомните, что FLIP-анимации не работают с элементами `display: inline`. В таких случаях можно воспользоваться `display: inline-block` или расположить элементы внутри flex-контейнера.
:::

FLIP-анимации не ограничиваются изменениями по одной оси. Элементы в многомерных массивах [также можно анимировать](https://codesandbox.io/s/github/vuejs/vuejs.org/tree/master/src/v2/examples/vue-20-list-move-transitions):

<!-- TODO: example -->

## Упругая анимация элементов списка

Управление JavaScript-переходами через data-атрибуты позволяет организовать упругую анимацию списка:

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.3.4/gsap.min.js"></script>

<div id="demo">
  <input v-model="query" />
  <transition-group
    name="staggered-fade"
    tag="ul"
    :css="false"
    @before-enter="beforeEnter"
    @enter="enter"
    @leave="leave"
  >
    <li
      v-for="(item, index) in computedList"
      :key="item.msg"
      :data-index="index"
    >
      {{ item.msg }}
    </li>
  </transition-group>
</div>
```

```js
const Demo = {
  data() {
    return {
      query: '',
      list: [
        { msg: 'Bruce Lee' },
        { msg: 'Jackie Chan' },
        { msg: 'Chuck Norris' },
        { msg: 'Jet Li' },
        { msg: 'Kung Fury' }
      ]
    }
  },
  computed: {
    computedList() {
      var vm = this
      return this.list.filter(item => {
        return item.msg.toLowerCase().indexOf(vm.query.toLowerCase()) !== -1
      })
    }
  },
  methods: {
    beforeEnter(el) {
      el.style.opacity = 0
      el.style.height = 0
    },
    enter(el, done) {
      gsap.to(el, {
        opacity: 1,
        height: '1.6em',
        delay: el.dataset.index * 0.15,
        onComplete: done
      })
    },
    leave(el, done) {
      gsap.to(el, {
        opacity: 0,
        height: 0,
        delay: el.dataset.index * 0.15,
        onComplete: done
      })
    }
  }
}

Vue.createApp(Demo).mount('#demo')
```

<common-codepen-snippet title="Упругая анимация элементов списка" slug="c2fc5107bd3025ceadea049b3ee44ec0" tab="js,result" :editable="false" :preview="false" />

## Переиспользование анимаций переходов

Благодаря компонентной системе Vue анимации переходов можно переиспользовать. Всё что нужно сделать — расположить компонент `<transition>` или `<transition-group>` в корне компонента, а затем передавать ему любые дочерние компоненты для отображения.

<!-- TODO: refactor to Vue 3 -->

Пример такого компонента для переиспользования:

```js
Vue.component('my-special-transition', {
  template: '\
    <transition\
      name="very-special-transition"\
      mode="out-in"\
      @before-enter="beforeEnter"\
      @after-enter="afterEnter"\
    >\
      <slot></slot>\
    </transition>\
  ',
  methods: {
    beforeEnter(el) {
      // ...
    },
    afterEnter(el) {
      // ...
    }
  }
})
```

Также для этой цели подходят [функциональные компоненты](render-function.md#функциональные-компоненты):

```js
Vue.component('my-special-transition', {
  functional: true,
  render: function(createElement, context) {
    var data = {
      props: {
        name: 'very-special-transition',
        mode: 'out-in'
      },
      on: {
        beforeEnter(el) {
          // ...
        },
        afterEnter(el) {
          // ...
        }
      }
    }
    return createElement('transition', data, context.children)
  }
})
```

## Динамические переходы

Да, переходы во Vue тоже могут зависеть от данных! Самый простой пример — привязка атрибута `name` к динамическому свойству.

```html
<transition :name="transitionName">
  <!-- ... -->
</transition>
```

Это удобно, когда заранее определены CSS-переходы или анимации, используя принятые во Vue соглашения об именовании классов, и хочется между ними переключаться.

Динамическую привязку можно использовать для любого атрибута `<transition>`. И речь не только об атрибутах. Ведь хуки это просто методы и у них есть доступ ко всем данных в текущем контексте. А значит можно реализовать, чтобы JavaScript-переходы вели себя по-разному, в зависимости от состояния компонента.

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/velocity/1.2.3/velocity.min.js"></script>

<div id="dynamic-fade-demo" class="demo">
  Появление:
  <input type="range" v-model="fadeInDuration" min="0" :max="maxFadeDuration" />
  Исчезновение:
  <input
    type="range"
    v-model="fadeOutDuration"
    min="0"
    :max="maxFadeDuration"
  />
  <transition
    :css="false"
    @before-enter="beforeEnter"
    @enter="enter"
    @leave="leave"
  >
    <p v-if="show">привет</p>
  </transition>
  <button v-if="stop" @click="stop = false; show = false">
    Начать анимацию
  </button>
  <button v-else @click="stop = true">Остановить!</button>
</div>
```

```js
const app = Vue.createApp({
  data() {
    return {
      show: true,
      fadeInDuration: 1000,
      fadeOutDuration: 1000,
      maxFadeDuration: 1500,
      stop: true
    }
  },
  mounted() {
    this.show = false
  },
  methods: {
    beforeEnter(el) {
      el.style.opacity = 0
    },
    enter(el, done) {
      var vm = this
      Velocity(
        el,
        { opacity: 1 },
        {
          duration: this.fadeInDuration,
          complete: function() {
            done()
            if (!vm.stop) vm.show = false
          }
        }
      )
    },
    leave(el, done) {
      var vm = this
      Velocity(
        el,
        { opacity: 0 },
        {
          duration: this.fadeOutDuration,
          complete: function() {
            done()
            vm.show = true
          }
        }
      )
    }
  }
})

app.mount('#dynamic-fade-demo')
```

<!-- TODO: example -->

Будет ещё больше возможностей для создания динамических переходов, если создать компоненты, которые по входным параметрам будут определять каким образом должна работать анимация перехода. Звучит избито, но всё ограничивается лишь воображением.
