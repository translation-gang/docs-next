---
badges:
  - new
---

# Асинхронные компоненты <MigrationBadges :badges="$frontmatter.badges" />

## Обзор

Краткий обзор того, что изменилось:

- Новый вспомогательный метод `defineAsyncComponent`, который явно определяет асинхронные компоненты
- Опция `component` переименована в `loader`
- Функция загрузки не принимает аргументы `resolve` и `reject` и должна возвращать Promise

Для более подробного объяснения, читайте дальше!

## Введение

Ранее асинхронные компоненты создавались путём определения функции, которая возвращает Promise, например так:

```js
const asyncPage = () => import('./NextPage.vue')
```

Или, при использовании продвинутого синтаксиса с опциями:

```js
const asyncPage = {
  component: () => import('./NextPage.vue'),
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
const asyncPage = defineAsyncComponent(() => import('./NextPage.vue'))

// Асинхронный компонент с дополнительными опциями
const asyncPageWithOptions = defineAsyncComponent({
  loader: () => import('./NextPage.vue'),
  delay: 200,
  timeout: 3000,
  errorComponent: ErrorComponent,
  loadingComponent: LoadingComponent
})
```

Другое изменение заключается в том, что опция `component` теперь переименована в `loader` для более точного указания на то, что определение компонента не может предоставляться напрямую.

```js{4}
import { defineAsyncComponent } from 'vue'

const asyncPageWithOptions = defineAsyncComponent({
  loader: () => import('./NextPage.vue'),
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

- [Руководство: Динамические и асинхронные компоненты](../component-dynamic-async.md#dynamic-components-with-keep-alive)
