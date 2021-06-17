---
badges:
  - removed
---

# Удалено свойство $children <MigrationBadges :badges="$frontmatter.badges" />

## Обзор

Свойство экземпляра `$children` было удалено из Vue 3.x и больше не поддерживается.

## Синтаксис в 2.x

В версии 2.x, к дочерним компонентам текущего экземпляра можно получить доступ через свойство `this.$children`:

```vue
<template>
  <div>
    <img alt="Vue logo" src="./assets/logo.png">
    <my-button>Изменить логотип</my-button>
  </div>
</template>

<script>
import MyButton from './MyButton'

export default {
  components: {
    MyButton
  },
  mounted() {
    console.log(this.$children) // [VueComponent]
  }
}
</script>
```

## Изменения в 3.x

В версии 3.x, свойство `$children` было удалено и больше не поддерживается. Вместо этого, при необходимости получить доступ к экземпляру дочернего компонента рекомендуется использовать [$refs](../component-template-refs.md).

## Стратегия миграции

[Флаг сборки для миграции: `INSTANCE_CHILDREN`](migration-build.md#конфигурация-совместимости)
