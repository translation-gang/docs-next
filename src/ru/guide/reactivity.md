# Подробно о реактивность

Пришло время нырнуть поглубже в тему!  Одна из наиболее примечательных возможностей Vue — это ненавязчивая реактивность. Модели представляют собой проксированные JavaScript-объекты. По мере их изменения обновляется и представление данных. Благодаря этому управление состоянием приложения так просто и очевидно. Тем не менее, у механизма реактивности есть ряд особенностей, знакомство с которыми позволит избежать распространённых ошибок. В этом разделе руководства мы подробно рассмотрим низкоуровневую реализацию системы реактивности Vue.

## Что такое реактивность?

В последнее время этот термин часто используется в программировании, но что он значит? Реактивность - это концепция в программировании, которая позволяет нам приспособиться к изменениям декларативным способом. Отличный канонический пример, который обычно демонстрируется - это электронная таблица в Excel.

<video width="550" height="400" controls>
  <source src="/images/reactivity-spreadsheet.mp4" type="video/mp4">
  Ваш браузер не поддерживает тег video.
</video>

Если вы введёте цифру 2 в первую ячейку, цифру 3 во вторую и попробуете получить сумму с помощью встроенной в Excel функции SUM, таблица её посчитает. Ничего необычного. Но если вы обновите число в первой ячейке, сумма тоже автоматически обновится.

Обычно JavaScript так не работает. Если бы вы писали что-то подобное на JavaScript:

```js
var val1 = 2
var val2 = 3
var sum = val1 + val2

// сумма
// 5

val1 = 3

// сумма
// 5
```

Если мы обновим первое значение, сумма не скорректируется.

Итак, как бы мы это сделали в JavaScript?

- Определить, когда одно из значений изменяется
- Найти функцию, которая их изменяет
- Вызвать функцию, чтобы она обновила конечный результат

## Как Vue отслеживает изменения

Когда простой JavaScript-объект передаётся в приложение или экземпляр Vue в качестве опции `data`, Vue обходит все его свойства и превращает их в [Прокси](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy), используя обработчик с геттерами и сеттерами. Эта возможность присутствует только в ES6, но Vue 3 также поддерживает старый подход с `Object.defineProperty` для совместимости с браузерами IE. Обе версии используют одинаковое API верхнего уровня, но версия с Proxy более легковесная и производительная.

<div class="reactivecontent">
  <common-codepen-snippet title="Визуальное объяснение Proxy и реактивности во Vue" slug="zYYzjBg" tab="result" theme="light" :height="500" :team="false" user="sdras" name="Sarah Drasner" :editable="false" :preview="false" />
</div>

Это было довольно быстро и требует некоторых знаний о [Прокси](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) для полного понимания. Давайте немного углубимся. Есть достаточно много литературы про Прокси, но что вам действительно важно знать, что **Прокси это объект, который содержит в себе другой объект или функцию и позволяет "перехватить" их.**

Вот как мы это используем: `new Proxy(target, handler)`

```js
const dinner = {
  meal: 'tacos'
}

const handler = {
  get(target, prop) {
    return target[prop]
  }
}

const proxy = new Proxy(dinner, handler)
console.log(proxy.meal)

// tacos
```
Пока что мы просто оборачиваем объект и возвращаем его. Круто, но ещё не так полезно. Но взгляните сюда, мы также можем перехватить этот объект пока оборачиваем его в Прокси. Этот перехват называется ловушкой. 

```js
const dinner = {
  meal: 'tacos'
}

const handler = {
  get(target, prop) {
    console.log('перехвачен!')
    return target[prop]
  }
}

const proxy = new Proxy(dinner, handler)
console.log(proxy.meal)

// перехвачен!
// tacos
```

Beyond a console log, we could do anything here we wish. We could even _not_ return the real value if we wanted to. This is what makes Proxies so powerful for creating APIs.

Furthermore, there’s another feature Proxies offer us. Rather than just returning the value like this: `target[prop]`, we could take this a step further and use a feature called `Reflect`, which allows us to do proper `this` binding. It looks like this:

```js{7}
const dinner = {
  meal: 'tacos'
}

const handler = {
  get(target, prop, receiver) {
    return Reflect.get(...arguments)
  }
}

const proxy = new Proxy(dinner, handler)
console.log(proxy.meal)

// tacos
```

