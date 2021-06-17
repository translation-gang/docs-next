---
badges:
  - new
---

# Изменение синтаксиса для асинхронных компонентов <MigrationBadges :badges="$frontmatter.badges" />

## Обзор

Краткий обзор того, что изменилось:

- Новый вспомогательный метод `defineAsyncComponent`, который явно определяет асинхронные компоненты
- Опция `component` переименована в `loader`
- Функция загрузки не принимает аргументы `resolve` и `reject` и должна возвращать Promise

Для более подробного объяснения, читайте дальше!

## Введение

Ранее асинхронные компоненты создавались путём определения функции, которая возвращает Promise, например так:

```js
const asyncModal = () => import('./Modal.vue')
```

Или, при использовании продвинутого синтаксиса с опциями:

```js
const asyncModal = {
  component: () => import('./Modal.vue'),
  delay: 200,
  timeout: 3000,
  error: ErrorComponent,
  loading: LoadingComponent
}
```

## Синтаксис в 3.x

Теперь, во Vue 3, так как функциональные компоненты определяются как чистые функции, то объявление асинхронных компонентов необходимо явно оборачивать в новый вспомогательный метод `defineAsyncComponent`:

```js
import { defineAsyncComponent } from 'vue'
import ErrorComponent from './components/ErrorComponent.vue'
import LoadingComponent from './components/LoadingComponent.vue'

// Асинхронный компонент без опций
const asyncModal = defineAsyncComponent(() => import('./Modal.vue'))

// Асинхронный компонент с дополнительными опциями
const asyncModalWithOptions = defineAsyncComponent({
  loader: () => import('./Modal.vue'),
  delay: 200,
  timeout: 3000,
  errorComponent: ErrorComponent,
  loadingComponent: LoadingComponent
})
```

:::tip Примечание
Vue Router поддерживает аналогичный механизм асинхронной загрузки компонентов маршрута, также именуемый *ленивой загрузкой*. Несмотря на сходство, эта функция отличается от поддержки асинхронных компонентов во Vue. Поэтому **не нужно использовать** `defineAsyncComponent` при настройке компонентов маршрута во Vue Router. Подробнее об этом можно прочитать в разделе [Lazy Loading Routes](https://next.router.vuejs.org/guide/advanced/lazy-loading.html) документации Vue Router.
:::

Другое изменение заключается в том, что опция `component` теперь переименована в `loader` для более точного указания на то, что определение компонента не может предоставляться напрямую.

```js{4}
import { defineAsyncComponent } from 'vue'

const asyncModalWithOptions = defineAsyncComponent({
  loader: () => import('./Modal.vue'),
  delay: 200,
  timeout: 3000,
  errorComponent: ErrorComponent,
  loadingComponent: LoadingComponent
})
```

Кроме того, в отличие от 2.x, функция загрузчика больше не получает аргументы `resolve` и `reject` и должна всегда возвращать Promise.

```js
// Vue 2.x
const oldAsyncComponent = (resolve, reject) => {
  /* ... */
}

// Vue 3.x
const asyncComponent = defineAsyncComponent(
  () =>
    new Promise((resolve, reject) => {
      /* ... */
    })
)
```

Подробная информация об использовании асинхронных компонентов:

- [Руководство: Динамические и асинхронные компоненты](../component-dynamic-async.md#динамические-компоненты-с-keep-alive)
- [Флаг сборки для миграции: `COMPONENT_ASYNC`](migration-build.md#конфигурация-совместимости)
