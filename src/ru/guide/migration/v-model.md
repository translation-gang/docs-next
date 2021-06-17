---
badges:
  - breaking
---

# Изменения в работе `v-model` <MigrationBadges :badges="$frontmatter.badges" />

## Обзор

Краткий обзор что изменилось:

- **КАРДИНАЛЬНОЕ ИЗМЕНЕНИЕ:** При использовании на компонентах `v-model` поменялись входной параметр и имя события:
  - входной параметр: `value` -> `modelValue`;
  - событие: `input` -> `update:modelValue`;
- **КАРДИНАЛЬНОЕ ИЗМЕНЕНИЕ:** Модификатор `.sync` для `v-bind` и опция компонента `model` были удалены и заменяются возможностью указать аргумент `v-model`;
- **НОВОЕ:** Теперь возможны несколько привязок `v-model` на одном компоненте;
- **НОВОЕ:** Добавлена возможность создавать собственные модификаторы для `v-model`.

Для получения дополнительной информации читайте дальше!

## Введение

Во Vue 2.0 директива `v-model` требовала, чтобы использовался входной параметр `value`. Для управления ещё другим входным параметром приходилось использовать `v-bind.sync`. Кроме того, жёстко прописанная связь между `v-model` и `value` приводила к проблемам при обработке нативных и пользовательских элементов.

В версии 2.2 была добавлена опция компонента `model`, которая позволила настраивать входной параметр и событие, используемые для `v-model`. Но использовать можно было всё равно только одну `v-model` на компоненте.

Во Vue 3 было унифицировано API для двусторонней привязки данных, чтобы уменьшить путаницу и предоставить разработчикам больше гибкости в работе с директивой `v-model`.

## Синтаксис в 2.x

В версии 2.x использование `v-model` на компоненте эквивалентно передаче входного параметра `value` и отслеживании сгенерированного события `input`:

```html
<ChildComponent v-model="pageTitle" />

<!-- будет сокращённой версией для: -->

<ChildComponent :value="pageTitle" @input="pageTitle = $event" />
```

Если хочется изменить используемый входной параметр или имя события на другие, можно воспользоваться опцией `model` в компоненте `ChildComponent`:

```html
<!-- ParentComponent.vue -->

<ChildComponent v-model="pageTitle" />
```

```js
// ChildComponent.vue

export default {
  model: {
    prop: 'title',
    event: 'change'
  },
  props: {
    // это позволит использовать входной параметр `value` в других целях
    value: String,
    // теперь используем входной параметр `title` вместо `value`
    title: {
      type: String,
      default: 'Заголовок по умолчанию'
    }
  }
}
```

В этом случае `v-model` будет сокращённым вариантом для

```html
<ChildComponent :title="pageTitle" @change="pageTitle = $event" />
```

### Использование `v-bind.sync`

В некоторых случаях может потребоваться «двусторонняя привязка» к входному параметру (иногда в дополнение к существующей `v-model` для другого). Для этого рекомендуется генерировать событие по шаблону `update:myPropName`. Например, для `ChildComponent` из предыдущего примера с входным параметром `title` можно передавать намерение о присвоении нового значения следующий образом:

```js
this.$emit('update:title', newValue)
```

Тогда родительский компонент может прослушать это событие и обновить локальное свойство, если захочет. Например:

```html
<ChildComponent :title="pageTitle" @update:title="pageTitle = $event" />
```

Для удобства, сокращённый вариант для этого шаблона с модификатором `.sync` такой:

```html
<ChildComponent :title.sync="pageTitle" />
```

## Синтаксис в 3.x

В версии 3.x `v-model` на пользовательском компоненте эквивалентно передаче входного параметра `modelValue` и отслеживании сгенерированного события `update:modelValue`:

```html
<ChildComponent v-model="pageTitle" />

<!-- будет сокращённой версия для: -->

<ChildComponent
  :modelValue="pageTitle"
  @update:modelValue="pageTitle = $event"
/>
```

### Аргументы `v-model`

Чтобы изменить имя свойства, вместо использования опции `model` в компоненте, теперь можно передавать _аргумент_ в директиву `v-model`:

```html
<ChildComponent v-model:title="pageTitle" />

<!-- будет сокращённой версией для: -->

<ChildComponent :title="pageTitle" @update:title="pageTitle = $event" />
```

![Анатомия v-bind](/images/v-bind-instead-of-sync.png)

Это также служит заменой модификатору `.sync` и позволяет указать несколько `v-model` на пользовательском компоненте.

```html
<ChildComponent
  v-model:title="pageTitle"
  v-model:content="pageContent"
/>

<!-- будет сокращённой версией для: -->

<ChildComponent
  :title="pageTitle"
  @update:title="pageTitle = $event"
  :content="pageContent"
  @update:content="pageContent = $event"
/>
```

### Модификаторы `v-model`

Кроме жёстко заданных модификаторов `v-model` в версии 2.x, таких как `.trim`, в версии 3.x теперь поддерживается создание пользовательских модификаторов:

```html
<ChildComponent v-model.capitalize="pageTitle" />
```

Подробнее о пользовательских модификаторах `v-model` можно прочитать в разделе [пользовательских событий](../component-custom-events.md#обработка-модификаторов-v-model).

## Стратегия миграции

Рекомендуется:

- проверить кодовую базу на использование `.sync` и заменить их на `v-model`:

  ```html
  <ChildComponent :title.sync="pageTitle" />

  <!-- следует заменить на -->

  <ChildComponent v-model:title="pageTitle" />
  ```

- для всех `v-model` без аргументов убедитесь, что изменили входной параметр и имя события на `modelValue` и `update:modelValue` соответственно

  ```html
  <ChildComponent v-model="pageTitle" />
  ```

  ```js{5,7,10}
  // ChildComponent.vue

  export default {
    props: {
      modelValue: String // раньше было `value: String`
    },
    emits: ['update:modelValue'],
    methods: {
      changePageTitle(title) {
        this.$emit('update:modelValue', title) // раньше было `this.$emit('input', title)`
      }
    }
  }
  ```

[Флаги сборки для миграции:](migration-build.md#конфигурация-совместимости)

- `COMPONENT_V_MODEL`
- `COMPILER_V_BIND_SYNC`

## Дальнейшие шаги

Более подробную информацию о новом синтаксисе `v-model` можно получить здесь:

- [Использование `v-model` на компонентах](../component-basics.md#использование-v-model-на-компонентах)
- [Аргументы `v-model`](../component-custom-events.md#аргументы-v-model)
- [Обработка модификаторов `v-model`](../component-custom-events.md#обработка-модификаторов-v-model)
