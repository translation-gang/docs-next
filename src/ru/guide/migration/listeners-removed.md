---
title: $listeners удалены
badges:
  - breaking
---

# `$listeners` удалены <MigrationBadges :badges="$frontmatter.badges" />

## Обзор

Объект `$listeners` был удалён во Vue 3. Отслеживание событий теперь является частью `$attrs`:

```js
{
  text: 'это какой-то атрибут',
  onClose: () => console.log('произошло событие close')
}
```

## Синтаксис в 2.x

In Vue 2, you can access attributes passed to your components with `this.$attrs`, and event listeners with `this.$listeners`.
In combination with `inheritAttrs: false`, they allow the developer to apply these attributes and listeners to some other element instead of the root element:

```html
<template>
  <label>
    <input type="text" v-bind="$attrs" v-on="$listeners" />
  </label>
</template>

<script>
  export default {
    inheritAttrs: false
  }
</script>
```

## Что изменилось в 3.x

In Vue 3's virtual DOM, event listeners are now just attributes, prefixed with `on`, and as such are part of the `$attrs` object, so `$listeners` has been removed.

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

If this component received an `id` attribute and a `v-on:close` listener, the `$attrs` object will now look like this:

```js
{
  id: 'my-input',
  onClose: () => console.log('произошло событие close')
}
```

## Стратегия миграции

Удалить все случаи использования `$listeners`.

## См. также

- [Соответствующий RFC](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0031-attr-fallthrough.md)
- [Руководство по миграции — `$attrs` включает `class` и `style`](attrs-includes-class-style.md)
- [Руководство по миграции — Изменения в API render-функций](render-function-api.md)
- [Руководство по миграции — Новая опция emits](emits-option.md)
- [Руководство по миграции — Модификатор `.native` удалён](v-on-native-modifier-removed.md)
