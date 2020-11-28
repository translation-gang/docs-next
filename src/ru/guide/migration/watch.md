---
title: Watch on Arrays
badges:
  - breaking
---

# {{ $frontmatter.title }} <MigrationBadges :badges="$frontmatter.badges" />

## Обзор

- **КАРДИНАЛЬНОЕ ИЗМЕНЕНИЕ:** When watching an array, the callback will only trigger when the array is replaced. If you need to to trigger on mutation, the `deep` option must be specified.

## Синтаксис в 3.x

When using [the `watch` option](/api/options-data.html#watch) to watch an array, the callback will only trigger when the array is replaced. In other words, the watch callback will no longer be triggered on array mutation. To trigger on mutation, the `deep` option must be specified.

```js
watch: {
  bookList: {
    handler(val, oldVal) {
      console.log('book list changed')
    },
    deep: true
  },
}
```

## Стратегия миграции

If you rely on watching array mutations, add the `deep` property to ensure that your callback is triggered correctly.
