# Семантика

## Формы

При создании формы используйте следующие элементы: `<form>`, `<label>`, `<input>`, `<textarea>` и `<button>`.

Как правило, метки размещаются сверху или слева от полей формы:

```html
<form action="/dataCollectionLocation" method="post" autocomplete="on">
  <div v-for="item in formItems" :key="item.id" class="form-item">
    <label :for="item.id">{{ item.label }}: </label>
    <input
      :type="item.type"
      :id="item.id"
      :name="item.id"
      v-model="item.value"
    />
  </div>
  <button type="submit">Submit</button>
</form>
```

<common-codepen-snippet title="Простая форма" slug="YzwpPYZ" :height="368" tab="js,result" :team="false" user="mlama007" name="Maria" theme="light" :preview="false" :editable="false" />

Обратите внимание, если добавить `autocomplete='on'` к самому элементу формы, то он этот атрибут применится ко всем полям формы. Для каждого поля можно задать разные [значения в атрибуте autocomplete](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/autocomplete).

### Метки

В метках даётся описание полей формы; они устанавливают связь между элементами с атрибутами `for` и `id`:

```html
<label for="name">Name</label>
<input type="text" name="name" id="name" v-model="name" />
```

<common-codepen-snippet title="Метка формы" slug="wvMrGqz" :height="265" tab="js,result" :team="false" user="mlama007" name="Maria" theme="light" :preview="false" :editable="false" />

Если открыть инструменты разработки (например, DevTools в браузере Chrome) на вкладке Elements, затем перейти на вкладку Accessibility, можно заметить, что имя поле извлекается из её метки:

![В инструментах разработки Chrome показывается имя поле, указанное в метке](/images/AccessibleLabelChromeDevTools.png)

:::warning ВНИМАНИЕ:
Возможно, вы видели, как поля ввода находится внутри элемента метки: 

```html
<label>
  Name:
  <input type="text" name="name" id="name" v-model="name" />
</label>
```

Однако так лучше не делать, поскольку вспомогательные технологии лучше работают с метками, явно связанные с идентификаторами.
:::

#### aria-label

С помощью атрибута [`aria-label`](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_aria-label_attribute) можно задать описание поля для вспомогательных технологий.

```html
<label for="name">Name</label>
<input
  type="text"
  name="name"
  id="name"
  v-model="name"
  :aria-label="nameLabel"
/>
```

<common-codepen-snippet title="Form ARIA label" slug="jOWGqgz" :height="265" tab="js,result" :team="false" user="mlama007" name="Maria" theme="light" :preview="false" :editable="false" />

В Chrome DevTools посмотрите сами, что метка формы изменилась:

![В инструментах разработки Chrome показывается имя поля, заданное в aria-label](/images/AccessibleARIAlabelDevTools.png)

#### aria-labelledby

Использование атрибута [`aria-labelledby`](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_aria-labelledby_attribute) похоже на `aria-label`, за исключением того, что текст метки отображается на экране. Он создаёт связь между элементами с атрибутом `id`, допускается указать несколько `id`:

```html
<form
  class="demo"
  action="/dataCollectionLocation"
  method="post"
  autocomplete="on"
>
  <h1 id="billing">Billing</h1>
  <div class="form-item">
    <label for="name">Name:</label>
    <input
      type="text"
      name="name"
      id="name"
      v-model="name"
      aria-labelledby="billing name"
    />
  </div>
  <button type="submit">Submit</button>
</form>
```

<common-codepen-snippet title="Form ARIA labelledby" slug="ZEQXOLP" :height="265" tab="js,result" :team="false" user="mlama007" name="Maria" theme="light" :preview="false" :editable="false" />

![В инструментах разработки Chrome показывается имя поля, указанное в атрибуте aria-labelledby](/images/AccessibleARIAlabelledbyDevTools.png)

#### aria-describedby

Атрибут [aria-describedby](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_aria-describedby_attribute) аналогичен `aria-labelledby`, но представляет собой дополнительную информацию для пользователя. Его можно использовать для добавления условий ввода для поля:

```html
<form
  class="demo"
  action="/dataCollectionLocation"
  method="post"
  autocomplete="on"
>
  <h1 id="billing">Billing</h1>
  <div class="form-item">
    <label for="name">Full Name:</label>
    <input
      type="text"
      name="name"
      id="name"
      v-model="name"
      aria-labelledby="billing name"
      aria-describedby="nameDescription"
    />
    <p id="nameDescription">Please provide first and last name.</p>
  </div>
  <button type="submit">Submit</button>
</form>
```

<common-codepen-snippet title="Form ARIA describedby" slug="JjGrKyY" :height="265" tab="js,result" :team="false" user="mlama007" name="Maria" theme="light" :preview="false" :editable="false" />

Через Chrome DevTools можно увидеть применённое описание:

![В инструментах разработки Chrome показывается имя из атрибута aria-labelledby вместе с описанием, полученным из атрибута aria-describedby](/images/AccessibleARIAdescribedby.png)

### Подсказка поля

Постарайтесь не использовать подсказки полей, поскольку они могут запутать многих пользователей.

