# Передача обычных атрибутов

> Подразумевается, что уже изучили и разобрались с разделом [Основы компонентов](component-basics.md). Если нет — прочитайте его сначала.

Обычный атрибут для компонента — атрибут или слушатель события, который передаётся в компонент, но не имеет соответствующего свойства указанного в [props](component-props.md) или [emits](component-custom-events.md#определение-пользовательских-событии). Частыми примерами подобного являются атрибуты `class`, `style` и `id`. Доступ к этим атрибутам можно получить через свойство `$attrs`.

## Наследование атрибутов

Если в компоненте один корневой элемент, то обычные атрибуты будут добавляться к этому элементу автоматически. Например, для экземпляра компонента выбора даты:

```js
app.component('date-picker', {
  template: `
    <div class="date-picker">
      <input type="datetime-local" />
    </div>
  `
})
```

Если потребуется определять статус компонента выбора даты через атрибут `data-status`, значение будет добавлено к корневому элементу (т.е. к `div.date-picker`).

```html
<!-- Компонент выбора даты с обычным атрибутом -->
<date-picker data-status="activated"></date-picker>

<!-- Отрисованный компонент выбора даты -->
<div class="date-picker" data-status="activated">
  <input type="datetime-local" />
</div>
```

Аналогично правило применяется и для обработчиков событий:

```html
<date-picker @change="submitChange"></date-picker>
```

```js
app.component('date-picker', {
  created() {
    console.log(this.$attrs) // { onChange: () => {}  }
  }
})
```

Это удобно, когда корневым элементом компонента `date-picker` будет HTML-элемент, генерирующий событие `change` .

```js
app.component('date-picker', {
  template: `
    <select>
      <option value="1">Вчера</option>
      <option value="2">Сегодня</option>
      <option value="3">Завтра</option>
    </select>
  `
})
```

Обработчик события `change` будет передан из родительского компонента в дочерний и станет вызываться при нативном событии `change` на корневом элементе `<select>`. При таком подходе не потребуется явно генерировать событие внутри `date-picker`:

```html
<div id="date-picker" class="demo">
  <date-picker @change="showChange"></date-picker>
</div>
```

```js
const app = Vue.createApp({
  methods: {
    showChange(event) {
      console.log(event.target.value) // выведет значение выбранного варианта
    }
  }
})
```

## Отключение наследования атрибутов

При **необходимости отключить** автоматическое наследование обычных атрибутов компонентом это можно сделать с помощью опции `inheritAttrs: false`.

Популярная причина, когда требуется отключать наследование атрибутов — необходимость добавления атрибутов на другой элемент вместо корневого.

Установив опцию `inheritAttrs` в `false`, можно управлять добавлением атрибутов на другие элементы компонента через свойство `$attrs`, которое будет содержать все атрибуты, не входящие в состав свойств `props` и `emits` (например, `class`, `style`, обработчики `v-on`, и т.д.).

Используя компонент для выбора дат из [предыдущей главы](#наследование-атрибутов), изменим добавление обычных атрибутов на элемент `input`, вместо корневого элемента `div`. Для этого воспользуемся сокращённой записью `v-bind`:

```js{2,5}
app.component('date-picker', {
  inheritAttrs: false,
  template: `
    <div class="date-picker">
      <input type="datetime-local" v-bind="$attrs" />
    </div>
  `
})
```

В таком случае обычный атрибут `data-status` будет добавляться на элемент `input`!

```html
<!-- Компонент выбора даты с обычным атрибутом -->
<date-picker data-status="activated"></date-picker>

<!-- Отрисованный компонент выбора даты -->
<div class="date-picker">
  <input type="datetime-local" data-status="activated" />
</div>
```

## Наследование атрибутов при нескольких корневых элементах

В отличие от компонентов с одним корневым элементом, для компонентов с несколькими корневыми элементами не будет автоматического наследования атрибутов. Если `$attrs` не привязать к элементу явно, то это приведёт к предупреждению во время выполнения.

```html
<custom-layout id="custom-layout" @click="changeValue"></custom-layout>
```

```js
// НЕПРАВИЛЬНО, будет выведено предупреждение
app.component('custom-layout', {
  template: `
    <header>...</header>
    <main>...</main>
    <footer>...</footer>
  `
})

// ПРАВИЛЬНО, $attrs передаются на элемент <main>
app.component('custom-layout', {
  template: `
    <header>...</header>
    <main v-bind="$attrs">...</main>
    <footer>...</footer>
  `
})
```
