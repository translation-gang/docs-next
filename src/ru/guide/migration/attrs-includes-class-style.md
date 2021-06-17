---
title: Свойство $attrs теперь содержит class и style
badges:
  - breaking
---

# Свойство `$attrs` теперь содержит `class` и `style` <MigrationBadges :badges="$frontmatter.badges" />

## Обзор

Свойство `$attrs` теперь _содержит все атрибуты_, передаваемые компоненту, в том числе `class` и `style`.

## Поведение в 2.x

Атрибуты `class` и `style` получали специальную обработку в реализации виртуального DOM во Vue 2. По этой причине они _не содержались_ в свойстве `$attrs`, в то время как остальные атрибуты были.

Побочный эффект этого проявлялся при использовании `inheritAttrs: false`:

- Атрибуты в свойстве `$attrs` больше не добавлялись автоматически к корневому элементу, оставляя решение за разработчиком, куда их следует добавлять.
- Но атрибуты `class` и `style`, не являясь частью `$attrs`, всё равно применялись к корневому элементу компонента:

  ```vue
  <template>
    <label>
      <input type="text" v-bind="$attrs" />
    </label>
  </template>

  <script>
  export default {
    inheritAttrs: false
  }
  </script>
  ```

  при таком использовании:

  ```html
  <my-component id="my-id" class="my-class"></my-component>
  ```

  ...получался следующий HTML:

  ```html
  <label class="my-class">
    <input type="text" id="my-id" />
  </label>
  ```

## Поведение в 3.x

Свойство `$attrs` содержит _все атрибуты_, что облегчает их применение к разным элементам. Для примера выше будет сгенерирован следующий HTML:

```html
<label>
  <input type="text" id="my-id" class="my-class" />
</label>
```

## Стратегия миграции

В компонентах, использующих `inheritAttrs: false`, следует убедиться что стилизация работает как и задумано. Если раньше полагались на специальное поведение для `class` и `style`, то некоторые визуальные стили могут быть нарушены, поскольку эти атрибуты теперь будут применяться к другому элементу.

[Флаг сборки для миграции: `INSTANCE_ATTRS_CLASS_STYLE`](migration-build.md#конфигурация-совместимости)

## См. также

- [Соответствующий RFC](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0031-attr-fallthrough.md)
- [Руководство по миграции — `$listeners` удалены](listeners-removed.md)
- [Руководство по миграции — Новая опция emits](emits-option.md)
- [Руководство по миграции — Модификатор `.native` удалён](v-on-native-modifier-removed.md)
- [Руководство по миграции — Изменения в API render-функций](render-function-api.md)
