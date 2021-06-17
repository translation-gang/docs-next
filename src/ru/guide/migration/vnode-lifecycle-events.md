---
badges:
  - breaking
---

# События хуков жизненного цикла VNode <MigrationBadges :badges="$frontmatter.badges" />

## Обзор

Во Vue 2 можно использовать события для отслеживания ключевых этапов жизненного цикла компонента. Эти события именуются с префикса `hook:`, после которого указывается имя соответствующего хука жизненного цикла.

Во Vue 3 префикс был переименован на `vnode-`. Кроме того, теперь эти события доступны как для компонентов, так и для обычных HTML-элементов.

## Синтаксис в 2.x

Во Vue 2 имя события совпадает с именем хука жизненного цикла с префиксом `hook:`:

```html
<template>
  <child-component @hook:updated="onUpdated">
</template>
```

## Синтаксис в 3.x

Во Vue 3 имя события теперь должно быть с префиксом `vnode-`:

```html
<template>
  <child-component @vnode-updated="onUpdated">
</template>
```

Или просто `vnode`, при использовании записи в camelCase:

```html
<template>
  <child-component @vnodeUpdated="onUpdated">
</template>
```

## Стратегия миграции

В большинстве случаев потребуется просто изменить префикс. Хуки жизненного цикла `beforeDestroy` и `destroyed` были переименованы соответственно в `beforeUnmount` и `unmounted`, поэтому для них имена событий также потребуется обновить.

[Флаг сборки для миграции: `INSTANCE_EVENT_HOOKS`](migration-build.md#конфигурация-совместимости)

## См. также

- [Руководство по миграции — удалено API для событий](../migration/events-api.md)
