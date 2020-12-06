---
badges:
  - removed
---

# Фильтры удалены <MigrationBadges :badges="$frontmatter.badges" />

## Обзор

Фильтры были удалены из Vue 3.x и больше не поддерживаются.

## Синтаксис в 2.x

В 2.x можно было использовать фильтры для часто используемого форматирования текста.

Например:

```vue
<template>
  <h1>Bank Account Balance</h1>
  <p>{{ accountBalance | currencyUSD }}</p>
</template>

<script>
  export default {
    props: {
      accountBalance: {
        type: Number,
        required: true
      }
    },
    filters: {
      currencyUSD(value) {
        return '$' + value
      }
    }
  }
</script>
```

Хоть это и кажется удобным, но требует дополнительного синтаксиса, который нарушает ожидания, что выражения внутри фигурных скобок это «обычный JavaScript», что увеличивает затраты на обучение и реализацию.

## Что изменилось в 3.x

В 3.x фильтры удалены и больше не поддерживаются. Вместо них рекомендуем использовать вызовы методов или вычисляемые свойства.

Для примера выше это может быть реализовано таким образом:

```vue
<template>
  <h1>Bank Account Balance</h1>
  <p>{{ accountInUSD }}</p>
</template>

<script>
  export default {
    props: {
      accountBalance: {
        type: Number,
        required: true
      }
    },
    computed: {
      accountInUSD() {
        return '$' + this.accountBalance
      }
    }
  }
</script>
```

## Стратегия миграции

Использование фильтров заменить на вычисляемые свойства или вызовы методов.

### Глобальные фильтры

Если использовались глобально зарегистрированные фильтры, которые затем повсеместно использовались в приложении, то вряд ли будет удобно их заменять вычисляемыми свойствами или методами в каждом компоненте.

Можно сделать такие фильтры доступными для всех компонентов через [globalProperties](../../api/application-config.md#globalproperties):

```js
// main.js
const app = createApp(App)

app.config.globalProperties.$filters = {
  currencyUSD(value) {
    return '$' + value
  }
}
```

Тогда потребуется исправить все шаблоны, используя объект `$filters`:

```html
<template>
  <h1>Bank Account Balance</h1>
  <p>{{ $filters.currencyUSD(accountBalance) }}</p>
</template>
```

Однако при таком подходе можно использовать только методы, но не вычисляемые свойства, поскольку они имеют смысл лишь тогда, когда определены в контексте отдельного компонента.
