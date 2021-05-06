# Однофайловые компоненты

## Введение

Во многих проектах, глобальные компоненты объявляются с помощью `app.component` и затем `app.mount('#app')` для указания элемента-контейнера в теле каждой страницы.

Для маленьких и средних проектов, где JavaScript используется лишь на паре страниц, этот подход будет прекрасно работать. Но в более сложных проектах или в случаях, когда весь фронтенд управляется JavaScript становятся явными следующие недостатки:

- **Глобальное определение** заставляет указывать уникальное имя каждому компоненту.
- **В строковых шаблонах** не хватает подсветки синтаксиса. Кроме того, для многострочного HTML приходится использовать уродливые конструкции со слэшами для переноса строк.
- **Нет модульной поддержки CSS** — в то время как HTML и JavaScript разбиваются на модули-компоненты, CSS оказывается за бортом.
- **Отсутствие шага сборки** ограничивает использованием HTML и ES5 JavaScript и не позволяет использовать препроцессоры, например Pug (бывший Jade) и Babel.

Эти проблемы решаются с помощью **однофайловых компонентов**, файлов с расширением `.vue`. Их можно использовать с инструментами сборки, такими как Webpack и Browserify.

Вот пример такого файла, который называется `Hello.vue`:

<a href="https://codepen.io/team/Vue/pen/3de13b5cd0133df4ecf307b6cf2c5f94" target="_blank" rel="noopener noreferrer"><img src="/images/sfc.png" width="403" alt="Пример однофайлового компонента (кликните для просмотра кода)" style="display: block; margin: 15px auto; max-width: 100%"></a>

С однофайловыми компонентами получили:

