# Динамические и асинхронные компоненты

> Подразумевается, что вы уже изучили и разобрались с разделом [Основы компонентов](component-basics.md). Если нет — прочитайте его сначала.

## Динамические компоненты с `keep-alive`

Ранее, атрибут `is` использовался для переключения между компонентами в интерфейсе с вкладками:

```vue
<component :is="currentTabComponent"></component>
```

Однако при переключении между этими компонентами может возникнуть необходимость сохранить их состояние или избежать их перерисовку в целях производительности. Например при небольшой доработки интерфейса с вкладками:

<common-codepen-snippet title="Dynamic components: without keep-alive" slug="jOPjZOe" tab="html,result" :preview="false" />

Можно заметить, что если выбрать пост, переключиться на вкладку _Archive_, а затем снова вернуться на _Posts_, то выбранный пост больше не отображается. Так происходит потому, что каждый раз, когда переключаемся на новую вкладку Vue будет создавать новый экземпляр `currentTabComponent`.

Пересоздание динамических компонентов обычно полезно, но в данном случае хотелось бы чтобы экземпляры компонентов вкладок кэшировались после их создания в первый раз. Для решения этой проблемы можно обернуть динамический компонент в `<keep-alive>`:

```vue
<!-- Неактивные компоненты будут закэшированы! -->
<keep-alive>
  <component :is="currentTabComponent"></component>
</keep-alive>
```

Проверим результат ниже:

<common-codepen-snippet title="Dynamic components: with keep-alive" slug="VwLJQvP" tab="html,result" :preview="false" />

Теперь вкладка _Posts_ сохраняет своё состояние (выбранный пост) даже тогда, когда не отрисовывается.

Подробнее о компоненте `<keep-alive>` можно узнать в [справочнике API](../api/built-in-components.md#keep-alive).

## Асинхронные компоненты

В больших приложениях может потребоваться разделить приложение на более мелкие части и загружать компоненты с сервера только тогда, когда они необходимы. Чтобы реализовать подобное Vue предоставляет метод `defineAsyncComponent`:

```js
const app = Vue.createApp({})

const AsyncComp = Vue.defineAsyncComponent(
  () =>
    new Promise((resolve, reject) => {
      resolve({
        template: '<div>I am async!</div>'
      })
    })
)

app.component('async-example', AsyncComp)
```

Как можно увидеть, метод принимает функцию-фабрику, возвращающую `Promise`. В Promise коллбэк `resolve` должен быть вызван, когда получено определение компонента с сервера. Также можно вызвать `reject(reason)` для обработки неудачи при загрузке.

Можно также возвращать `Promise` в функции-фабрике, так что с Webpack 2 или более поздней версии и синтаксисом ES2015 можно сделать так:

```js
import { defineAsyncComponent } from 'vue'

const AsyncComp = defineAsyncComponent(() =>
  import('./components/AsyncComponent.vue')
)

app.component('async-component', AsyncComp)
```

Также можно использовать `defineAsyncComponent` при [локальной регистрации компонента](component-registration.md#local-registration):

```js
import { createApp, defineAsyncComponent } from 'vue'

createApp({
  // ...
  components: {
    AsyncComponent: defineAsyncComponent(() =>
      import('./components/AsyncComponent.vue')
    )
  }
})
```

### Использование с Suspense

Асинхронные компоненты по умолчанию считаются _suspensible_. Это означает, что если имеется `<Suspense>` в родительской цепочке, то компонент будет рассматриваться как асинхронная зависимость от этого `<Suspense>`. В таком случае состояние загрузки будет контролироваться `<Suspense>`, а собственные опции компонента для загрузки, ошибки, задержки и таймаута будут проигнорированы.

Асинхронный компонент может отказаться от управления `Suspense` и всегда использовать собственное состояние для загрузки, с помощью опции `suspensible: false`.

Список доступных опций можно посмотреть в [справочнике API](../api/global-api.md#arguments-4)
