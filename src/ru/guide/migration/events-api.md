---
badges:
  - breaking
---

# Удалено API для событий <MigrationBadges :badges="$frontmatter.badges" />

## Обзор

Методы экземпляров `$on`, `$off` и `$once` были удалены. Экземпляры приложения больше не реализуют интерфейс эмиттера событий.

## Синтаксис в 2.x

В версии 2.x, экземпляр Vue можно использовать для вызова обработчиков, подключённых через API событий (`$on`, `$off` и `$once`). Это использовалось для создания _шины событий_ и создания глобальных прослушивателей событий, используемых во всем приложении:

```js
// eventHub.js

const eventHub = new Vue()

export default eventHub
```

```js
// ChildComponent.vue
import eventHub from './eventHub.js'

export default {
  mounted() {
    // добавление слушателя в шину событий
    eventHub.$on('custom-event', () => {
      console.log('Вызвано пользовательское событие!')
    })
  },
  beforeDestroy() {
    // удаление слушателя из шины событий
    eventHub.$off('custom-event')
  }
}
```

```js
// ParentComponent.vue
import eventHub from './eventHub.js'

export default {
  methods: {
    callGlobalCustomEvent() {
      eventHub.$emit('custom-event') // если ChildComponent примонтирован, то появится сообщение в консоли
    }
  }
}
```

## Что изменилось в 3.x

Из экземпляра полностью удалены методы `$on`, `$off` и `$once`. Метод `$emit` всё ещё является частью существующего API, так как он используется для запуска обработчиков событий, декларативно прикреплённых к родительским компонентам.

## Стратегия миграции

Во Vue 3 больше нет возможности использовать эти API для прослушивания генерируемых событий в компоненте в других компонентах, для такого случая использования нет пути миграции.

Но паттерн шины событий можно использовать с помощью внешней библиотеки, реализующей интерфейс эмиттера событий, например [mitt](https://github.com/developit/mitt) или [tiny-emitter](https://github.com/scottcorgan/tiny-emitter).

Например:

```js
// eventHub.js
import emitter from 'tiny-emitter/instance'

export default {
  $on: (...args) => emitter.on(...args),
  $once: (...args) => emitter.once(...args),
  $off: (...args) => emitter.off(...args),
  $emit: (...args) => emitter.emit(...args),
}
```

Это обеспечивает такой же API событий, как и во Vue 2.

Эти методы также могут поддерживаться в специальной сборке для совместимости Vue 3 в будущем.
