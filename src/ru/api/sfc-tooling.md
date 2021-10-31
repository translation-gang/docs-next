# Инструментарий SFC

## Онлайн песочницы

Чтобы попробовать однофайловые компоненты Vue не нужно ничего устанавливать — есть множество онлайн-песочниц, которые работают в браузере:

- [Vue SFC песочница](https://sfc.vuejs.org) (официальная, публикуется из последнего коммита)
- [VueUse песочница](https://play.vueuse.org)
- [Vue на CodeSandbox](https://codesandbox.io/s/vue-3)
- [Vue на Repl.it](https://replit.com/@templates/VueJS-with-Vite)
- [Vue на Codepen](https://codepen.io/pen/editor/vue)
- [Vue на StackBlitz](https://stackblitz.com/fork/vue)
- [Vue на Components.studio](https://components.studio/create/vue3)
- [Vue на WebComponents.dev](https://webcomponents.dev/create/cevue)

Также рекомендуем использовать онлайн-песочницы чтобы приложить пример неработающего кода к сообщениям об ошибках.

## Создание проекта

### Vite

[Vite](https://vitejs.dev/) — лёгкий и быстрый инструмент для сборки с отличной поддержкой однофайловых компонентов Vue. Его создал Эван Ю, который также является автором самого Vue! Чтобы начать работу с Vite + Vue, просто запустите команду:

```sh
npm init vite@latest
```

После этого выберите шаблон Vue и следуйте инструкциям.

- Чтобы узнать больше о Vite, ознакомьтесь с [документацией Vite](https://vitejs.dev/guide/).
- Чтобы настроить специфическое для Vue поведение в проекте Vite, например передачу опций компилятору Vue, обратитесь к документации для [@vitejs/plugin-vue](https://github.com/vitejs/vite/tree/main/packages/plugin-vue#readme).

[Песочница SFC](https://sfc.vuejs.org/) также поддерживает загрузку файлов в виде проекта Vite.

### Vue CLI

[Vue CLI](https://cli.vuejs.org/ru/) — официальный инструмент для сборки на основе webpack для проектов Vue. Чтобы начать работу с Vue CLI выполните команды:

```sh
npm install -g @vue/cli
vue create hello-vue
```

- Подробнее о Vue CLI можно прочитать в [документации Vue CLI](https://cli.vuejs.org/ru/guide/installation.html).

### Vite или Vue CLI?

Рекомендуем начинать новые проекты с Vite, поскольку он предоставляет лучший опыт для разработки с точки зрения времени запуска сервера для разработки и производительности горячей перезагрузки модулей ([подробности](https://vitejs.dev/guide/why.html)). Переходите на Vue CLI только в случае, если полагаетесь на специфические функции webpack (например, Module Federation).

При использовании [Rollup](https://rollupjs.org/), можно смело брать Vite, поскольку он использует Rollup для production сборок и поддерживает совместимую с Rollup систему плагинов. [Даже разработчик Rollup рекомендует Vite в качестве обёртки Rollup для веб-разработки](https://twitter.com/lukastaegert/status/1412119729431584774).

## Поддержка IDE

Рекомендуем использовать IDE [VSCode](https://code.visualstudio.com/) + расширение [Volar](https://github.com/johnsoncodehk/volar). Volar предоставляет подсветку синтаксиса и улучшенный IntelliSense в выражениях шаблонов, входных параметров компонентов и даже валидации слотов. Настоятельно рекомендуем эту пару, если хотите получить наилучший возможный опыт работы с однофайловыми компонентами Vue, особенно если также используете TypeScript.

[WebStorm](https://www.jetbrains.com/webstorm/) также предоставляет достойную поддержку однофайловых компонентов Vue. Однако, обратите внимание, что поддержка `<script setup>` пока ещё находится [в стадии разработки](https://youtrack.jetbrains.com/issue/WEB-49000).

Большинство других редакторов имеют созданную сообществом поддержку синтаксиса Vue, но не имеют такого же уровня IntelliSense для кода. В перспективе надеемся, что сможем расширить диапазон поддержки редакторов, за счёт использования [Language Service Protocol](https://microsoft.github.io/language-server-protocol/) поскольку основная логика Volar реализована в виде стандартного языкового сервера.

## Поддержка тестирования

- При использовании Vite рекомендуем [Cypress](https://www.cypress.io/) в качестве программы для запуска тестов, как модульных так и e2e-тестов. Модульные тесты для однофайловых компонентов Vue могут выполняться с помощью [Cypress Component Test Runner](https://www.cypress.io/blog/2021/04/06/introducing-the-cypress-component-test-runner/).

- Vue CLI поставляется с интеграциями для [Jest](https://jestjs.io/) и [Mocha](https://mochajs.org/).

- При самостоятельной настройке Jest для работы с однофайловыми компонентами Vue, ознакомьтесь с [vue-jest](https://github.com/vuejs/vue-jest) — официальный модуль преобразования однофайловых компонентов Vue для Jest.

## Интеграция пользовательских секций

Пользовательские секции компилируются в импорты в тот же файл Vue с различными типами запросов. Обработка этих запросов при импорте зависит от используемого инструмента сборки.

- При использовании Vite следует использовать пользовательский плагин Vite для преобразования сопоставленных пользовательских секций в исполняемый JavaScript. [[Пример](https://github.com/vitejs/vite/tree/main/packages/plugin-vue#example-for-transforming-custom-blocks)]

- При использовании Vue CLI или webpack необходимо настроить загрузчик webpack для преобразования пользовательских секций. [[Пример](https://vue-loader.vuejs.org/ru/guide/custom-blocks.html#%D0%BF%D0%BEn%D1%8C%D0%B7%D0%BE%D0%B2%D0%B0%D1%82%D0%B5n%D1%8C%D1%81%D0%BA%D0%B8%D0%B5-%D0%B1n%D0%BE%D0%BA%D0%B8)]

## Низко-уровневые инструменты

### `@vue/compiler-sfc`

- [Документация](https://github.com/vuejs/vue-next/tree/master/packages/compiler-sfc)

Этот пакет является частью моно-репозитория ядра Vue и всегда публикуется с такой же версией, что и основной пакет `vue`. Как правило, он будет указан как peer dependency от `vue` в проекте. Чтобы обеспечить корректное поведение, его версия всегда должна соответствовать версии `vue` — т.е. каждый раз при обновлении версии `vue`, необходимо также обновить `@vue/compiler-sfc` чтобы снова соответствовать ей.

Пакет предоставляет низкоуровневые утилиты для обработки однофайловых компонентов Vue и предназначен только для разработчиков инструментов, которым необходимо поддерживать однофайловые компоненты Vue в них.

### `@vitejs/plugin-vue`

- [Документация](https://github.com/vitejs/vite/tree/main/packages/plugin-vue)

Официальный плагин, предоставляющий поддержку однофайловых компонентов Vue в Vite.

### `vue-loader`

- [Документация](https://vue-loader.vuejs.org/ru/)

Официальный загрузчик, предоставляющий поддержку однофайловых компонентов Vue в webpack. Если используете Vue CLI, также обратите внимание на [документацию по модификации настроек `vue-loader` во Vue CLI](https://cli.vuejs.org/ru/guide/webpack.html#%D0%B8%D0%B7%D0%BC%D0%B5%D0%BD%D0%B5%D0%BD%D0%B8%D0%B5-%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B5%D0%BA-%D0%B7%D0%B0%D0%B3%D1%80%D1%83%D0%B7%D1%87%D0%B8%D0%BA%D0%B0).
