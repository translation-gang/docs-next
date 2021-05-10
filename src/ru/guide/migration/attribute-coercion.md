---
badges:
  - breaking
---

# Изменено поведение при приведении значения атрибутов <MigrationBadges :badges="$frontmatter.badges" />

:::info Информация
Это изменение низкоуровневого внутреннего API не затрагивает большинство разработчиков.
:::

## Обзор

Краткий обзор изменений:

- Отказ от внутреннего концепта перечисляемых атрибутов и обработка таких атрибутов также, как и обычных не-булевых атрибутов.
- **КАРДИНАЛЬНОЕ ИЗМЕНЕНИЕ:** Больше не удаляется атрибут, если его значение `false`. Вместо этого будет устанавливаться `attr="false"`. Для удаления атрибута необходимо использовать `null` или `undefined`.

Для получения дополнительной информации, читайте дальше!

## Синтаксис в 2.x

В 2.x было несколько стратегий для приведения значений `v-bind`:

- Для некоторых пар атрибут/элемент Vue всегда использовал соответствующий IDL атрибут (свойство): [например `value` для `<input>`, `<select>`, `<progress>`, и т.д.](https://github.com/vuejs/vue/blob/bad3c326a3f8b8e0d3bcf07917dc0adf97c32351/src/platforms/web/util/attrs.js#L11-L18).

- Для «[булевых атрибутов](https://github.com/vuejs/vue/blob/bad3c326a3f8b8e0d3bcf07917dc0adf97c32351/src/platforms/web/util/attrs.js#L33-L40)» и [xlinks](https://github.com/vuejs/vue/blob/bad3c326a3f8b8e0d3bcf07917dc0adf97c32351/src/platforms/web/util/attrs.js#L44-L46) Vue удалял их, если они «ложны» ([`undefined`, `null` или `false`](https://github.com/vuejs/vue/blob/bad3c326a3f8b8e0d3bcf07917dc0adf97c32351/src/platforms/web/util/attrs.js#L52-L54)) и добавлял их в обратном случае (см. [здесь](https://github.com/vuejs/vue/blob/bad3c326a3f8b8e0d3bcf07917dc0adf97c32351/src/platforms/web/runtime/modules/attrs.js#L66-L77) и [здесь](https://github.com/vuejs/vue/blob/bad3c326a3f8b8e0d3bcf07917dc0adf97c32351/src/platforms/web/runtime/modules/attrs.js#L81-L85)).

- Для «[перечисляемых атрибутов](https://github.com/vuejs/vue/blob/bad3c326a3f8b8e0d3bcf07917dc0adf97c32351/src/platforms/web/util/attrs.js#L20)» (в настоящий момент `contenteditable`, `draggable` и `spellcheck`), Vue пытался [привести](https://github.com/vuejs/vue/blob/bad3c326a3f8b8e0d3bcf07917dc0adf97c32351/src/platforms/web/util/attrs.js#L24-L31) их к строке (со специальной обработкой для `contenteditable`, чтобы исправить [vuejs/vue#9397](https://github.com/vuejs/vue/issues/9397)).

- Для других атрибутов, удалялись при «ложных» значениях (`undefined`, `null` или `false`) и устанавливали другие значения как есть (см. [здесь](https://github.com/vuejs/vue/blob/bad3c326a3f8b8e0d3bcf07917dc0adf97c32351/src/platforms/web/runtime/modules/attrs.js#L92-L113)).

В следующей таблице описывается каким образом Vue по-разному приводил «перечисляемые атрибуты» в сравнении с обычными не-булевыми атрибутами:

| Выражение           | `foo` <sup>обычный</sup> | `draggable` <sup>перечисляемый</sup> |
| ------------------- | ----------------------- | ------------------------------------- |
| `:attr="null"`      | /                       | `draggable="false"`                   |
| `:attr="undefined"` | /                       | /                                     |
| `:attr="true"`      | `foo="true"`            | `draggable="true"`                    |
| `:attr="false"`     | /                       | `draggable="false"`                   |
| `:attr="0"`         | `foo="0"`               | `draggable="true"`                    |
| `attr=""`           | `foo=""`                | `draggable="true"`                    |
| `attr="foo"`        | `foo="foo"`             | `draggable="true"`                    |
| `attr`              | `foo=""`                | `draggable="true"`                    |

Как можно увидеть из таблицы, текущая реализация приводит `true` к `'true'`, но удаляет атрибут при значении `false`. Это также приводит к несогласованности и требовало от пользователей вручную приводить булевые значения к строке в распространённых случаях использования, например `aria-*` атрибутов, таких как `aria-selected`, `aria-hidden`, и т.д.

## Синтаксис в 3.x

От внутреннего концепта «перечисляемых атрибутов» было решено отказаться и обрабатывать их как обычные не-булевы HTML-атрибуты.

- Это решает проблемы несоответствия между обычными не-булевыми атрибутами и «перечисляемыми атрибутами»
- Это также позволяет использовать значения, отличные от `'true'` и `'false'`, или даже ключевых слов, которые могут быть в будущем, для таких атрибутов, как `contenteditable`

Для не-булевых атрибутов Vue перестанет удалять их, если значение `false` и вместо этого станет приводить их к `'false'`.

- Это решает проблему несоответствия между `true` и `false` и облегчает работу с атрибутами `aria-*`

Следующая таблица описывает новое поведение:

| Выражение           | `foo` <sup>обычный</sup>    | `draggable` <sup>перечисляемый</sup> |
| ------------------- | -------------------------- | ------------------------------------- |
| `:attr="null"`      | /                          | / <sup>*</sup>                        |
| `:attr="undefined"` | /                          | /                                     |
| `:attr="true"`      | `foo="true"`               | `draggable="true"`                    |
| `:attr="false"`     | `foo="false"` <sup>*</sup> | `draggable="false"`                   |
| `:attr="0"`         | `foo="0"`                  | `draggable="0"` <sup>*</sup>          |
| `attr=""`           | `foo=""`                   | `draggable=""` <sup>*</sup>           |
| `attr="foo"`        | `foo="foo"`                | `draggable="foo"` <sup>*</sup>        |
| `attr`              | `foo=""`                   | `draggable=""` <sup>*</sup>           |

<small>*: поведение изменилось</small>

Приведение для булевых атрибутов осталось без изменений.

## Стратегия миграции

### Перечисляемые атрибуты

Отсутствие перечисляемого атрибута и `attr="false"` может привести к разным значениям атрибутов IDL (которые будут отражать действительное состояние), описанным ниже:

| Отсутствие перечисляемого атрибута | IDL атрибут & значение               |
| ---------------------------------- | ------------------------------------ |
| `contenteditable`                  | `contentEditable` &rarr; `'inherit'` |
| `draggable`                        | `draggable` &rarr; `false`           |
| `spellcheck`                       | `spellcheck` &rarr; `true`           |

Чтобы сохранить старое поведение, так как будут приводиться значения `false` к `'false'`, во Vue 3.x разработчикам необходимо доработать привязки `v-bind` таким образом, чтобы разрешались значением `false` или `'false'` для `contenteditable` и `spellcheck`.

Во Vue 2.x, для перечисляемых атрибутов недействительные значения приводились к `'true'`. Обычно такое поведение не требовалось и вряд ли на него можно было рассчитывать в больших масштабах. Во Vue 3.x `true` или `'true'` должны быть явно определены.

### Приведение `false` к `'false'` вместо удаления атрибута

Во Vue 3.x, `null` или `undefined` должны явно использоваться для удаления атрибута.

### Сравнение поведения в 2.x и в 3.x

<table>
  <thead>
    <tr>
      <th>Атрибут</th>
      <th><code>v-bind</code> значение <sup>2.x</sup></th>
      <th><code>v-bind</code> значение <sup>3.x</sup></th>
      <th>HTML результат</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="3">2.x «Перечисляемые атрибуты»<br><small>напр., <code>contenteditable</code>, <code>draggable</code> и <code>spellcheck</code>.</small></td>
      <td><code>undefined</code>, <code>false</code></td>
      <td><code>undefined</code>, <code>null</code></td>
      <td><i>удалён</i></td>
    </tr>
    <tr>
      <td>
        <code>true</code>, <code>'true'</code>, <code>''</code>, <code>1</code>,
        <code>'foo'</code>
      </td>
      <td><code>true</code>, <code>'true'</code></td>
      <td><code>"true"</code></td>
    </tr>
    <tr>
      <td><code>null</code>, <code>'false'</code></td>
      <td><code>false</code>, <code>'false'</code></td>
      <td><code>"false"</code></td>
    </tr>
    <tr>
      <td rowspan="2">Остальные не-булевы атрибуты<br><small>напр., <code>aria-checked</code>, <code>tabindex</code>, <code>alt</code>, и т.д.</small></td>
      <td><code>undefined</code>, <code>null</code>, <code>false</code></td>
      <td><code>undefined</code>, <code>null</code></td>
      <td><i>удалён</i></td>
    </tr>
    <tr>
      <td><code>'false'</code></td>
      <td><code>false</code>, <code>'false'</code></td>
      <td><code>"false"</code></td>
    </tr>
  </tbody>
</table>
