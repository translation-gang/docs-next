# Конфигурация сервера

Для [структуры кода](structure.md) и [конфигурации webpack](build-config.md), описанных ранее, также потребуются и некоторые изменения в коде сервера Express.

- необходимо создать приложение с собранным `entry-server.js` из итоговой сборки. Путь к нему можно найти с помощью манифеста webpack:

  ```js
  // server.js
  const path = require('path')
  const manifest = require('./dist/server/ssr-manifest.json')

  // Имя 'app.js' получается из имени точки входа с добавленным суффиксом `.js`
  const appPath = path.join(__dirname, './dist', 'server', manifest['app.js'])
  const createApp = require(appPath).default
  ```

- требуется определить правильные пути ко всем используемым ресурсам:

  ```js
  // server.js
  server.use(
    '/img',
    express.static(path.join(__dirname, './dist/client', 'img'))
  )
  server.use(
    '/js',
    express.static(path.join(__dirname, './dist/client', 'js'))
  )
  server.use(
    '/css',
    express.static(path.join(__dirname, './dist/client', 'css'))
  )
  server.use(
    '/favicon.ico',
    express.static(path.join(__dirname, './dist/client', 'favicon.ico'))
  )
  ```

- нужно заменять содержимое `index.html` на содержимое приложения с отрисовкой на стороне сервера:

  ```js
  // server.js
  const indexTemplate = fs.readFileSync(
    path.join(__dirname, '/dist/client/index.html'),
    'utf-8'
  )

  server.get('*', async (req, res) => {
    const { app } = createApp()

    const appContent = await renderToString(app)

    const html = indexTemplate
      .toString()
      .replace('<div id="app">', `<div id="app">${appContent}`)

    res.setHeader('Content-Type', 'text/html')
    res.send(html)
  })
  ```

Полный код сервера Express представлен ниже:

```js
const path = require('path')
const express = require('express')
const fs = require('fs')
const { renderToString } = require('@vue/server-renderer')
const manifest = require('./dist/server/ssr-manifest.json')

const server = express()

const appPath = path.join(__dirname, './dist', 'server', manifest['app.js'])
const createApp = require(appPath).default

server.use('/img', express.static(path.join(__dirname, './dist/client', 'img')))
server.use('/js', express.static(path.join(__dirname, './dist/client', 'js')))
server.use('/css', express.static(path.join(__dirname, './dist/client', 'css')))
server.use('/favicon.ico', express.static(path.join(__dirname, './dist/client', 'favicon.ico')))

server.get('*', async (req, res) => {
  const { app } = createApp()

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

console.log('Сервер запущен по адресу: http://localhost:8080')

server.listen(8080)
```
