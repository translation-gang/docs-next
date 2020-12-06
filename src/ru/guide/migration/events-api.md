---
badges:
  - breaking
---

# Удалено API событий <MigrationBadges :badges="$frontmatter.badges" />

## Обзор

Методы экземпляров `$on`, `$off` и `$once` были удалены. Экземпляры приложения больше не реализуют интерфейс эмиттера событий.

## Синтаксис в 2.x

В версии 2.x, экземпляр Vue можно было использовать для запуска обработчиков, подключённых через API эмиттера событий (`$on`, `$off` и `$once`). Это использовалось для создания _шины событий_ чтобы создавать глобальные слушатели событий, используемые во всем приложении:

```js
// eventHub.js

const eventHub = new Vue()

export default eventHub
```

```js
// ChildComponent.vue
import eventHub from './eventHub'

export default {
  mounted() {
    // добавление слушателя в eventHub
    eventHub.$on('custom-event', () => {
      console.log('Custom event triggered!')
    })
  },
  beforeDestroy() {
    // удаление слушателя из eventHub
    eventHub.$off('custom-event')
  }
}
```

```js
// ParentComponent.vue
import eventHub from './eventHub'

export default {
  methods: {
    callGlobalCustomEvent() {
      eventHub.$emit('custom-event') // если ChildComponent примонтирован, то появится сообщение в консоли
    }
  }
}
```

## Что изменилось в 3.x

Были полностью удалены методы экземпляра `$on`, `$off` и `$once`. Метод `$emit` всё ещё является частью существующего API, так как он используется для запуска обработчиков событий, декларативно прикреплённых родительским компонентом.

## Стратегия миграции

Существующие шины событий могут быть заменены на реализацию внешней библиотеки, реализующей интерфейс эмиттера событий, например [mitt](https://github.com/developit/mitt).

Эти методы также могут поддерживаться в специальных сборках для совместимости.
