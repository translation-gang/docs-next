# Работа с формами

## Типичное использование

Можно использовать директиву `v-model` для двунаправленного связывания данных с элементами форм input, textarea и select. Способ обновления элемента выбирается автоматически в зависимости от типа элемента. Хотя `v-model` и выглядит как нечто волшебное, в действительности это лишь синтаксический сахар для обновления данных в элементах ввода, с некоторыми поправками для исключительных случаев.

:::tip Совет
`v-model` игнорирует начальное значение атрибутов `value`, `checked` или `selected` на любых элементах форм. Данные текущего активного экземпляра всегда считаются источником истины. Начальное значение необходимо объявить на стороне JavaScript, внутри опции `data` компонента.
:::

Внутренне `v-model` использует разные свойства и генерирует разные события для различных элементов ввода:

- элементы для ввода текста и многострочного текста используют свойство `value` и событие `input`;
- чекбоксы и радиокнопки используют свойство `checked` и событие `change`;
- выпадающие списки используют свойство `value` и событие `change`.

<span id="vmodel-ime-tip"></span>
:::tip Примечание
В языках, требующих [IME](https://ru.wikipedia.org/wiki/IME) (китайский, японский, корейский и т.д.), можно заметить, что `v-model` не обновляется по мере IME-композиции. Если вы хотите обрабатывать и эти обновления, используйте события `input`.
:::

### Текст

```html
<input v-model="message" placeholder="отредактируй меня" />
<p>Введённое сообщение: {{ message }}</p>
```

<common-codepen-snippet title="Handling forms: basic v-model" slug="eYNPEqj" :preview="false" />

### Многострочный текст

```html
<span>Введённое многострочное сообщение:</span>
<p style="white-space: pre-line;">{{ message }}</p>
<br />
<textarea v-model="message" placeholder="введите несколько строчек"></textarea>
```

<common-codepen-snippet title="Handling forms: textarea" slug="xxGyXaG" :preview="false" />

Интерполяция внутри textarea не будет работать. Используйте директиву `v-model` вместо неё.

```html
<!-- плохо -->
<textarea>{{ text }}</textarea>

<!-- хорошо -->
<textarea v-model="text"></textarea>
```

### Чекбоксы

Один чекбокс, привязанный к булевому значению:

```html
<input type="checkbox" id="checkbox" v-model="checked" />
<label for="checkbox">{{ checked }}</label>
```

<common-codepen-snippet title="Handling forms: checkbox" slug="PoqyJVE" :preview="false" />

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

<common-codepen-snippet title="Handling forms: multiple checkboxes" slug="bGdmoyj" :preview="false" />

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

<common-codepen-snippet title="Handling forms: radiobutton" slug="MWwPEMM" :preview="false" />

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

<common-codepen-snippet title="Handling forms: select" slug="KKpGydL" :preview="false" />

:::tip Примечание
Если начальное значение выражения `v-model` не соответствует ни одному из вариантов списка, элемент `<select>` будет отображаться в «невыбранном» состоянии. В iOS это приведёт к тому, что пользователь не сможет выбрать первый элемент, потому что iOS не сгенерирует событие `change` в этом случае. Поэтому рекомендуется предоставлять отключённый `disabled`-вариант выбора с пустым значением value, как показано в примере выше.
:::

Выбор нескольких вариантов из списка (с привязкой к массиву):

```html
<select v-model="selected" multiple>
  <option>А</option>
  <option>Б</option>
  <option>В</option>
</select>
<br />
<span>Выбрано: {{ selected }}</span>
```

<common-codepen-snippet title="Handling forms: select bound to array" slug="gOpBXPz" tab="html,result" :preview="false" />

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

<common-codepen-snippet title="Handling forms: select with dynamic options" slug="abORVZm" :preview="false" />

## Связывание значений

Для радиокнопок и выпадающих списков в качестве `v-model` обычно используются статические строки (или булево для чекбокса):

```html
<!-- `picked` получает строковое значение "a" при клике -->
<input type="radio" v-model="picked" value="a" />

<!-- `toggle` может иметь значение true или false -->
<input type="checkbox" v-model="toggle" />

<!-- `selected` при выборе первого пункта становится равным строке "abc" -->
<select v-model="selected">
  <option value="abc">ABC</option>
</select>
```

Иногда необходимо связать значение с динамическим свойством текущего активного экземпляра. Для этого можно использовать `v-bind`. Кроме того, использование `v-bind` позволяет связывать поле ввода с нестроковыми значениями.

### Чекбокс

```html
<input type="checkbox" v-model="toggle" true-value="да" false-value="нет" />
```

```js
// если чекбокс выбран:
vm.toggle === 'да'
// если чекбокс сброшен:
vm.toggle === 'нет'
```

:::tip Совет
Атрибуты `true-value` и `false-value` не влияют на атрибут `value` тега input, потому что браузеры пропускают невыбранные чекбоксы при отправке форм. Чтобы гарантированно отправлять одно из двух значений с формой (например, «да» или «нет») используйте радиокнопки.
:::

### Радиокнопки

```html
<input type="radio" v-model="pick" :value="a" />
```

```js
// если отмечено:
vm.pick === vm.a
```

### Списки выбора

```html
<select v-model="selected">
  <!-- инлайновый объект с данными -->
  <option :value="{ number: 123 }">123</option>
</select>
```

```js
// когда выбрано:
typeof vm.selected // => 'object'
vm.selected.number // => 123
```

## Модификаторы

### `.lazy`

По умолчанию `v-model` синхронизирует ввод с данными по событию `input` (за исключением [вышеупомянутых](#vmodel-ime-tip) событий IME). Можно указать модификатор `lazy`, чтобы использовать для синхронизации после события `change`:

```html
<!-- синхронизируется по событию "change", а не "input" -->
<input v-model.lazy="msg" />
```

### `.number`

Для автоматического приведения введённого пользователем к Number, добавьте модификатор `number`:

```html
<input v-model.number="age" type="number" />
```

Зачастую это полезно, потому что даже при указанном атрибуте `type="number"` значением поля ввода всегда будет строка. Если значение не удаётся распарсить с помощью `parseFloat()`, то возвращается оригинальное значение.

### `.trim`

Если необходимо, чтобы автоматически обрезались пробелы в начале и в конце строки, используйте модификатор `trim` для полей ввода, обрабатываемых через `v-model`:

```html
<input v-model.trim="msg">
```

## Использование `v-model` с компонентами

> Если вы ещё не знакомы с компонентами Vue, пока просто пропустите эту секцию

Встроенных в HTML элементов ввода не всегда достаточно. К счастью, компоненты Vue позволяют создавать собственные аналоги с полностью настраиваемым поведением. Эти элементы тоже могут работать с директивой `v-model`!

Подробнее в разделе [пользовательские элементы ввода](component-basics.md#using-v-model-on-components).
