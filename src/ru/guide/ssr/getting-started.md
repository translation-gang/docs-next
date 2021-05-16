# Начало работы

> Данное руководство находится в стадии активной разработки

## Установка

Для создания приложения с отрисовкой на стороне сервера потребуется сначала установить пакет `@vue/server-renderer`:

```bash
npm install @vue/server-renderer
## ИЛИ
yarn add @vue/server-renderer
```

#### Примечания

- Рекомендуется использовать Node.js версии 12+.
- Версии `@vue/server-renderer` и `vue` **должны совпадать**.
- `@vue/server-renderer` полагается на некоторые встроенные модули Node.js и поэтому может использоваться только с Node.js. В будущем возможно появится более простая сборка, которая сможет быть запущена и в других средах выполнения JavaScript.

## Отрисовка приложения Vue

В отличие от клиентского приложения Vue, которое создаётся с помощью `createApp`, приложение с SSR нужно создавать с использованием `createSSRApp`:

```js
const { createSSRApp } = require('vue')

const app = createSSRApp({
  data() {
    return {
      user: 'Василий Пупкин'
    }
  },
  template: `<div>Текущий пользователь: {{ user }}</div>`
})
```

Теперь, можно использовать функцию `renderToString` для отрисовки экземпляра приложения в строку. Она возвращает Promise, который разрешается отрисованным HTML.

```js{2,13}
const { createSSRApp } = require('vue')
const { renderToString } = require('@vue/server-renderer')

const app = createSSRApp({
  data() {
    return {
      user: 'Василий Пупкин'
    }
  },
  template: `<div>Текущий пользователь: {{ user }}</div>`
})

const appContent = await renderToString(app)
```

## Интеграция с сервером

В этом руководстве для запуска приложения будет использоваться [Express](https://expressjs.com/):

```bash
npm install express
## ИЛИ
yarn add express
```

```js
// server.js

const { createSSRApp } = require('vue')
const { renderToString } = require('@vue/server-renderer')
const server = require('express')()

server.get('*', async (req, res) => {
  const app = createSSRApp({
    data() {
      return {
        user: 'Василий Пупкин'
      }
    },
    template: `<div>Текущий пользователь: {{ user }}</div>`
  })

  const appContent = await renderToString(app)
  const html = `
  <html>
    <body>
      <h1>Мой первый заголовок</h1>
      <div id="app">${appContent}</div>
    </body>
  </html>
  `

  res.end(html)
})

server.listen(8080)
```

При запуске этого скрипта Node.js теперь сможем получить статичную HTML-страницу по адресу `localhost:8080`. Но этот код ещё не _гидратирован_: Vue ещё не взял на себя управление статическим HTML, отправленным сервером, чтобы превратить его в динамический DOM, который может реагировать на изменения данных на стороне клиента. Это будет рассмотрено далее, в разделе [гидратация клиентской части](hydration.md).
