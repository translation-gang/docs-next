# Обзор возможностей

Vue предоставляет несколько абстракций, которые могут помочь в работе с переходами и анимациями, особенно в качестве реакции на изменения чего-либо. Вот некоторые из них:

- Хуки для компонентов, которые добавляются в DOM или удаляются из него, как с использованием CSS, так и JS, с помощью встроенного компонента `<transition>`.
- Управление режимом перехода для оркестрации очерёдности во время перехода.
- Хуки для списка элементов, чьё расположение на странице будет изменяться, используя технику FLIP под капотом для увеличения производительности, с помощью компонента `<transition-group>`.
- Переходы с помощью `watchers` между различными состояниями приложения.

Всё это (и не только) будет рассматриваться в трёх следующих разделах руководства. Но несмотря на эти удобные API, следует запомнить, что обычные привязки классов и стилей, изученные ранее, можно легко использовать для анимаций и переходов в простых случаях.

В следующем разделе познакомимся с основами веб-анимаций и переходов и предоставим ссылки на ресурсы для дальнейшего изучения. Его можно пропустить, если уже знакомы с веб-анимациями и тем, как эти принципы могут работать с директивами Vue. Для тех, кому интересно узнать немного больше об основах веб-анимаций рекомендуем читать дальше.

## Использование классов для анимаций и переходов

Хоть и компонент `<transition>` отлично подходит для анимаций появления и исчезновения компонентов, анимацию можно активировать и без использования компонента — просто добавляя класс по условию.

```html
<div id="demo">
  Нажмите кнопку, чтобы сделать то, чего не должны делать:<br />

  <div :class="{ shake: noActivated }">
    <button @click="noActivated = true">Нажми меня</button>
    <span v-if="noActivated">О, нет!</span>
  </div>
</div>
```

```css
.shake {
  animation: shake 0.82s cubic-bezier(0.36, 0.07, 0.19, 0.97) both;
  transform: translate3d(0, 0, 0);
  backface-visibility: hidden;
  perspective: 1000px;
}

@keyframes shake {
  10%,
  90% {
    transform: translate3d(-1px, 0, 0);
  }

  20%,
  80% {
    transform: translate3d(2px, 0, 0);
  }

  30%,
  50%,
  70% {
    transform: translate3d(-4px, 0, 0);
  }

  40%,
  60% {
    transform: translate3d(4px, 0, 0);
  }
}
```

```js
const Demo = {
  data() {
    return {
      noActivated: false
    }
  }
}

Vue.createApp(Demo).mount('#demo')
```

<common-codepen-snippet title="Создание анимации с помощью класса" slug="ff45b91caf7a98c8c9077ad8ab539260" tab="css,result" :editable="false" :preview="false" />

## Использование привязок стилей для переходов

Некоторые эффекты переходов можно реализовать подстановкой изменяемых значений, например, во время взаимодействия привязывая определённые стили к элементам:

```html
<div id="demo">
  <div
    @mousemove="xCoordinate"
    :style="{ backgroundColor: `hsl(${x}, 80%, 50%)` }"
    class="movearea"
  >
    <h3>Перемещайте курсор по экрану...</h3>
    <p>x: {{x}}</p>
  </div>
</div>
```

```css
.movearea {
  transition: 0.2s background-color ease;
}
```

```js
const Demo = {
  data() {
    return {
      x: 0
    }
  },
  methods: {
    xCoordinate(e) {
      this.x = e.clientX
    }
  }
}

Vue.createApp(Demo).mount('#demo')
```

<common-codepen-snippet title="Использование привязок стилей" slug="JjGezQY" :editable="false" />

Анимация в примере создана с помощью интерполяции, которая привязана к перемещению курсора мыши. CSS-переход применяется и к элементу, чтобы при обновлении элемента было понятно, какую функцию плавности нужно использовать для перехода.

## Производительность

Можно отметить, что в примерах анимации выше используются такие вещи как `transform` и применяются странные свойства, например `perspective` — зачем же они нужны, и почему всё реализовано таким образом, а не с помощью свойств `margin` и `top` и т.п.?

Для созданий плавных анимаций необходимо не забывать о производительности. Стоит использовать аппаратное ускорение для элементов, где это только возможно и выбирать свойства, которые не будут приводить к перерисовкам (repaints). Рассмотрим подробнее как этого можно достичь.

### Трансформации и прозрачность

