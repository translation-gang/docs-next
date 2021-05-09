# Введение

:::info Информация
Новичок во Vue.js? Начинать изучение лучше с [руководства](../introduction.md).
:::

Это руководство в первую очередь предназначено для пользователей с опытом работы во Vue 2, которые хотят узнать о новых функциях и изменениях во Vue 3. **Это не то, с чего надо начинать изучение и читать от начала до конца, прежде чем пробовать Vue 3.** Несмотря на кажущиеся многочисленные изменения, многое что уже знаете и любите во Vue осталось прежним; просто хотим как можно основательнее и подробнее дать объяснения и примеры для каждого произошедшего изменения.

- [Быстрый старт](#быстрыи-старт)
- [Важные новые возможности](#важные-новые-возможности)
- [Кардинальные изменения](#кардинальные-изменения)
- [Поддержка библиотек](#поддержка-библиотек)

## Обзор

<br>
<iframe src="https://player.vimeo.com/video/440868720" width="640" height="360" frameborder="0" allow="autoplay; fullscreen" allowfullscreen></iframe>

Начать изучение Vue 3 на [Vue Mastery](https://www.vuemastery.com/courses-path/vue3) (на англ.).

## Быстрый старт

- Через CDN: `<script src="https://unpkg.com/vue@next"></script>`
- Песочница в браузере на [Codepen](https://codepen.io/yyx990803/pen/OJNoaZL)
- Песочница в браузере на [CodeSandbox](https://v3.vue.new)
- Развернуть проект с помощью [Vite](https://github.com/vitejs/vite):

  ```bash
  npm init @vitejs/app hello-vue3 # ИЛИ yarn create @vitejs/app hello-vue3
  ```

- Развернуть проект с помощью [vue-cli](https://cli.vuejs.org/ru/):

  ```bash
  npm install -g @vue/cli # ИЛИ yarn global add @vue/cli
  vue create hello-vue3
  # выбрать пресет vue 3
  ```

## Важные новые возможности

Некоторые из новых функций, которые появились во Vue 3:

- [Composition API](../composition-api-introduction.md)
- [Телепорты](../teleport.md)
- [Добавлены фрагменты](fragments.md)
- [Добавлена опция emits в компонентах](../component-custom-events.md)
- [`createRenderer` API из `@vue/runtime-core`](https://github.com/vuejs/vue-next/tree/master/packages/runtime-core) для создания пользовательских рендеров
- [SFC Синтаксический сахар для Composition API (`<script setup>`)](https://github.com/vuejs/rfcs/blob/script-setup-2/active-rfcs/0000-script-setup.md) <Badge text="экспериментально" type="warning" />
- [SFC CSS-переменные на основе состояния (`v-bind` in `<style>`)](https://github.com/vuejs/rfcs/blob/style-vars-2/active-rfcs/0000-sfc-style-variables.md) <Badge text="экспериментально" type="warning" />
- [SFC `<style scoped>` теперь могут содержать глобальные правила или правила только для содержимого слотов](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0023-scoped-styles-changes.md)
- [Добавлен компонент suspense](suspense.md) <Badge text="экспериментально" type="warning" />

## Кардинальные изменения

:::info Информация
Всё ещё ведётся работа над созданием специальной сборки для перехода на Vue 3 с поведением, совместимым с Vue 2, и выводом предупреждений о предстоящих несовместимостях при миграции. Если планируете миграцию большого приложения на Vue 2, рекомендуемся дождаться этой сборки для более спокойного перехода.
:::

Ниже приведён список кардинальных изменений в сравнении с 2.x:

### Глобальное API

- [Глобальное API Vue теперь применяется к экземпляру приложения](global-api.md)
- [Глобальное и внутреннее API были реорганизованы для поддержки tree-shaking](global-api-treeshaking.md)

### Директивы в шаблонах

- [Использование `v-model` на компонентах было переработано, заменяя `v-bind.sync`](v-model.md)
- [Изменения использования`key` на `<template v-for>` и не-`v-for` узлах](key-attribute.md)
- [Изменён приоритет `v-if` и `v-for` при использовании на одном элементе](v-if-v-for.md)
- [Теперь учитывается порядок при использовании `v-bind="object"`](v-bind.md)
- [Удалён модификатор `v-on:event.native`](v-on-native-modifier-removed.md)
- [Использование `ref` внутри `v-for` больше не регистрирует массив ссылок](array-refs.md)

### Компоненты

- [Функциональные компоненты могут создаваться только простыми функциями](functional-components.md)
- [Удалён атрибут `functional` для `<template>` однофайловых компонентов, а также удалена опция `functional` в компонентах](functional-components.md)
- [Создание асинхронных компонентов теперь с помощью метода `defineAsyncComponent`](async-components.md)
- [События компонента теперь должны описываться с помощью опции `emits`](emits-option.md)

### Render-функции

- [Изменён API render-функций](render-function-api.md)
- [Удалено свойство `$scopedSlots`, теперь все слоты доступны как функции через `$slots`](slots-unification.md)
- [Удалено свойство `$listeners` / теперь всё в `$attrs`](listeners-removed.md)
- [Свойство `$attrs` теперь содержит `class` и `style`](attrs-includes-class-style.md)

### Пользовательские элементы

- [Проверка пользовательских элементов теперь происходит на этапе компиляции шаблона](custom-elements-interop.md)
- [Использование специального атрибута `is` ограничено тегом `<component>`](custom-elements-interop.md#модифицированные-встроенные-элементы)

### Другие незначительные изменения

- Хук жизненного цикла `destroyed` переименован в `unmounted`
- Хук жизненного цикла `beforeDestroy` переименован в `beforeUnmount`
- [Удалён доступ к `this` в функции значения по умолчанию входного параметра](props-default-this.md)
- [Переименованы функции хуков директив, чтобы лучше соответствовать жизненному циклу компонента](custom-directives.md)
- [Опцию `data` необходимо указывать только функцией](data-option.md)
- [Опция `data` из примесей теперь объединяется неглубоко](data-option.md#изменение-поведения-при-слиянии-с-примесями)
- [Изменено поведение при приведении значения атрибутов](attribute-coercion.md)
- [Переименованы некоторые классы transition](transition.md)
- [Корневой элемент `<TransitionGroup>` больше не создаётся по умолчанию](transition-group.md)
- [Отслеживание изменений массива с помощью watch будет вызывать обработчик только при замене массива. При необходимости отслеживаний изменений необходимо указывать опцию `deep`.](watch.md)
- Теги `<template>` без специальных директив (`v-if/else-if/else`, `v-for` или `v-slot`) теперь обрабатываются как обычные элементы и в результате будет отрисован нативный элемент  `<template>` вместо отрисовки его содержимого
- [Монтируемое приложение не заменяет элемент, к которому монтируется](mount-changes.md)
- [Префикс событий жизненных хуков `hook:` был изменён на `vnode-`](vnode-lifecycle-events.md)

### Удалённые API

- [Удалена поддержка `keyCode` в модификаторах `v-on`](keycode-modifiers.md)
- [Удалены методы экземпляра $on, $off и $once](events-api.md)
- [Удалены фильтры](filters.md)
- [Удалена поддержка атрибута inline-template](inline-template-attribute.md)
- [Удалено свойство экземпляра `$children`](children.md)
- [Удалена опция `propsData`](props-data.md)
- Удалён метод экземпляра `$destroy`. Пользователи больше не должны вручную управлять жизненным циклом отдельных компонентов Vue.
- Удалены глобальные функции `set` и `delete` и методы экземпляра `$set` и `$delete`. При использовании прокси для отслеживания изменений они больше не нужны.

## Поддержка библиотек

Все официальные библиотеки и инструменты поддерживают Vue 3, но некоторые из них ещё находятся в бета-версии или в статусе релиз-кандидата. Подробная информация по отдельным библиотекам будет ниже. Многие из них на данный момент распространяются под тегом `next` в npm. **Переключение на использование тега `latest` планируется, когда все официальные библиотеки будут иметь совместимые и стабильные версии.**

### Vue CLI

<a href="https://www.npmjs.com/package/@vue/cli" target="_blank" noopener noreferrer><img src="https://img.shields.io/npm/v/@vue/cli"></a>

Начиная с версии v4.5.0, `vue-cli` предоставляет из коробки опцию для выбора Vue 3 при создании нового проекта. Можно обновить `vue-cli` и выполнить команду `vue create` для создания проекта на Vue 3 уже сегодня.

- [Документация](https://cli.vuejs.org/ru/)
- [GitHub](https://github.com/vuejs/vue-cli)

### Vue Router

<a href="https://www.npmjs.com/package/vue-router/v/next" target="_blank" noopener noreferrer><img src="https://img.shields.io/npm/v/vue-router/next.svg"></a>

Vue Router 4.0 предоставляет поддержку Vue 3 и имеет ряд собственных кардинальных изменений. Для получения более подробной информации ознакомьтесь с его [руководством по миграции](https://next.router.vuejs.org/guide/migration/).

- [Документация](https://next.router.vuejs.org/)
- [GitHub](https://github.com/vuejs/vue-router-next)
- [RFCs](https://github.com/vuejs/rfcs/pulls?q=is%3Apr+is%3Amerged+label%3Arouter)

### Vuex

<a href="https://www.npmjs.com/package/vuex/v/next" target="_blank" noopener noreferrer><img src="https://img.shields.io/npm/v/vuex/next.svg"></a>

Vuex 4.0 предоставляет поддержку Vue 3 с тем же самым API, что и 3.x. Единственное кардинальное изменение заключается в том [как устанавливается плагин](https://next.vuex.vuejs.org/guide/migrating-to-4-0-from-3-x.html#breaking-changes).

- [GitHub](https://github.com/vuejs/vuex/tree/4.0)

### Расширение инструментов для разработчика

Сейчас идёт работа над новой версией Devtools с новым интерфейсом и переработанной внутренней частью для поддержки нескольких версий Vue. Новая версия в настоящее время находится в бета-тестировании и поддерживает только Vue 3 (на данный момент). Также ведётся работа над интеграцией обновлённых версий Vuex и Router.

- Для Chrome: [Установить из магазина приложений Chrome](https://chrome.google.com/webstore/detail/vuejs-devtools/ljjemllljcmogpfapbkkighbhhppjdbg?hl=en)

  - Примечание: канал обновлений с бета-версиями может конфликтовать со стабильной версией devtools, поэтому может потребоваться отключить стабильную версию для корректной работы канала обновлений с бета-версиями.

- Для Firefox: [Скачать подписанное расширение](https://github.com/vuejs/vue-devtools/releases/tag/v6.0.0-beta.2) (файл `.xpi` в Assets)

### Поддержка IDE

Рекомендуется использовать [VSCode](https://code.visualstudio.com/) с нашим официальным расширением [Vetur](https://marketplace.visualstudio.com/items?itemName=octref.vetur), которое обеспечивает IDE всестороннюю поддержку Vue 3.

### Другие проекты

| Проект                | npm                           | Репозиторий          |
| --------------------- | ----------------------------- | -------------------- |
| @vue/babel-plugin-jsx | [![rc][jsx-badge]][jsx-npm]   | [[GitHub][jsx-code]] |
| eslint-plugin-vue     | [![ga][epv-badge]][epv-npm]   | [[GitHub][epv-code]] |
| @vue/test-utils       | [![beta][vtu-badge]][vtu-npm] | [[GitHub][vtu-code]] |
| vue-class-component   | [![beta][vcc-badge]][vcc-npm] | [[GitHub][vcc-code]] |
| vue-loader            | [![rc][vl-badge]][vl-npm]     | [[GitHub][vl-code]]  |
| rollup-plugin-vue     | [![beta][rpv-badge]][rpv-npm] | [[GitHub][rpv-code]] |

[jsx-badge]: https://img.shields.io/npm/v/@vue/babel-plugin-jsx.svg
[jsx-npm]: https://www.npmjs.com/package/@vue/babel-plugin-jsx
[jsx-code]: https://github.com/vuejs/jsx-next
[vd-badge]: https://img.shields.io/npm/v/@vue/devtools/beta.svg
[vd-npm]: https://www.npmjs.com/package/@vue/devtools/v/beta
[vd-code]: https://github.com/vuejs/vue-devtools/tree/next
[epv-badge]: https://img.shields.io/npm/v/eslint-plugin-vue.svg
[epv-npm]: https://www.npmjs.com/package/eslint-plugin-vue
[epv-code]: https://github.com/vuejs/eslint-plugin-vue
[vtu-badge]: https://img.shields.io/npm/v/@vue/test-utils/next.svg
[vtu-npm]: https://www.npmjs.com/package/@vue/test-utils/v/next
[vtu-code]: https://github.com/vuejs/vue-test-utils-next
[jsx-badge]: https://img.shields.io/npm/v/@ant-design-vue/babel-plugin-jsx.svg
[jsx-npm]: https://www.npmjs.com/package/@ant-design-vue/babel-plugin-jsx
[jsx-code]: https://github.com/vueComponent/jsx
[vcc-badge]: https://img.shields.io/npm/v/vue-class-component/next.svg
[vcc-npm]: https://www.npmjs.com/package/vue-class-component/v/next
[vcc-code]: https://github.com/vuejs/vue-class-component/tree/next
[vl-badge]: https://img.shields.io/npm/v/vue-loader/next.svg
[vl-npm]: https://www.npmjs.com/package/vue-loader/v/next
[vl-code]: https://github.com/vuejs/vue-loader/tree/next
[rpv-badge]: https://img.shields.io/npm/v/rollup-plugin-vue/next.svg
[rpv-npm]: https://www.npmjs.com/package/rollup-plugin-vue/v/next
[rpv-code]: https://github.com/vuejs/rollup-plugin-vue/tree/next

:::info Информация
Для получения дополнительной информации о совместимости библиотек и плагинов с Vue 3, обязательно ознакомьтесь с [этим issue в awesome-vue](https://github.com/vuejs/awesome-vue/issues/3544).
:::
