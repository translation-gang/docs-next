# Семантика

## Формы

При создании форм можно использовать следующие элементы: `<form>`, `<label>`, `<input>`, `<textarea>` и `<button>`.

Как правило, метки к полям формы размещаются сверху или слева:

```html{3}
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
  <button type="submit">Отправить</button>
</form>
```

<common-codepen-snippet title="Простая форма" slug="dyNzzWZ" :height="368" tab="js,result" theme="light" :preview="false" :editable="false" />

Обратите внимание, атрибут `autocomplete="on"` можно добавить к элементу формы, и он применится ко всем полям формы. Каждому полю можно задать и своё [значение атрибута autocomplete](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/autocomplete).

### Метки

Добавляйте метки для описания полей формы и для создания связи между элементами с атрибутами `for` и `id`:

```html
<label for="name">Имя</label>
<input type="text" name="name" id="name" v-model="name" />
```

<common-codepen-snippet title="Метки в форме" slug="XWpaaaj" :height="265" tab="js,result" theme="light" :preview="false" :editable="false" />

Если изучить элемент через инструменты разработчика в браузере и перейти на вкладку Accessibility в разделе Elements, то можно увидеть, что поле получает имя из его метки:

![В инструментах разработки Chrome показывается имя поле, указанное в метке](/images/AccessibleLabelChromeDevTools.png)

:::warning ВНИМАНИЕ:
Часто встречаются случаи, когда поле ввода находится внутри элемента с меткой:

```html
<label>
  Имя:
  <input type="text" name="name" id="name" v-model="name" />
</label>
```

Явное указание метки с соответствующим идентификатором лучше поддерживается вспомогательными технологиями.
:::

#### aria-label

Указать имя поля для использования вспомогательными технологиями можно с помощью атрибута [`aria-label`](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_aria-label_attribute).

```html{7}
<label for="name">Имя</label>
<input
  type="text"
  name="name"
  id="name"
  v-model="name"
  :aria-label="nameLabel"
/>
```

<common-codepen-snippet title="Определение ARIA меток в форме" slug="NWdvvYQ" :height="265" tab="js,result" theme="light" :preview="false" :editable="false" />

Можно самостоятельно убедиться что имя элемента изменилось с помощью инструментов разработчика браузера:

![В инструментах разработки Chrome показывается имя поля, заданное в aria-label](/images/AccessibleARIAlabelDevTools.png)

#### aria-labelledby

Использование атрибута [`aria-labelledby`](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_aria-labelledby_attribute) похоже на `aria-label`, за исключением того, что текст метки показывается на экране. Он создаёт связь между элементами с атрибутом `id`, при этом допускается указать несколько `id`:

```html{15}
<form
  class="demo"
  action="/dataCollectionLocation"
  method="post"
  autocomplete="on"
>
  <h1 id="billing">Оплата</h1>
  <div class="form-item">
    <label for="name">Имя:</label>
    <input
      type="text"
      name="name"
      id="name"
      v-model="name"
      aria-labelledby="billing name"
    />
  </div>
  <button type="submit">Отправить</button>
</form>
```

<common-codepen-snippet title="Определение меток ARIA labelledby в форме" slug="MWJvvBe" :height="265" tab="js,result" theme="light" :preview="false" :editable="false" />

![В инструментах разработки Chrome показывается имя поля, указанное в атрибуте aria-labelledby](/images/AccessibleARIAlabelledbyDevTools.png)

#### aria-describedby

Атрибут [aria-describedby](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_aria-describedby_attribute) используется аналогично `aria-labelledby`, но предоставляет дополнительную информацию, которая может потребоваться пользователю. Его можно использовать для описания ограничений ввода:

```html{16}
<form
  class="demo"
  action="/dataCollectionLocation"
  method="post"
  autocomplete="on"
>
  <h1 id="billing">Оплата</h1>
  <div class="form-item">
    <label for="name">Имя и фамилия:</label>
    <input
      type="text"
      name="name"
      id="name"
      v-model="name"
      aria-labelledby="billing name"
      aria-describedby="nameDescription"
    />
    <p id="nameDescription">Пожалуйста укажите имя и фамилию.</p>
  </div>
  <button type="submit">Отправить</button>
</form>
```

<common-codepen-snippet title="Определение меток ARIA describedby в форме" slug="gOgxxQE" :height="265" tab="js,result" theme="light" :preview="false" :editable="false" />

В инструментах разработчика браузера можно увидеть изменённое описание:

![В инструментах разработки Chrome показывается имя из атрибута aria-labelledby вместе с описанием, полученным из атрибута aria-describedby](/images/AccessibleARIAdescribedby.png)

### Подсказка внутри поля

Старайтесь ограничить использование подсказок внутри поля, так как они могут запутать многих пользователей.

