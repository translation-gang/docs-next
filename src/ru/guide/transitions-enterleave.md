# Анимация появления и исчезновения элемента

Vue предоставляет несколько способов для анимации эффектов переходов при добавлении элементов в DOM, обновлении или удалении из него. Эти возможности позволяют:

- автоматически применять CSS-классы для переходов и анимаций
- интегрировать сторонние библиотеки CSS-анимаций, такие как [Animate.css](https://animate.style/)
- использовать JavaScript для манипуляции DOM напрямую с помощью хуков
- интегрировать сторонние библиотеки JavaScript-анимаций

В этом разделе рассматриваются только анимации переходов появления и исчезновения элементов. В следующих — [анимация списков](transitions-list.md) и [анимация переходов между состояниями](transitions-state.md).

## Анимация одиночного элемента/компонента

Vue предоставляет компонент-обёртку `<transition>`, позволяющий добавить анимации появления/исчезновения для любого элемента или компонента для следующих случаев:

- Условная отрисовка (с использованием `v-if`)
- Условное отображение (с использованием `v-show`)
- Динамические компоненты
- Корневые элементы компонента

Посмотрим на небольшой пример в действии:

```html
<div id="demo">
  <button @click="show = !show">
    Переключить
  </button>

  <transition name="fade">
    <p v-if="show">привет</p>
  </transition>
</div>
```

```js
const Demo = {
  data() {
    return {
      show: true
    }
  }
}

Vue.createApp(Demo).mount('#demo')
```

```css
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
```

<common-codepen-snippet title="Простое использование компонента Transition" slug="3466d06fb252a53c5bc0a0edb0f1588a" tab="html,result" :editable="false" />

Когда элемент, обёрнутый в компонент `<transition>`, будет добавляться или удаляться, произойдут следующие действия:

1. Vue автоматически определит, применяются ли к целевому элементу CSS-переходы или анимации. Если да, то в соответствующие моменты времени будут добавляться и удаляться классы CSS-переходов.

2. Если на компоненте есть [JavaScript-хуки](#javascript-хуки), то в соответствующие моменты времени они будут вызваны.

3. Если CSS-переходов или анимаций не обнаружено и нет JavaScript-хуков, DOM-операции для вставки и/или удаления будут вызваны незамедлительно или в следующем фрейме (примечание: это фрейм анимации браузера, отличный от концепции `nextTick` во Vue).

### Классы переходов

Существует шесть классов для анимаций переходов появления и исчезновения элементов.

1. `v-enter-from`: Означает начало анимации появления элемента. Этот класс добавляется перед вставкой элемента, а удаляется в следующем фрейме после вставки.

2. `v-enter-active`: Означает активное состояние анимации появления элемента. Этот класс остаётся на элементе в течение всей анимации появления. Он добавляется перед вставкой элемента, а удаляется после завершения перехода или анимации. Этот класс можно использовать для установки длительности, задержки или функции плавности (easing curve) анимации появления.

3. `v-enter-to`: Означает завершение анимации появления элемента. Добавляется в следующем фрейме после вставки элемента (тогда же удаляется `v-enter-from`), а удаляется после завершения перехода или анимации.

4. `v-leave-from`: Означает начало анимации исчезновения элемента. Добавляется сразу после вызова анимации исчезновения, а удаляется в следующем фрейме после этого.

5. `v-leave-active`: Означает активное состояние анимации исчезновения. Этот класс остаётся на элементе в течение всей анимации исчезновения. Он добавляется при вызове анимации исчезновения, а удаляется после завершения перехода или анимации. Этот класс можно использовать для установки длительности, задержки или функции плавности (easing curve) анимации исчезновения.

6. `v-leave-to`: Означает завершение анимации исчезновения элемента. Добавляется в следующем фрейме после вызова анимации исчезновения (тогда же удаляется `v-leave-from`), а удаляется после завершения перехода или анимации.

![Диаграмма переходов](/images/transitions.svg)

Каждый класс имеет префикс с именем перехода. По умолчанию будет префикс `v-`, если в `<transition>` не указан атрибут `name`. Например, для `<transition name="super">` вместо класса `v-enter-from` будет применяться `super-enter-from`.

Классы `v-enter-active` и `v-leave-active` позволяют определить кривые плавности для переходов появления и исчезновения элемента. Пример использования рассмотрим ниже.

### CSS-переходы

Один из наиболее распространённых типов анимации — CSS-переходы:

```html
<div id="demo">
  <button @click="show = !show">
    Переключить отображение
  </button>

  <transition name="slide-fade">
    <p v-if="show">привет</p>
  </transition>
</div>
```

```js
const Demo = {
  data() {
    return {
      show: true
    }
  }
}

Vue.createApp(Demo).mount('#demo')
```

```css
/* Анимации появления и исчезновения могут иметь    */
/* различные продолжительности и функции плавности. */
.slide-fade-enter-active {
  transition: all 0.3s ease-out;
}

.slide-fade-leave-active {
  transition: all 0.8s cubic-bezier(1, 0.5, 0.8, 1);
}

.slide-fade-enter-from,
.slide-fade-leave-to {
  transform: translateX(20px);
  opacity: 0;
}
```

<common-codepen-snippet title="Различные переходы для появления и исчезновения" slug="0dfa7869450ef43d6f7bd99022bc53e2" tab="css,result" :editable="false" />

### CSS-анимации

CSS-анимации применяются таким же образом, что и CSS-переходы. Отличие лишь в том, что `v-enter-from` удаляется не сразу после вставки элемента, а по событию `animationend`.

Небольшой пример (браузерные CSS-префиксы опущены для краткости):

```html
<div id="demo">
  <button @click="show = !show">Переключить отображение</button>
  <transition name="bounce">
    <p v-if="show">
      Lorem ipsum dolor sit amet, consectetur adipiscing elit. Mauris facilisis
      enim libero, at lacinia diam fermentum id. Pellentesque habitant morbi
      tristique senectus et netus.
    </p>
  </transition>
</div>
```

```js
const Demo = {
  data() {
    return {
      show: true
    }
  }
}

Vue.createApp(Demo).mount('#demo')
```

```css
.bounce-enter-active {
  animation: bounce-in 0.5s;
}
.bounce-leave-active {
  animation: bounce-in 0.5s reverse;
}
@keyframes bounce-in {
  0% {
    transform: scale(0);
  }
  50% {
    transform: scale(1.25);
  }
  100% {
    transform: scale(1);
  }
}
```

<common-codepen-snippet title="Пример перехода с помощью CSS-анимации" slug="8627c50c5514752acd73d19f5e33a781" tab="html,result" :editable="false" />

### Пользовательские классы для переходов

Можно указать пользовательские классы для переходов с помощью следующих атрибутов:

- `enter-from-class`
- `enter-active-class`
- `enter-to-class`
- `leave-from-class`
- `leave-active-class`
- `leave-to-class`

Таким образом можно переопределить стандартные названия классов. Например, при необходимости комбинировать систему анимаций Vue с возможностями сторонних библиотек CSS-анимаций, таких как [Animate.css](https://daneden.github.io/animate.css/).

Пример переопределения стандартных имён классов:

```html
<link
  href="https://cdnjs.cloudflare.com/ajax/libs/animate.css/4.1.0/animate.min.css"
  rel="stylesheet"
  type="text/css"
/>

<div id="demo">
  <button @click="show = !show">
    Переключить отображение
  </button>

  <transition
    name="custom-classes-transition"
    enter-active-class="animate__animated animate__tada"
    leave-active-class="animate__animated animate__bounceOutRight"
  >
    <p v-if="show">привет</p>
  </transition>
</div>
```

```js
const Demo = {
  data() {
    return {
      show: true
    }
  }
}

Vue.createApp(Demo).mount('#demo')
```

### Совместное использование переходов и анимаций

Чтобы определить завершение анимации, Vue прослушивает события `transitionend` или `animationend`, в зависимости от типа применяемых CSS-правил. Если используется только один подход из них — Vue автоматически определит правильный тип события.

Но в некоторых случах может потребоваться использовать оба подхода на одном элементе. Например CSS-анимация, запущенная Vue, может соседствовать с эффектом CSS-перехода при наведении на элемент. Для таких случаев потребуется явно указать тип события, на которое должен ориентироваться Vue. Для этого потребуется установить атрибут `type` со значением `animation` или `transition`.

### Указание длительности перехода

<!-- TODO: validate and provide an example -->

Vue может автоматически определить завершение перехода для большинства случаев. По умолчанию Vue дожидается на корневом элементе первого события `transitionend` или `animationend`. Но такое поведение не всегда нужно — например, может потребоваться некая хореографическая последовательность переходов, в которой часть вложенных элементов могут иметь задержку перед началом перехода или другую продолжительность, отличную от корневого элемента.

В таких случаях можно явно задать продолжительность перехода (в миллисекундах) через входной параметр `duration` на компоненте `<transition>`:

```html
<transition :duration="1000">...</transition>
```

Также можно устанавливать раздельные значения для анимаций появления и исчезновения:

```html
<transition :duration="{ enter: 500, leave: 800 }">...</transition>
```

### JavaScript-хуки

На компоненте также можно указывать JavaScript-хуки:

```html
<transition
  @before-enter="beforeEnter"
  @enter="enter"
  @after-enter="afterEnter"
  @enter-cancelled="enterCancelled"
  @before-leave="beforeLeave"
  @leave="leave"
  @after-leave="afterLeave"
  @leave-cancelled="leaveCancelled"
  :css="false"
>
  <!-- ... -->
</transition>
```

```js
// ...
methods: {
  // ---------
  // ПОЯВЛЕНИЕ
  // ---------

  beforeEnter(el) {
    // ...
  },
  // коллбэк done опционален
  // при использовании в комбинации с CSS
  enter(el, done) {
    // ...
    done()
  },
  afterEnter(el) {
    // ...
  },
  enterCancelled(el) {
    // ...
  },

  // ------------
  // ИСЧЕЗНОВЕНИЕ
  // ------------

  beforeLeave(el) {
    // ...
  },
  // коллбэк done опционален
  // при использовании в комбинации с CSS
  leave(el, done) {
    // ...
    done()
  },
  afterLeave(el) {
    // ...
  },
  // хук leaveCancelled доступен только для v-show
  leaveCancelled(el) {
    // ...
  }
}
```

Использовать хуки можно как самостоятельно, так и в сочетании с CSS-переходами и анимациями.

Если анимация реализуется только на JavaScript — **обязательно вызывайте коллбэки `done` в хуках `enter` и `leave`**. Если этого не сделать, то хуки будут вызваны синхронно и переход сразу закончится. Также можно помочь Vue не тратить время на обнаружение CSS указав `:css="false"`. Кроме бонуса в производительности, это предотвратит случайное влияние друг на друга CSS-правил и JS-перехода.

Пример JavaScript-перехода с использованием [GreenSock](https://greensock.com/):

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.3.4/gsap.min.js"></script>

<div id="demo">
  <button @click="show = !show">
    Переключить отображение
  </button>

  <transition
    @before-enter="beforeEnter"
    @enter="enter"
    @leave="leave"
    :css="false"
  >
    <p v-if="show">
      Демо
    </p>
  </transition>
</div>
```

```js
const Demo = {
  data() {
    return {
      show: false
    }
  },
  methods: {
    beforeEnter(el) {
      gsap.set(el, {
        scaleX: 0.8,
        scaleY: 1.2
      })
    },
    enter(el, done) {
      gsap.to(el, {
        duration: 1,
        scaleX: 1.5,
        scaleY: 0.7,
        opacity: 1,
        x: 150,
        ease: 'elastic.inOut(2.5, 1)',
        onComplete: done
      })
    },
    leave(el, done) {
      gsap.to(el, {
        duration: 0.7,
        scaleX: 1,
        scaleY: 1,
        x: 300,
        ease: 'elastic.inOut(2.5, 1)'
      })
      gsap.to(el, {
        duration: 0.2,
        delay: 0.5,
        opacity: 0,
        onComplete: done
      })
    }
  }
}

Vue.createApp(Demo).mount('#demo')
```

<common-codepen-snippet title="Переход с использованием JavaScript-хуков" slug="68ce1b8c41d0a6e71ff58df80fd85ae5" tab="js,result" :editable="false" />

## Анимация первоначальной отрисовки

Если необходимо применить анимацию перехода при первоначальной отрисовке элемента, то её можно добавить с помощью атрибута `appear`:

```html
<transition appear>
  <!-- ... -->
</transition>
```

## Анимация перехода между элементами

[Анимация перехода между компонентами](#анимация-перехода-между-компонентами) будет рассмотрена ниже, но создавать переходы можно также и между обычными элементами с помощью `v-if`/`v-else`. Один из наиболее распространённых случаев перехода между двумя элементами — между контейнером со списком элементов и сообщением об их отсутствии в нём:

```html
<transition>
  <table v-if="items.length > 0">
    <!-- ... -->
  </table>
  <p v-else>Нет элементов</p>
</transition>
```

Фактически, переход возможен между любым количеством элементов, используя `v-if`/`v-else-if`/`v-else` или с помощью привязки элемента к динамическому свойству. Например:

<!-- TODO: rewrite example and put in codepen example -->

```html
<transition>
  <button v-if="docState === 'saved'" key="saved">
    Изменить
  </button>
  <button v-else-if="docState === 'edited'" key="edited">
    Сохранить
  </button>
  <button v-else-if="docState === 'editing'" key="editing">
    Отмена
  </button>
</transition>
```

Что также можно записать и следующим образом:

```html
<transition>
  <button :key="docState">
    {{ buttonMessage }}
  </button>
</transition>
```

```js
// ...
computed: {
  buttonMessage() {
    switch (this.docState) {
      case 'saved': return 'Изменить'
      case 'edited': return 'Сохранить'
      case 'editing': return 'Отмена'
    }
  }
}
```

### Режимы переходов

Однако сохраняется одна проблема. Попробуйте в примере ниже кликнуть на кнопку:

<common-codepen-snippet title="Проблема с кнопками и режимы переходов" slug="Rwrqzpr" :editable="false" />

Во время анимации перехода от кнопки «on» к кнопке «off» одновременно будут видны обе кнопки: одна — исчезая, другая — появляясь. Это стандартное поведение `<transition>` — анимации появления и исчезновения происходят одновременно.

Иногда такое поведение подходит, например когда обе кнопки позиционируются абсолютно друг над другом:

<common-codepen-snippet title="Проблема с кнопками и режимы переходов - позиционирование" slug="abdQgLr" :editable="false" />

Но иногда это не вариант, когда имеем дело с более сложными случаями, где состояния появления и исчезновения должны быть скоординированы. Для таких ситуаций Vue предоставляет возможность указать **режима перехода**:

- `in-out`: Сначала появляется новый элемент и только после этого исчезает старый.
- `out-in`: Сначала исчезает старый элемент и только после этого появляется новый.

:::tip Совет
Очень быстро поймёте, что `out-in` — то, что требуется чаще всего :)
:::

Доработаем пример для кнопок «on»/«off» с использованием режима перехода `out-in`:

```html
<transition name="fade" mode="out-in">
  <!-- ... кнопки ... -->
</transition>
```

<common-codepen-snippet title="Проблема с кнопками и режимы переходов - решение" slug="ZEQmdvq" :editable="false" />

Добавив один атрибут, получилось исправить работу анимации перехода и избежать необходимости указывать стили специально для решения этой проблемы.

Их можно использовать и для координации более выразительных движений, к примеру переворачивания карты (см. пример ниже). На самом деле это два элемента, переходящие друг в друга, но так как начальное и конечное состояние одинаковы: по горизонтали до 0, то похоже на одно плавное движение. Это может пригодиться для создания реалистичных микро-взаимодействий в пользовательском интерфейсе:

<common-codepen-snippet title="Режимы переходов: переворачивающиеся карты" slug="76e344bf057bd58b5936bba260b787a8" :editable="false" />

## Анимация перехода между компонентами

Анимация перехода между компонентами ещё проще — не требуется даже атрибут `key`. Всё, что потребуется — обернуть [динамический компонент](component-basics.md#динамическое-переключение-компонентов) в `<transition>`:

```html
<div id="demo">
  <input v-model="view" type="radio" value="v-a" id="a"><label for="a">A</label>
  <input v-model="view" type="radio" value="v-b" id="b"><label for="b">B</label>
  <transition name="component-fade" mode="out-in">
    <component :is="view"></component>
  </transition>
</div>
```

```js
const Demo = {
  data() {
    return {
      view: 'v-a'
    }
  },
  components: {
    'v-a': {
      template: '<div>Компонент А</div>'
    },
    'v-b': {
      template: '<div>Компонент B</div>'
    }
  }
}

Vue.createApp(Demo).mount('#demo')
```

```css
.component-fade-enter-active,
.component-fade-leave-active {
  transition: opacity 0.3s ease;
}

.component-fade-enter-from,
.component-fade-leave-to {
  opacity: 0;
}
```

<common-codepen-snippet title="Transitioning between components" slug="WNwVxZw" tab="html,result" theme="39028" />
