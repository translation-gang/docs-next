# Анимация переходов между состояниями

Система переходов Vue предлагает множество простых способ анимирования появления, исчезновения, а также списков, но что насчёт анимации самих данных? Например:

- числа и расчёты
- отображаемые цвета
- положение SVG-узлов
- размеры и другие свойства элементов

Все они либо уже хранятся в виде необработанных чисел, либо могут быть преобразованы в числа. Как только это сделать, можно анимировать изменения состояний с помощью сторонних библиотек, в сочетании с системой реактивности и компонентами Vue.

## Анимация состояния при помощи наблюдателей

Методы-наблюдатели (watch) позволяют анимировать изменения одного числа на другое. Это может звучать сложно, поэтому рассмотрим пример с использованием [GreenSock](https://greensock.com/):

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.2.4/gsap.min.js"></script>

<div id="animated-number-demo">
  <input v-model.number="number" type="number" step="20" />
  <p>{{ animatedNumber }}</p>
</div>
```

```js
const Demo = {
  data() {
    return {
      number: 0,
      tweenedNumber: 0
    }
  },
  computed: {
    animatedNumber() {
      return this.tweenedNumber.toFixed(0)
    }
  },
  watch: {
    number(newValue) {
      gsap.to(this.$data, { duration: 0.5, tweenedNumber: newValue })
    }
  }
}

Vue.createApp(Demo).mount('#animated-number-demo')
```

<common-codepen-snippet title="Transitioning State 1" slug="22903bc3b53eb5b7817378ecb985ce96" tab="js,result" :editable="false" :preview="false" />

При обновлении числа в поле ввода произойдёт анимация изменения под ним.

## Динамические переходы между состояниями

Обновляться в реальном времени могут как компоненты Vue для анимации, так и данные, на которых строятся переходы, что особенно полезно для прототипирования! Используя даже простой SVG-полигон можно реализовать множества интересных эффектов, которых сложно достичь без небольшой игры с переменными.

<common-codepen-snippet title="Обновление SVG" slug="a8e00648d4df6baa1b19fb6c31c8d17e" :height="500" tab="js,result" :editable="false" />

## Организация переходов в компонентах

Управление множеством состояний переходов может быстро увеличить сложность экземпляра компонента. К счастью, многие анимации можно выделить в отдельные дочерние компоненты. Создадим такой для анимированного числа из примера выше:

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.2.4/gsap.min.js"></script>

<div id="app">
  <input v-model.number="firstNumber" type="number" step="20" /> +
  <input v-model.number="secondNumber" type="number" step="20" /> = {{ result }}
  <p>
    <animated-integer :value="firstNumber"></animated-integer> +
    <animated-integer :value="secondNumber"></animated-integer> =
    <animated-integer :value="result"></animated-integer>
  </p>
</div>
```

```js
const app = Vue.createApp({
  data() {
    return {
      firstNumber: 20,
      secondNumber: 40
    }
  },
  computed: {
    result() {
      return this.firstNumber + this.secondNumber
    }
  }
})

app.component('animated-integer', {
  template: '<span>{{ fullValue }}</span>',
  props: {
    value: {
      type: Number,
      required: true
    }
  },
  data() {
    return {
      tweeningValue: 0
    }
  },
  computed: {
    fullValue() {
      return Math.floor(this.tweeningValue)
    }
  },
  methods: {
    tween(newValue, oldValue) {
      gsap.to(this.$data, {
        duration: 0.5,
        tweeningValue: newValue,
        ease: 'sine'
      })
    }
  },
  watch: {
    value(newValue, oldValue) {
      this.tween(newValue, oldValue)
    }
  },
  mounted() {
    this.tween(this.value, 0)
  }
})

app.mount('#app')
```

<common-codepen-snippet title="Компонент для управления переходом от состояния" slug="e9ef8ac7e32e0d0337e03d20949b4d17" tab="js,result" :editable="false" />

В дочерних компонентах можно использовать любую комбинацию стратегий переходов, упомянутых на этой странице, наряду со [встроенной системой переходов Vue](transitions-enterleave.md). Вместе они предоставляют практически безграничные возможности.

Как можно увидеть, нет никаких ограничений для использования — визуализация данных, физические эффекты, анимация персонажей и взаимодействий.

## Привнесение дизайна в жизнь

По одному из определений, анимация означает оживление. К сожалению, когда дизайнеры создают новые иконки, логотипы и талисманы, результаты обычно оказываются картинками или статичными SVG. Таким образом осьминожек в GitHub, птичка в Twitter и многие другие логотипы напоминают живых существ, но в действительности не кажутся живыми.

Vue может помочь. Поскольку SVG это всего лишь данные, нам нужны только примеры того, как выглядят эти существа когда они радуются, думают или встревожены. Затем Vue берёт на себя всю работу по реализации переходов между этими состояниями, помогая создавать более эмоциональные приветственные страницы, индикаторы загрузки и уведомления.

Сара Драснер (Sarah Drasner) демонстрирует это в демо ниже, используя комбинацию временных и интерактивных изменений состояния:

<common-codepen-snippet title="Vue-controlled Wall-E" slug="YZBGNp" :height="400" :team="false" user="sdras" name="Sarah Drasner" :editable="false" :preview="false" version="2" theme="light" />