Одна из проблем подсказок внутри поля в том, что по умолчанию они не соответствуют [критериям цветового контраста](https://www.w3.org/WAI/WCAG21/Understanding/contrast-minimum.html). Попытка исправить цветовой контраст может сделать подсказку похожей на уже заполненное поле. Посмотрите на следующий пример: подсказка поля Last Name соответствует критериям цветового контраста, хотя она не отличима от поля с подставленным значением:

<common-codepen-snippet title="Подсказки полей в формах" slug="ExZvvMw" :height="265" tab="js,result" theme="light" :preview="false" :editable="false" />

Всю необходимую информацию для заполнения лучше всего указывать вне полей формы.

### Инструкции

При добавлении инструкций для заполнения к полям формы убедитесь, что они правильно связаны между собой.
Можно указать дополнительные инструкции в виде идентификаторов в атрибуте [`aria-labelledby`](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_aria-labelledby_attribute). Это делает дизайн более гибким.

```html{3,8,10}
<fieldset>
  <legend>Использование aria-labelledby</legend>
  <label id="date-label" for="date">Текущая дата:</label>
  <input
    type="date"
    name="date"
    id="date"
    aria-labelledby="date-label date-instructions"
  />
  <p id="date-instructions">MM/DD/YYYY</p>
</fieldset>
```

Также можно привязать инструкции к полю с помощью атрибута [`aria-describedby`](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_aria-describedby_attribute):

```html{4,5}
<fieldset>
  <legend>Использование aria-describedby</legend>
  <label id="dob" for="dob">Дата рождения:</label>
  <input type="date" name="dob" id="dob" aria-describedby="dob-instructions" />
  <p id="dob-instructions">MM/DD/YYYY</p>
</fieldset>
```

<common-codepen-snippet title="Инструкции в формах" slug="WNREEqv" :height="265" tab="js,result" theme="light" :preview="false" :editable="false" />

### Скрытие содержимого

Обычно не рекомендуется визуально скрывать метки, даже если для поля ввода задано имя. Тем не менее, если смысл поля понятен из контекста, то метку можно скрыть.

Рассмотрим поле поиска:

```html
<form role="search">
  <label for="search" class="hidden-visually">Поиск: </label>
  <input type="text" name="search" id="search" v-model="search" />
  <button type="submit">Поиск</button>
</form>
```

В данном случае можно скрыть метку, потому что кнопка поиска визуально поможет определить назначение поля.

Можно использовать CSS, чтобы визуально скрывать элементы, но оставить их доступными для вспомогательных технологий:

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

<common-codepen-snippet title="Поиск в формах" slug="QWdMqWy" :height="265" tab="js,result" theme="light" :preview="false" :editable="false" />

#### aria-hidden="true"

Добавление `aria-hidden="true"` скрывает элемент от обнаружения вспомогательными технологиями, но оставит его визуально доступным для остальных пользователей. Не стоит использовать его на фокусируемых элементах, а применять только для декоративных, дублирующихся или не отображаемых на экране элементов.

```html
<p>Это ВИДИМО для устройств чтения с экрана.</p>
<p aria-hidden="true">Это СКРЫТО от устройств чтения с экрана.</p>
```

### Кнопки

При использовании кнопок внутри формы, следует указывать их тип, чтобы предотвратить отправку формы.
Для создания кнопок можно также использовать и обычное поле ввода:

```html
<form action="/dataCollectionLocation" method="post" autocomplete="on">
  <!-- Кнопки -->
  <button type="button">Отменить</button>
  <button type="submit">Сохранить</button>

  <!-- Поля ввода в виде кнопок -->
  <input type="button" value="Отменить" />
  <input type="submit" value="Сохранить" />
</form>
```

<common-codepen-snippet title="Кнопки в формах" slug="JjEyrYZ" :height="467" tab="js,result" theme="light" :preview="false" :editable="false" />

#### Функциональные изображения

Для создания функциональных изображений можно использовать указанную ниже технику.

- Поля ввода

  - Изображение будет работать как кнопка отправки формы

  ```html{4-9}
  <form role="search">
    <label for="search" class="hidden-visually">Поиск: </label>
    <input type="text" name="search" id="search" v-model="search" />
    <input
      type="image"
      class="btnImg"
      src="https://img.icons8.com/search"
      alt="Поиск"
    />
  </form>
  ```

- Иконки

```html{5}
<form role="search">
  <label for="searchIcon" class="hidden-visually">Поиск: </label>
  <input type="text" name="searchIcon" id="searchIcon" v-model="searchIcon" />
  <button type="submit">
    <i class="fas fa-search" aria-hidden="true"></i>
    <span class="hidden-visually">Поиск</span>
  </button>
</form>
```

<common-codepen-snippet title="Функциональные изображения" slug="jOyLGqM" :height="265" tab="js,result" theme="light" :preview="false" :editable="false" />
