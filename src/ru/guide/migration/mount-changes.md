---
title: 'Изменения в API монтирования'
badges:
  - breaking
---

# Монтируемое приложение не заменяет элемент <MigrationBadges :badges="$frontmatter.badges" />

## Обзор

Во Vue 2.x при монтировании приложения у которого есть `template`, отрисованное содержимое будет заменять элемент, к которому монтируется. Во Vue 3.x отрисованное приложение добавляется дочерним к такому элементу, заменяя у элемента `innerHTML`.

## Синтаксис в 2.x

Во Vue 2.x передаётся селектор HTML-элемента в `new Vue()` или `$mount`:

```js
new Vue({
  el: '#app',
  data() {
    return {
      message: 'Привет Vue!'
    }
  },
  template: `
    <div id="rendered">{{ message }}</div>
  `
})

// или
const app = new Vue({
  data() {
    return {
      message: 'Привет Vue!'
    }
  },
  template: `
    <div id="rendered">{{ message }}</div>
  `
})

app.$mount('#app')
```

При монтировании такого приложения на страницу, на которой будет `div` с указанным селектором (в данном случае `id="app"`):

```html
<body>
  <div id="app">
    Какая-то информация приложения
  </div>
</body>
```

в результате указанный `div` будет заменён на содержимое отрисовываемого приложения:

```html
<body>
  <div id="rendered">Привет Vue!</div>
</body>
```

## Синтаксис в 3.x

Во Vue 3.x при монтировании приложения его отрисовываемое содержимое будет заменять `innerHTML` элемента, который передаётся в `mount`:

```js
const app = Vue.createApp({
  data() {
    return {
      message: 'Привет Vue!'
    }
  },
  template: `
    <div id="rendered">{{ message }}</div>
  `
})

app.mount('#app')
```

При монтировании такого приложения на страницу, на которой будет `div` с указанным селектором (в данном случае `id="app"`), результат будет следующим:

```html
<body>
  <div id="app" data-v-app="">
    <div id="rendered">Привет Vue!</div>
  </div>
</body>
```

## Стратегия миграции

[Флаг сборки для миграции: `GLOBAL_MOUNT_CONTAINER`](migration-build.md#конфигурация-совместимости)

## См. также

- [`mount` API](../../api/application-api.md#mount)