- [полную подсветку синтаксиса](https://github.com/vuejs/awesome-vue#source-code-editing)
- [модули CommonJS](https://webpack.js.org/concepts/modules/#what-is-a-webpack-module)
- [модульный CSS](https://vue-loader.vuejs.org/ru/features/scoped-css.html)

Как говорилось ранее, можно также использовать препроцессоры, такие как Pug, Babel (с модулями ES2015) и Stylus для создания более понятных и функциональных компонентов.

<a href="https://codesandbox.io/s/vue-single-file-component-with-pre-processors-mr3ik?file=/src/App.vue" target="_blank" rel="noopener noreferrer"><img src="/images/sfc-with-preprocessors.png" width="563" alt="Пример однофайлового компонента с использованием пре-процессоров (кликните для просмотра кода)" style="display: block; margin: 15px auto; max-width: 100%"></a>

Перечисленные варианты даны лишь для примера. С тем же успехом можно использовать TypeScript, SCSS, PostCSS, или любые другие пре- или постпроцессоры. При использовании Webpack вместе с `vue-loader` также получаете и прекрасную поддержку CSS-модулей.

### Что насчёт разделения ответственности?

Одна важная вещь, которую следует отметить — **разделение ответственности это не то же самое, что разделение на файлы по типу.** В современной разработке UI обычно вместо разделения кодовой базы на три огромных слоя, что тесно переплетаются друг с другом, имеет больше смысла разделять их на слабо связанные компоненты и компоновать уже их. Внутри компонента, его шаблон, логика и стили неразрывно связаны между собой, что позволяет сделать компонент более цельным и удобным в поддержке.

Если идея однофайловых компонентов не по душе — можно вынести в отдельные файлы JavaScript и CSS и пользоваться возможностями горячей замены модулей и прекомпиляции:

```html
<!-- my-component.vue -->
<template>
  <div>Это будет предварительно скомпилировано</div>
</template>
<script src="./my-component.js"></script>
<style src="./my-component.css"></style>
```

## Начало работы

### Песочница с примером

Начать изучение однофайловых компонентов можно с этого [простого todo приложения](https://codesandbox.io/s/vue-todo-list-app-with-single-file-component-vzkl3?file=/src/App.vue) на CodeSandbox.

### Для новичков в модульных системах сборки JavaScript

С использования `.vue`-компонентов начинается мир продвинутых JavaScript-приложений. А это значит, что нужно будет освоить некоторые дополнительные инструменты, если ещё этого не сделали:

- **Менеджер пакетов Node (npm)**: Прочитайте [руководство по началу работы](https://docs.npmjs.com/packages-and-modules/getting-packages-from-the-registry) и о том как устанавливать пакеты.

- **Современный JavaScript стандартов ES2015/16**: Прочитайте [руководство по ES2015](https://babeljs.io/docs/en/learn) от Babel. Не нужно сразу запоминать все новые возможности, но держите эту страничку под рукой в качестве справочника.

После того как уделили время этим ресурсам, советуем разобраться с [Vue CLI](https://cli.vuejs.org/ru/). Следуйте инструкциям, чтобы создать рабочий проект с `.vue`-компонентами, ES2015, webpack и горячей перезагрузкой модулей.

### Для продвинутых пользователей

О большинстве настроек конфигурации сборки позаботится CLI, но при необходимости всё можно настроить детальнее через соответствующие [опции конфигурации](https://cli.vuejs.org/ru/config/).

Если предпочитаете с нуля создавать конфигурацию сборки, то потребуется вручную настроить webpack для работы с [vue-loader](https://vue-loader.vuejs.org/ru/). Подробнее о webpack можно узнать в [его официальной документации](https://webpack.js.org/configuration/) и на сайте [webpack learning academy](https://webpack.academy/p/the-core-concepts).

### Сборка с Rollup

Большая часть времени разработки сторонней библиотеки уходит на то, чтобы позволить пользователям библиотеки использовать преимущества [tree shaking](https://webpack.js.org/guides/tree-shaking/). Для поддержки tree-shaking требуется сборка в виде `esm` модуля. Поскольку webpack и, соответственно, vue-cli не поддерживают сборку `esm` модулей, потребуется использовать [Rollup](https://rollupjs.org/).

#### Установка Rollup

Сначала необходимо установить Rollup и несколько зависимостей для него:

```bash
npm install --save-dev rollup @rollup/plugin-commonjs rollup-plugin-vue
```

Это минимальный набор плагинов rollup, необходимых для компиляции `esm` модулей. Можно также добавить [rollup-plugin-babel](https://github.com/rollup/plugins/tree/master/packages/babel) для транспиляции кода и [node-resolve](https://github.com/rollup/plugins/tree/master/packages/node-resolve), если используются зависимости, которые должны поставляться в итоговой сборке библиотеки.

#### Конфигурация Rollup

Для конфигурации сборки с Rollup нужно создать файл `rollup.config.js` в корне проекта:

```bash
touch rollup.config.js
```

После создания файла нужно открыть его в любом редакторе и добавить следующий код:

```js
// импорт используемых плагинов
import commonjs from 'rollup-plugin-commonjs'
import VuePlugin from 'rollup-plugin-vue'
import pkg from './package.json' // импорт package.json для переиспользования имени

export default {
  // файл, который будет содержать все экспортируемые компоненты/функции
  input: 'src/index.js',
  // массив выходных форматов
  output: [
    {
      file: pkg.module, // название создаваемой esm библиотеки
      format: 'esm', // выбранный формат
      sourcemap: true, // флаг rollup для генерации sourcemaps
    }
  ],
  // массив плагинов, которые используем
  plugins: [
    commonjs(),
    VuePlugin()
  ],
  // указываем rollup не добавлять Vue в сборку библиотеки
  external: ['vue']
}
```

#### Конфигурирование package.json

Чтобы воспользоваться только что созданным `esm` модулем необходимо добавить в файл `package.json` несколько полей:

```json
 "scripts": {
   ...
   "build": "rollup -c rollup.config.js",
   ...
 },
 "module": "dist/my-library-name.esm.js",
 "files": [
   "dist/",
 ],
 ```

Таким образом уточняем следующее:

- каким образом собирать пакет
- какие файлы хотим упаковать в пакет
- какой файл будет `esm` модулем

#### Сборка `umd` и `cjs` модулей

Можно одновременно собирать и `umd` и `cjs` модули, достаточно добавить несколько строк конфигурации в `rollup.config.js` и `package.json`

##### rollup.config.js

```js
output: [
  ...
   {
      file: pkg.main,
      format: 'cjs',
      sourcemap: true,
    },
    {
      file: pkg.unpkg,
      format: 'umd',
      name: 'MyLibraryName',
      sourcemap: true,
      globals: {
        vue: 'Vue',
      },
    },
]
```

##### package.json

```json
"module": "dist/my-library-name.esm.js",
"main": "dist/my-library-name.cjs.js",
"unpkg": "dist/my-library-name.global.js",
```