Можно обращаться к специальным ресурсам, таким как [CSS-Triggers](https://csstriggers.com/), чтобы определять какие свойства приводят к перерисовке страницы (repaint), если их анимировать. Например, если посмотрим что указано для `transform`, то увидим следующее:

> Применение трансформаций не вызывает никаких изменений геометрии или отрисовки, что очень хорошо. Это означает, что операция скорее всего может быть выполнена в compositor thread с поддержкой GPU.

Свойство `opacity` ведёт себя аналогичным образом. Поэтому они являются идеальными кандидатами для реализации анимаций перемещений.

### Аппаратное ускорение

Свойства `perspective`, `backface-visibility` и `transform: translateZ(x)` подсказывают браузеру, когда необходимо использовать аппаратное ускорение.

Если требуется аппаратное ускорение для элемента, то можно применить любое из этих свойств (только одно, не все сразу):

```css
perspective: 1000px;
backface-visibility: hidden;
transform: translateZ(0);
```

Многие JS-библиотеки, такие как GreenSock, предполагают что аппаратное ускорение нужно всегда и применяют его по умолчанию, поэтому включать вручную не потребуется.

## Тайминги

Анимация перехода в интерфейсе чаще всего означает переход из одного состояния в другое без каких-либо промежуточных состояний, а тайминг в таких случаях где-то между 0.1s и 0.4s, в большинстве случаев _0.25s_. Можно ли использовать этот тайминг для всего? Нет, не стоит. Если есть что-то, чему нужно двигаться на большие расстояния или пройти больше шагов или смен состояний, то далеко не всегда 0.25s будут смотреться уместно, поэтому потребуется детально определить уникальные тайминги. Но это не значит, что не видать хороших настроек по умолчанию, которые можно переиспользовать в приложении.

Также отметим, что появление обычно лучше выглядит с чуть большей длительностью, нежели исчезновение. Пользователь, как правило, на этапе появления направляется, но немного нетерпелив на этапе исчезновения, потому что желает идти дальше своим путём.

## Функции плавности

Функция плавности (easing) — важный способ передачи глубины анимации. Наиболее частая ошибка у новичков — использовать `ease-in` для появлений и `ease-out` для исчезновений. На самом деле, лучше всё надо сделать наоборот.

Если к элементу применить эти состояния перехода, то будет выглядеть примерно так:

```css
.button {
  background: #1b8f5a;
  /* применяется к начальному состоянию, поэтому */
  /* этот переход будет виден при возвращении к этому состоянию */
  transition: background 0.25s ease-in;
}

.button:hover {
  background: #3eaf7c;
  /* применяется к состоянию при наведении, поэтому */
  /* этот переход будет применяться при срабатывании hover */
  transition: background 0.35s ease-out;
}
```

<common-codepen-snippet title="Пример функций плавности для перехода" slug="996a9665131e7902327d350ca8a655ac" tab="css,result" :editable="false" :preview="false" />

С помощью функции плавности анимации можно также передать свойства материала. Возьмём песочницу с примером, какой из шариков кажется твёрдым, а какой мягким?

<common-codepen-snippet title="Демо с подпрыгивающим шариком" slug="wvgqyyW" :height="500" :editable="false" />

Можно получить множество уникальных эффектов и сделать анимацию очень стильной, управляя параметрами функции плавности. CSS позволяет сделать это с помощью свойства cubic-bezier, а [эта песочница](https://cubic-bezier.com/#.17,.67,.83,.67), созданная Lea Verou, будет очень полезна для изучения на практике.

Несмотря на то, что можно достичь больших успехов в простых анимациях, управляя двумя точками функций плавности cubic-bezier, JavaScript предоставит несравненно больше точек для управления, а, следовательно, и большие возможности и вариативность в результате.

![Сравнение функций плавности](/images/css-vs-js-ease.svg)

Возьмём для примера анимацию прыжка. В CSS нужно объявить каждый кадр (keyframe), верх и низ. В JavaScript все перемещения можно выразить с помощью готовой функции анимации, передав `bounce` в [GreenSock API (GSAP)](https://greensock.com/) (в других JS-библиотеках также есть свои варианты анимаций по умолчанию).

Код для прыжка на CSS (пример из animate.css):

```css
@keyframes bounceInDown {
  from,
  60%,
  75%,
  90%,
  to {
    animation-timing-function: cubic-bezier(0.215, 0.61, 0.355, 1);
  }

  0% {
    opacity: 0;
    transform: translate3d(0, -3000px, 0) scaleY(3);
  }

  60% {
    opacity: 1;
    transform: translate3d(0, 25px, 0) scaleY(0.9);
  }

  75% {
    transform: translate3d(0, -10px, 0) scaleY(0.95);
  }

  90% {
    transform: translate3d(0, 5px, 0) scaleY(0.985);
  }

  to {
    transform: translate3d(0, 0, 0);
  }
}

.bounceInDown {
  animation-name: bounceInDown;
}
```

Аналогичная реализация в JS с использованием GreenSock:

```js
gsap.from(element, { duration: 1, ease: 'bounce.out', y: -500 })
```

Некоторые примеры в следующих разделах будут использовать GreenSock. Также есть отличный [визуализатор анимаций](https://greensock.com/ease-visualizer), который поможет в создании функций для анимаций.

## Дальнейшее изучение

- [Designing Interface Animation: Improving the User Experience Through Animation by Val Head](https://www.amazon.com/dp/B01J4NKSZA/)
- [Animation at Work by Rachel Nabors](https://abookapart.com/products/animation-at-work)
