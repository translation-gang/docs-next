---
title: Удалено свойство $listeners
badges:
  - breaking
---

# Удалено свойство `$listeners` <MigrationBadges :badges="$frontmatter.badges" />

## Обзор

Объект `$listeners` удалён в 3.x. Отслеживание событий теперь является частью `$attrs`:

```js
{
  text: 'это какой-то атрибут',
  onClose: () => console.log('произошло событие close')
}
```

## Синтаксис в 2.x

Во Vue 2 можно получить доступ к атрибутам, переданным в компонент через `this.$attrs`, а также к обработчикам событий через `this.$listeners`. Вместе с `inheritAttrs: false` это позволяло разработчику применить эти атрибуты и обработчики событий к какому-то другому элементу вместо корневого:

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

В виртуальном DOM Vue 3 обработчики событий теперь просто атрибуты с префиксом `on` и являются частью объекта `$attrs`, поэтому `$listeners` были удалены.

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

Если компонент получит атрибут `id` и обработчик события `@close`, то объект `$attrs` теперь будет выглядеть так:

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
