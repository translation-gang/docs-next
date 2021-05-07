# Маршрутизация и разделение кода

## Маршрутизация с помощью `vue-router`

Можно заметить, что в серверном коде использован обработчик `*`, который будет обрабатывать все запросы. Это позволяет передавать посещённый URL в приложение Vue и переиспользовать одну конфигурацию роутинга для клиента и сервера!

Рекомендуется использовать официальную библиотеку [vue-router](https://github.com/vuejs/vue-router-next) для этих целей. Давайте сначала создадим файл, в котором создадим маршрутизатор. Обратите внимание, что аналогично экземпляру приложения, здесь также для каждого запроса требуется свежий экземпляр маршрутизатора, поэтому файл экспортирует функцию `createRouter`:

```js
// router.js
import { createRouter, createMemoryHistory, createWebHistory } from 'vue-router'
import MyUser from './components/MyUser.vue'

const isServer = typeof window === 'undefined'

const history = isServer ? createMemoryHistory() : createWebHistory()

const routes = [{ path: '/user', component: MyUser }]

export default function() {
  return createRouter({ routes, history })
}
```

И обновить файл `app.js`, клиентскую и серверную точки входа:

```js{4,8,10,14}
// app.js
import { createSSRApp } from 'vue'
import App from './App.vue'
import createRouter from './router'

export default function(args) {
  const app = createSSRApp(App)
  const router = createRouter()

  app.use(router)

  return {
    app,
    router
  }
}
```

```js{2}
// entry-client.js
const { app, router } = createApp({
  /*...*/
})
```

```js{2}
// entry-server.js
const { app, router } = createApp({
  /*...*/
})
```

## Разделение кода

Разделение кода или ленивая загрузка частей приложения помогает уменьшить размер ресурсов, которые требуется загрузить браузеру для первоначальной отрисовки, и может значительно улучшить TTI (time-to-interactive) для приложений, чьи сборки много весят. Ключ в том, чтобы «загружать только то, что необходимо» для стартового экрана.

Vue Router предоставляет [поддержку lazy-loading](https://next.router.vuejs.org/guide/advanced/lazy-loading.html), позволяя [webpack разделять код в этой точке](https://webpack.js.org/guides/code-splitting-async/). Всё что нужно сделать это:

```js
// изменить это...
import MyUser from './components/MyUser.vue'
const routes = [
  { path: '/user', component: MyUser }
]

// на это:
const routes = [
  { path: '/user', component: () => import('./components/MyUser.vue') }
]
```

Как на клиенте, так и на сервере необходимо дожидаться разрешения маршрутизатором асинхронных компонентов для маршрута, чтобы корректно вызывать навигационные хуки в компонентах. Для этого используется метод [router.isReady](https://next.router.vuejs.org/api/#isready). Давайте обновим файл точки входа клиента:

```js{8,10}
// entry-client.js
import createApp from './app'

const { app, router } = createApp({
  /* ... */
})

router.isReady().then(() => {
  app.mount('#app')
})
```

Также необходимо обновить скрипт `server.js`:

```js{11}
// server.js
const path = require('path')

const appPath = path.join(__dirname, './dist', 'server', manifest['app.js'])
const createApp = require(appPath).default

server.get('*', async (req, res) => {
  const { app, router } = createApp()

  router.push(req.url)
  await router.isReady()

  const appContent = await renderToString(app)

  fs.readFile(path.join(__dirname, '/dist/client/index.html'), (err, html) => {
    if (err) {
      throw err
    }

    html = html
      .toString()
      .replace('<div id="app">', `<div id="app">${appContent}`)
    res.setHeader('Content-Type', 'text/html')
    res.send(html)
  })
})
```
