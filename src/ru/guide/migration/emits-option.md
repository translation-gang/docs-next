---
title: emits Option
badges:
  - new
---

# `emits` Option <MigrationBadges :badges="$frontmatter.badges" />

## Обзор

Vue 3 now offers an `emits` option similar to the existing `props` option. This option can be used to define the events that a component can emit to its parent.

## Поведение в 2.x

In Vue 2, you can define the props that a component received, but you can't declare which events it can emit:

```html
<template>
  <div>
    <p>{{ text }}</p>
    <button v-on:click="$emit('accepted')">OK</button>
  </div>
</template>

<script>
  export default {
    props: ['text']
  }
</script>
```

## Поведение в 3.x

Similar to props, the events that the component emits can now be defined with the `emits` option.

```html
<template>
  <p>{{ text }}</p>
  <button v-on:click="$emit('accepted')">OK</button>
</template>

<script>
  export default {
    props: ['text'],
    emits: ['accepted']
  }
</script>
```

The option also accepts an object notation, which allows the developer to define validators for the arguments that are passed with the emitted event, similar to validators in props definitions.

For more information on this, please read the [API documentation for this feature](../../api/options-data.md#emits).

## Стратегия миграции

It is highly recommended that you document all of the emitted events by your each of components this way because of the [removal of the `.native` modifier](v-on-native-modifier-removed.md).

All events not defined with `emits` are now added as DOM event listeners to the component's root node (unless `inheritAttrs: false` has been set).

### Пример

For components that re-emit native events to their parent, this would now lead to two events being fired:

```vue
<template>
  <p>{{ text }}</p>
  <button v-on:click="$emit('click', $event)">OK</button>
</template>

<script>
export default {
  props: ['text'],
  emits: [] // without declared event
}
</script>
```

When a parent listens for the `click` event on the component:

```html
<my-button v-on:click="handleClick"></my-button>
```

it would now be triggered _twice_:

- Once from `$emit()`
- Once from a native event listener applied to the root element

Here you have two options:

1. Properly declare the `click` event. This is useful if you actually do add some logic to that event handler in `<my-button>`
2. Remove the re-emitting of the event, since the parent can now listen for the native event easily, without adding `.native`. Suitable when you really only re-emit the event anyway.

## См. также

- [Соответствующий RFC](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0030-emits-option.md)
- [Руководство по миграции — Модификатор `.native` удалён](v-on-native-modifier-removed.md)
- [Руководство по миграции — `$listeners` удалены](listeners-removed.md)
- [Руководство по миграции — `$attrs` включает `class` и `style`](attrs-includes-class-style.md)
- [Руководство по миграции — Изменения в API render-функций](render-function-api.md)