По умолчанию подсказки полей не соответствуют [критериям цветового контраста](https://www.w3.org/WAI/WCAG21/Understanding/contrast-minimum.html). Попытка исправить эту проблему приведёт к тому, что подсказка поля будет выглядеть как предварительно заполненные данные в элементах формы. Посмотрите на следующий пример: подсказка поля Last Name соответствует критериям цветового контраста, хотя она не отличается от введённого значения:

<common-codepen-snippet title="Form Placeholder" slug="PoZJzeQ" :height="265" tab="js,result" :team="false" user="mlama007" name="Maria" theme="light" :preview="false" :editable="false" />

Таким образом, лучше всего вынести всю справочную информацию за пределы полей формы.

### Инструкции

При добавлении инструкций для заполнения к полям формы убедитесь, что они правильно связаны между собой.
Все инструкции можно указать в виде идентификаторов внутри атрибута [`aria-labelledby`](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_aria-labelledby_attribute). Это делает дизайн более гибким.

```html
<fieldset>
  <legend>Using aria-labelledby</legend>
  <label id="date-label" for="date">Current Date:</label>
  <input
    type="date"
    name="date"
    id="date"
    aria-labelledby="date-label date-instructions"
  />
  <p id="date-instructions">MM/DD/YYYY</p>
</fieldset>
```

В качестве альтернативного варианта можно воспользоваться атрибутом [`aria-describedby`](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_aria-describedby_attribute):

```html
<fieldset>
  <legend>Using aria-describedby</legend>
  <label id="dob" for="dob">Date of Birth:</label>
  <input type="date" name="dob" id="dob" aria-describedby="dob-instructions" />
  <p id="dob-instructions">MM/DD/YYYY</p>
</fieldset>
```

<common-codepen-snippet title="Form Instructions" slug="GRoMqYy" :height="265" tab="js,result" :team="false" user="mlama007" name="Maria" theme="light" :preview="false" :editable="false" />

### Скрытие содержимого

Обычно не рекомендуется скрывать метки, даже если для поля ввода задано имя. Тем не менее, если смысл поля понятен из контекста, то метку можно скрыть.

Рассмотрим следующее поле поиска:

```html
<form role="search">
  <label for="search" class="hidden-visually">Search: </label>
  <input type="text" name="search" id="search" v-model="search" />
  <button type="submit">Search</button>
</form>
```

В данном случае можно скрыть метку, поскольку кнопка поиска даёт понять, для чего предназначено поле ввода.

При помощи CSS-класса скроем необходимый элемент, при этом сохранив его функциональность для вспомогательных технологий:

```css
.hidden-visually {
  position: absolute;
  overflow: hidden;
  white-space: nowrap;
  margin: 0;
  padding: 0;
  height: 1px;
  width: 1px;
  clip: rect(0 0 0 0);
  clip-path: inset(100%);
}
```

<common-codepen-snippet title="Form Search" slug="qBbpQwB" :height="265" tab="js,result" :team="false" user="mlama007" name="Maria" theme="light" :preview="false" :editable="false" />

#### aria-hidden="true"

Добавление `aria-hidden="true"` скроет элемент от вспомогательных технологий, однако он по-прежнему будет показываться другим пользователям. Не используйте его на фокусируемых элементах, только для декоративного, дублируемого или не предназначенного для вывода на экран содержимого.

```html
<p>This is not hidden from screen readers.</p>
<p aria-hidden="true">This is hidden from screen readers.</p>
```

### Кнопки

При использовании кнопок внутри формы, следует задать тип для них, чтобы избежать отправку формы.
Для создания кнопок также можно использовать обычное поле ввода:

```html
<form action="/dataCollectionLocation" method="post" autocomplete="on">
  <!-- Кнопки -->
  <button type="button">Cancel</button>
  <button type="submit">Submit</button>

  <!-- Кнопки как поля ввода -->
  <input type="button" value="Cancel" />
  <input type="submit" value="Submit" />
</form>
```

<common-codepen-snippet title="Form Buttons" slug="PoZEXoj" :height="467" tab="js,result" :team="false" user="mlama007" name="Maria" theme="light" :preview="false" :editable="false" />

#### Функциональные изображения

Для создания функциональных изображений можно использовать перечисленную ниже технику.

- Поля ввода

  - Изображение будет работать как кнопка отправки формы

  ```html
  <form role="search">
    <label for="search" class="hidden-visually">Search: </label>
    <input type="text" name="search" id="search" v-model="search" />
    <input
      type="image"
      class="btnImg"
      src="https://img.icons8.com/search"
      alt="Search"
    />
  </form>
  ```

- Иконки

```html
<form role="search">
  <label for="searchIcon" class="hidden-visually">Search: </label>
  <input type="text" name="searchIcon" id="searchIcon" v-model="searchIcon" />
  <button type="submit">
    <i class="fas fa-search" aria-hidden="true"></i>
    <span class="hidden-visually">Search</span>
  </button>
</form>
```

<common-codepen-snippet title="Функциональные изображения" slug="NWxXeqY" :height="265" tab="js,result" :team="false" user="mlama007" name="Maria" theme="light" :preview="false" :editable="false" />