We mentioned before that in order to have an API that updates a final value when something changes, we’re going to have to set new values when something changes. We do this in the handler, in a function called `track`, where we pass in the `target` and `key`.

```js{7}
const dinner = {
  meal: 'tacos'
}

const handler = {
  get(target, prop, receiver) {
    track(target, prop)
    return Reflect.get(...arguments)
  }
}

const proxy = new Proxy(dinner, handler)
console.log(proxy.meal)

// tacos
```

Finally, we also set new values when something changes. For this, we’re going to set the changes on our new proxy, by triggering those changes:

```js
const dinner = {
  meal: 'tacos'
}

const handler = {
  get(target, prop, receiver) {
    track(target, prop)
    return Reflect.get(...arguments)
  },
  set(target, key, value, receiver) {
    trigger(target, key)
    return Reflect.set(...arguments)
  }
}

const proxy = new Proxy(dinner, handler)
console.log(proxy.meal)

// tacos
```

Remember this list from a few paragraphs ago? Now we have some answers to how Vue handles these changes:

- <strike>Detect when there’s a change in one of the values</strike>: we no longer have to do this, as Proxies allow us to intercept it
- **Track the function that changes it**: We do this in a getter within the proxy, called `effect`
- **Trigger the function so it can update the final value**: We do in a setter within the proxy, called `trigger`

The proxied object is invisible to the user, but under the hood they enable Vue to perform dependency-tracking and change-notification when properties are accessed or modified. As of Vue 3, our reactivity is now available in a [separate package](https://github.com/vuejs/vue-next/tree/master/packages/reactivity). One caveat is that browser consoles format differently when converted data objects are logged, so you may want to install [vue-devtools](https://github.com/vuejs/vue-devtools) for a more inspection-friendly interface.

### Proxied Objects

Vue internally tracks all objects that have been made reactive, so it always returns the same proxy for the same object.

When a nested object is accessed from a reactive proxy, that object is _also_ converted into a proxy before being returned:

```js
const handler = {
  get(target, prop, receiver) {
    track(target, prop)
    const value = Reflect.get(...arguments)
    if (isObject(value)) {
      return reactive(value)
    } else {
      return value
    }
  }
  // ...
}
```

### Proxy vs. original identity

The use of Proxy does introduce a new caveat to be aware with: the proxied object is not equal to the original object in terms of identity comparison (`===`). For example:

```js
const obj = {}
const wrapped = new Proxy(obj, handlers)

console.log(obj === wrapped) // false
```

The original and the wrapped version will behave the same in most cases, but be aware that they will fail
operations that rely on strong identity comparisons, such as `.filter()` or `.map()`. This caveat is unlikely to come up when using the options API, because all reactive state is accessed from `this` and guaranteed to already be proxies.

However, when using the composition API to explicitly create reactive objects, the best practice is to never hold a reference to the original raw object and only work with the reactive version:

```js
const obj = reactive({
  count: 0
}) // no reference to original
```

## Watchers

Every component instance has a corresponding watcher instance, which records any properties "touched" during the component’s render as dependencies. Later on when a dependency’s setter is triggered, it notifies the watcher, which in turn causes the component to re-render.

<div class="reactivecontent">
  <common-codepen-snippet title="Second Reactivity with Proxies in Vue 3 Explainer" slug="GRJZddR" tab="result" theme="light" :height="500" :team="false" user="sdras" name="Sarah Drasner" :editable="false" :preview="false" />
</div>

When you pass an object to a component instance as data, Vue converts it to a proxy. This proxy enables Vue to perform dependency-tracking and change-notification when properties are accessed or modified. Each property is considered a dependency.

After the first render, a component would have tracked a list of dependencies &mdash; the properties it accessed during the render. Conversely, the component becomes a subscriber to each of these properties. When a proxy intercepts a set operation, the property will notify all of its subscribed components to re-render.

<!-- [//]: # 'TODO: Insert diagram' -->

> If you are using Vue 2.x and below, you may be interested in some of the change detection caveats that exist for those versions, [explored in more detail here](change-detection.md).
