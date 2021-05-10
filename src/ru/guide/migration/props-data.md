---
badges:
  - removed
---

# Удалена опция `propsData` <MigrationBadges :badges="$frontmatter.badges" />

## Обзор

Опция `propsData`, используемая для передачи входных параметров в экземпляр Vue во время его создания, была удалена. Для передачи входных параметров в корневой компонент приложения во Vue 3 теперь нужно использовать второй аргумент [createApp](../../api/global-api.md#createapp).

## Синтаксис в 2.x

В версии 2.x можно передавать входные параметры экземпляру Vue во время его создания:

```js
const Comp = Vue.extend({
  props: ['username'],
  template: '<div>{{ username }}</div>'
})

new Comp({
  propsData: {
    username: 'Evan'
  }
})
```

## Изменения в 3.x

Опция `propsData` была удалена. При необходимости передать входные параметры в экземпляр корневого компонента следует использовать второй аргумент `createApp`:

```js
const app = createApp(
  {
    props: ['username'],
    template: '<div>{{ username }}</div>'
  },
  { username: 'Evan' }
)
```
