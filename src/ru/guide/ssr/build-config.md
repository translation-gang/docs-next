# Конфигурация сборки

Конфигурация webpack для проекта с SSR будет аналогична конфигурации проекта только для клиентской стороны. Если ещё не знакомы с настройкой webpack, более подробную информацию можно найти в документации [Vue CLI](https://cli.vuejs.org/ru/guide/webpack.html) или [настроить Vue Loader вручную](https://vue-loader.vuejs.org/ru/guide/#%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B0-%D0%B2%D1%80%D1%83%D1%87%D0%BD%D1%83%D1%8E).

## Ключевые отличия от сборки только для клиента

1. Необходимо создать [манифест webpack](https://webpack.js.org/concepts/manifest/) для серверной части кода. Это JSON-файл, который webpack держит для отслеживания того, как все модули сопоставляются с получившимися сборками.

2. Требуется [экстернализация зависимостей приложения](https://webpack.js.org/configuration/externals/). Это делает сборку сервера намного быстрее и создаёт меньший по размеру файл. При этом необходимо исключить зависимости, которые должны быть обработаны webpack (например, файлы `.css`. или `.vue`).

3. Нужно изменить [target](https://webpack.js.org/concepts/targets/) в webpack на Node.js. Это позволяет webpack обрабатывать динамические импорты в соответствии с подходом в Node, а также укажет `vue-loader` выдавать серверно-ориентированный код при компиляции компонентов Vue.

4. При создании серверной точки входа, необходимо определить переменную окружения, чтобы указать что работаем с SSR. Может быть полезно добавить несколько строк в `scripts` в `package.json` проекта:

```json
"scripts": {
  "build:client": "vue-cli-service build --dest dist/client",
  "build:server": "SSR=1 vue-cli-service build --dest dist/server",
  "build": "npm run build:client && npm run build:server",
}
```

## Пример конфигурации

Ниже приведён пример `vue.config.js`, который добавляет SSR в проект Vue CLI, но его можно адаптировать для любой сборки webpack.

```js
const { WebpackManifestPlugin } = require('webpack-manifest-plugin')
const nodeExternals = require('webpack-node-externals')
const webpack = require('webpack')

module.exports = {
  chainWebpack: webpackConfig => {
    // Необходимо отключить cache-loader, иначе в сборке для клиента
    // будут использовать кэшированные компоненты из сборки для сервера
    webpackConfig.module.rule('vue').uses.delete('cache-loader')
    webpackConfig.module.rule('js').uses.delete('cache-loader')
    webpackConfig.module.rule('ts').uses.delete('cache-loader')
    webpackConfig.module.rule('tsx').uses.delete('cache-loader')

    if (!process.env.SSR) {
      // Указываем точку входа клиентской части приложения
      webpackConfig
        .entry('app')
        .clear()
        .add('./src/entry-client.js')
      return
    }

    // Указываем точку входа серверной части приложения
    webpackConfig
      .entry('app')
      .clear()
      .add('./src/entry-server.js')

    // Это позволяет webpack обрабатывать динамические импорты в соответствии
    // с подходом в Node, а также указывает `vue-loader` выдавать
    // серверно-ориентированный код при компиляции компонентов Vue.
    webpackConfig.target('node')
    // Это указывает сборке для сервера использовать экспорты в стиле Node
    webpackConfig.output.libraryTarget('commonjs2')

    webpackConfig
      .plugin('manifest')
      .use(new WebpackManifestPlugin({ fileName: 'ssr-manifest.json' }))

    // https://webpack.js.org/configuration/externals/#function
    // https://github.com/liady/webpack-node-externals
    // Экстернализация зависимостей приложения. Это сделает сборку для сервера
    // гораздо быстрее и создаст более легкий файл итоговой сборки.

    // Нужно не экстернализировать зависимости, которые должны быть обработаны webpack.
    // Также следует внести в белый список зависимости, которые изменяют `global` (например, полифилы)
    webpackConfig.externals(nodeExternals({ allowlist: /\.(css|vue)$/ }))

    webpackConfig.optimization.splitChunks(false).minimize(false)

    webpackConfig.plugins.delete('preload')
    webpackConfig.plugins.delete('prefetch')
    webpackConfig.plugins.delete('progress')
    webpackConfig.plugins.delete('friendly-errors')

    webpackConfig.plugin('limit').use(
      new webpack.optimize.LimitChunkCountPlugin({
        maxChunks: 1
      })
    )
  }
}
```

## Предостережения для externals

Обратите внимание, что в опции `externals` в белый список добавлены CSS-файлы. Это связано с тем, что CSS импортируемый из зависимостей, всё равно должен обрабатываться webpack. Если импортируете файлы других типов, которые также полагаются на webpack (например, `*.vue`, `*.sass`), их также следует добавить в белый список.

При использовании `runInNewContext: 'once'` или `runInNewContext: true` также необходимо добавить в белый список полифилы, которые изменяют `global`, например, `babel-polyfill`. Это требуется для того, что при использовании режима нового контекста **код внутри сборки для сервера будет иметь свой собственный объект `global`.** Поскольку на сервере он не нужен, проще просто импортировать его в файл клиентской точки входа.

## Генерация `clientManifest`

В дополнение к серверной сборке также можно сгенерировать манифест клиентской сборки. Благодаря клиентскому манифесту и серверной сборке, рендерер будет иметь информацию как о серверной _так и_ о клиентской сборках. Благодаря этому он сможет автоматически определять и внедрять [директивы для preload / prefetch](https://css-tricks.com/prefetching-preloading-prebrowsing/), теги `<link>` и `<script>` в создаваемый HTML.

Выгода здесь двойная:

1. Он может заменить `html-webpack-plugin` для внедрения корректных URL для ресурсов, когда в генерируемых именах файлов присутствуют хэши.

2. При генерации сборки, которая использует возможности webpack по разделению кода, можно обеспечить preload / prefetch необходимых фрагментов, а также интеллектуально внедрять теги `<script>` для требуемых асинхронных фрагментов, чтобы избежать водопада запросов на клиенте, тем самым улучшая TTI (time-to-interactive).
