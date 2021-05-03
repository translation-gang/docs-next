# Роутинг

## Официальный роутер

Для большинства одностраничных приложений (SPA) рекомендуется использовать официально поддерживаемую [библиотеку vue-router](https://github.com/vuejs/vue-router-next). Более подробную информацию можно найти в [документации библиотеки](https://next.router.vuejs.org/).

## Создание простой маршрутизации с нуля

Если нужна только очень простая маршрутизация и не хочется для этого использовать полнофункциональную библиотеку, то её можно реализовать динамической отрисовкой компонента на уровне страницы, например так:

```js
const { createApp, h } = Vue

const NotFoundComponent = { template: '<p>Страница не найдена</p>' }
const HomeComponent = { template: '<p>Главная страница</p>' }
const AboutComponent = { template: '<p>Страница о нас</p>' }

const routes = {
  '/': HomeComponent,
  '/about': AboutComponent
}

const SimpleRouter = {
  data: () => ({
    currentRoute: window.location.pathname
  }),

  computed: {
    CurrentComponent() {
      return routes[this.currentRoute] || NotFoundComponent
    }
  },

  render() {
    return h(this.CurrentComponent)
  }
})

createApp(SimpleRouter).mount('#app')
```

В сочетании с [History API](https://developer.mozilla.org/en-US/docs/Web/API/History_API/Working_with_the_History_API) можно создать очень простой, но рабочий маршрутизатор на стороне клиента. Чтобы увидеть на практике, посмотрите на [пример этого приложения](https://github.com/phanan/vue-3.0-simple-routing-example).

## Интеграция сторонних роутеров

Если предпочитаете использовать сторонний роутер, например [Page.js](https://github.com/visionmedia/page.js) или [Director](https://github.com/flatiron/director), то интеграция [будет такой же простой](https://github.com/phanan/vue-3.0-simple-routing-example/compare/master...pagejs). Вот [полный пример](https://github.com/phanan/vue-3.0-simple-routing-example/tree/pagejs) с использованием Page.js.
