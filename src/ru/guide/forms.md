# Работа с формами

<VideoLesson href="https://vueschool.io/lessons/user-inputs-vue-devtools-in-vue-3?friend=vuejs" title="Узнайте как работать с формами на Vue School">Узнайте как работать с формами в бесплатном уроке Vue School</VideoLesson>

## Обычное использование

Можно использовать директиву `v-model` для создания двусторонней привязки данных с элементами форм: input, textarea и select. Она автоматически выбирает правильный способ обновления элемента в зависимости от его типа. Хоть `v-model` и выглядит как некая магия, по сути это лишь синтаксический сахар для обновления данных при вводе пользователем, плюс небольшие доработки для некоторых граничных случаев.

:::tip Примечание
`v-model` игнорирует начальное значение атрибутов `value`, `checked` или `selected` на любых элементах форм. Источником истины всегда считаются данные текущего активного экземпляра. Начальное значение нужно объявить на стороне JavaScript, внутри опции `data` компонента.
:::

Внутренне `v-model` использует разные свойства и генерирует разные события для различных элементов форм:

- элементы для ввода текста и многострочного текста используют свойство `value` и событие `input`;
- чекбоксы и радиокнопки используют свойство `checked` и событие `change`;
- выпадающие списки используют свойство `value` и событие `change`.

