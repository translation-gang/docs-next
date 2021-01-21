# Семантика

## Формы

При создании формы можно использовать следующие элементы: `<form>`, `<label>`, `<input>`, `<textarea>` и `<button>`.

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

Обратите внимание, можно добавить `autocomplete='on'` к самому элементу формы, и этот атрибут применится ко всем полям формы. Каждому полю можно задать свои [значения атрибута autocomplete](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/autocomplete).

### Метки

Добавляйте метки для описания полей формы, а также создания связи между элементами с атрибутами `for` и `id`:

```html
<label for="name">Name</label>
<input type="text" name="name" id="name" v-model="name" />
```

<common-codepen-snippet title="Метка формы" slug="wvMrGqz" :height="265" tab="js,result" :team="false" user="mlama007" name="Maria" theme="light" :preview="false" :editable="false" />

Если посмотреть элемент через инструменты разработки (например, DevTools в браузере Chrome) и перейти на вкладку Accessibility внутри раздела Elements, то можно увидеть, что имя поле извлекается из связанной с ним метки:

![В инструментах разработки Chrome показывается имя поле, указанное в метке](/images/AccessibleLabelChromeDevTools.png)

:::warning ВНИМАНИЕ:
Часто можно встретить случаи, когда поле ввода находится внутри элемента с меткой:

```html
<label>
  Name:
  <input type="text" name="name" id="name" v-model="name" />
</label>
```

Однако явное указание меткам соответствующего идентификатора лучше поддерживается вспомогательными технологиями.
:::

#### aria-label

Указать имя поля для использования вспомогательными технологиями можно с помощью атрибута [`aria-label`](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_aria-label_attribute).

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

Можно убедиться самостоятельно что имя элемента изменилось с помощью инструментов разработки Chrome DevTools:

![В инструментах разработки Chrome показывается имя поля, заданное в aria-label](/images/AccessibleARIAlabelDevTools.png)

#### aria-labelledby

Использование атрибута [`aria-labelledby`](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_aria-labelledby_attribute) похоже на `aria-label`, за исключением того, что текст метки показывается на экране. Он создаёт связь между элементами с атрибутом `id`, допускается указать несколько `id`:

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

Атрибут [aria-describedby](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_aria-describedby_attribute) используется аналогично `aria-labelledby`, но предоставляет дополнительную информацию, которая может потребоваться пользователю. Его можно использовать для описания ограничений ввода:

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

Старайтесь ограничить использования подсказок полей, так как они могут запутать пользователя.

Одна из проблем в том, что по умолчанию подсказки полей не соответствуют [критериям цветового контраста](https://www.w3.org/WAI/WCAG21/Understanding/contrast-minimum.html). Попытка исправить цветовой контраст может сделать подсказку похожей на уже заполненное поле. Посмотрите на следующий пример: подсказка поля Last Name соответствует критериям цветового контраста, хотя она не отличается от введённого значения:

<common-codepen-snippet title="Form Placeholder" slug="PoZJzeQ" :height="265" tab="js,result" :team="false" user="mlama007" name="Maria" theme="light" :preview="false" :editable="false" />

Поэтому будет лучше всего перенести всю необходимую информацию для заполнения за пределы полей формы.

### Инструкции

При добавлении инструкций для заполнения к полям формы убедитесь, что они правильно связаны между собой.
Можно указать дополнительные инструкции в виде идентификаторов в атрибуте [`aria-labelledby`](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_aria-labelledby_attribute). Это делает дизайн более гибким.

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

В качестве альтернативы можно привязать инструкции к полю с помощью атрибута [`aria-describedby`](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_aria-describedby_attribute):

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

Обычно не рекомендуется визуально скрывать метки, даже если для поля ввода задано имя. Тем не менее, если смысл поля понятен из контекста, то метку можно скрыть.

Рассмотрим следующее поле поиска:

```html
<form role="search">
  <label for="search" class="hidden-visually">Search: </label>
  <input type="text" name="search" id="search" v-model="search" />
  <button type="submit">Search</button>
</form>
```

В данном случае можно скрыть метку, поскольку кнопка поиска поможет пользователю определить назначение поля.

С помощью CSS-класса визуально скрываем элемент, но оставляем их доступными для вспомогательных технологий:

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

Добавление `aria-hidden="true"` скроет элемент от обнаружения вспомогательными технологиями, но оставит его визуально доступным для остальных пользователей. Не используйте его на фокусируемых элементах, а добавляйте только для декоративных, дублирующихся или не отображаемых на экране элементов.

```html
<p>This is not hidden from screen readers.</p>
<p aria-hidden="true">This is hidden from screen readers.</p>
```

### Кнопки

При использовании кнопок внутри формы, следует указывать их тип, чтобы избежать отправку формы.
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