<span id="vmodel-ime-tip"></span>
:::tip Примечание
Для языков, которые требуют [IME](https://ru.wikipedia.org/wiki/IME) (китайский, японский, корейский и т.д.), можно заметить, что `v-model` не обновляется во время IME-композиции. Если необходимо обрабатывать и эти обновления, используйте слушатель события `input` и привязку к `value` вместо использования `v-model`.
:::

### Текст

```html
<input v-model="message" placeholder="отредактируй меня" />
<p>Сообщение: {{ message }}</p>
```

<common-codepen-snippet title="Работа с формами: обычное использование v-model" slug="eYNPEqj" :preview="false" />

### Многострочный текст

```html
<span>Многострочное сообщение:</span>
<p style="white-space: pre-line;">{{ message }}</p>
<br />
<textarea v-model="message" placeholder="введите несколько строчек"></textarea>
```

<common-codepen-snippet title="Работа с формами: многострочный текст" slug="xxGyXaG" :preview="false" />

Интерполяция внутри textarea **не работают**. Используйте директиву `v-model` вместо неё.

```html
<!-- НЕ РАБОТАЕТ -->
<textarea>{{ text }}</textarea>

<!-- ok -->
<textarea v-model="text"></textarea>
```

### Чекбоксы

Один чекбокс, привязанный к булевому значению:

```html
<input type="checkbox" id="checkbox" v-model="checked" />
<label for="checkbox">{{ checked }}</label>
```

<common-codepen-snippet title="Работа с формами: чекбоксы" slug="PoqyJVE" :preview="false" />

Список чекбоксов, привязанных к одному массиву:

```html
<div id="v-model-multiple-checkboxes">
  <input type="checkbox" id="jack" value="Джек" v-model="checkedNames" />
  <label for="jack">Джек</label>
  <input type="checkbox" id="john" value="Джон" v-model="checkedNames" />
  <label for="john">Джон</label>
  <input type="checkbox" id="mike" value="Майк" v-model="checkedNames" />
  <label for="mike">Майк</label>
  <br />
  <span>Отмеченные имена: {{ checkedNames }}</span>
</div>
```

```js
Vue.createApp({
  data() {
    return {
      checkedNames: []
    }
  }
}).mount('#v-model-multiple-checkboxes')
```

<common-codepen-snippet title="Работа с формами: список чекбоксов" slug="bGdmoyj" :preview="false" />

### Радиокнопки

```html
<div id="v-model-radiobutton">
  <input type="radio" id="one" value="Один" v-model="picked" />
  <label for="one">Один</label>
  <br />
  <input type="radio" id="two" value="Два" v-model="picked" />
  <label for="two">Два</label>
  <br />
  <span>Выбрано: {{ picked }}</span>
</div>
```

```js
Vue.createApp({
  data() {
    return {
      picked: ''
    }
  }
}).mount('#v-model-radiobutton')
```

<common-codepen-snippet title="Работа с формами: радиокнопки" slug="MWwPEMM" :preview="false" />

### Выпадающие списки

Выбор одного варианта из списка:

```html
<div id="v-model-select" class="demo">
  <select v-model="selected">
    <option disabled value="">Выберите один из вариантов</option>
    <option>А</option>
    <option>Б</option>
    <option>В</option>
  </select>
  <span>Выбрано: {{ selected }}</span>
</div>
```

```js
Vue.createApp({
  data() {
    return {
      selected: ''
    }
  }
}).mount('#v-model-select')
```

<common-codepen-snippet title="Работа с формами: выпадающие списки" slug="KKpGydL" :preview="false" />

:::tip Примечание
Если начальное значение выражения `v-model` не соответствует ни одному из вариантов списка, элемент `<select>` будет отображаться в «невыбранном» состоянии. В iOS это приведёт к тому, что пользователь не сможет выбрать первый элемент, потому что iOS не сгенерирует событие `change` в этом случае. Поэтому рекомендуется добавлять отключённый `disabled`-вариант выбора с пустым значением value, как показано в примере выше.
:::

Выбор нескольких вариантов из списка (с привязкой к массиву):

```html
<select v-model="selected" multiple>
  <option>А</option>
  <option>Б</option>
  <option>В</option>
</select>
<br />
<span>Выбраны: {{ selected }}</span>
```

<common-codepen-snippet title="Работа с формами: выпадающий список с привязкой к массиву" slug="gOpBXPz" tab="result" :preview="false" />

Динамическое отображение списка опций с помощью `v-for`:

```html
<div id="v-model-select-dynamic" class="demo">
  <select v-model="selected">
    <option v-for="option in options" :value="option.value">
      {{ option.text }}
    </option>
  </select>
  <span>Выбрано: {{ selected }}</span>
</div>
```

```js
Vue.createApp({
  data() {
    return {
      selected: 'A',
      options: [
        { text: 'Один', value: 'A' },
        { text: 'Два', value: 'B' },
        { text: 'Три', value: 'C' }
      ]
    }
  }
}).mount('#v-model-select-dynamic')
```

<common-codepen-snippet title="Работа с формами: выпадающий список с динамическими опциями" slug="abORVZm" :preview="false" />

## Привязка значений

Для радиокнопок и выпадающих списков в качестве привязываемых значений `v-model` обычно будут статические строки (или булево для чекбокса):

```html
<!-- `picked` будет строкой "a" при выборе -->
<input type="radio" v-model="picked" value="a" />

<!-- `toggle` может принимать значение true или false -->
<input type="checkbox" v-model="toggle" />

<!-- `selected` будет строкой "abc" при выборе первого пункта -->
<select v-model="selected">
  <option value="abc">ABC</option>
</select>
```

Но иногда может потребоваться привязать значение к динамическому свойству текущего активного экземпляра. Для этого можно использовать `v-bind`. Кроме того, использование `v-bind` позволяет привязать значение поля с нестроковыми значениями.

### Чекбокс

```html
<input type="checkbox" v-model="toggle" true-value="да" false-value="нет" />
```

```js
// когда чекбокс выбран
vm.toggle === 'да'

// когда чекбокс сброшен
vm.toggle === 'нет'
```

:::tip Совет
Атрибуты `true-value` и `false-value` не влияют на атрибут `value` элемента input, потому что браузеры пропускают невыбранные чекбоксы при отправке форм. Чтобы гарантировать отправку одного из двух значений с формой (например, «да» или «нет») используйте радиокнопки.
:::

### Радиокнопки

```html
<input type="radio" v-model="pick" :value="a" />
```

```js
// когда выбран
vm.pick === vm.a
```

### Выпадающие списки

```html
<select v-model="selected">
  <!-- инлайновый объект с данными -->
  <option :value="{ number: 123 }">123</option>
</select>
```

```js
// когда выбран
typeof vm.selected // => 'object'
vm.selected.number // => 123
```

## Модификаторы

### `.lazy`

По умолчанию `v-model` синхронизирует поле ввода с данными по событию `input` (кроме [вышеупомянутых](#vmodel-ime-tip)  исключений для событий IME). Можно указать модификатор `lazy`, чтобы синхронизация происходила после события `change`:

```html
<!-- синхронизация после события "change" вместо "input" -->
<input v-model.lazy="msg" />
```

### `.number`

Для автоматического приведения введённого пользователем к числу можно добавить модификатор `number`:

```html
<input v-model.number="age" type="text" />
```

Это часто бывает полезно при использовании полей с типом `text`. При указании типа поля `number` — Vue автоматически станет конвертировать строковое значение в число, добавлять модификатор `.number` к `v-model` не потребуется. Если значение не получится привести к числу с помощью `parseFloat()`, то будет возвращено исходное значение.

### `.trim`

Если необходимо, чтобы пробельные символы в начале и в конце строки автоматически обрезались, то можно добавить модификатор `trim` для полей ввода, управляемых через `v-model`:

```html
<input v-model.trim="msg">
```

## Использование `v-model` с компонентами

> Если ещё не знакомы с компонентами Vue, пока просто пропустите эту секцию

Встроенные в HTML элементы ввода не всегда соответствуют потребностям. К счастью, компоненты Vue позволяют создавать собственные аналоги с полностью настраиваемым поведением. Эти элементы тоже могут работать с директивой `v-model`!

Подробнее в разделе [пользовательские элементы ввода](component-basics.md#использование-v-model-на-компонентах).
