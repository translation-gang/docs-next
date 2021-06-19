# Сборка для миграции

## Обзор

`@vue/compat` (она же «сборка для миграции») — специальная сборка Vue 3, которая обеспечивает настраиваемое поведение для совместимости с Vue 2.

По умолчанию сборка для миграции работает в режиме Vue 2 — большинство публичных API ведут себя точно также, как и во Vue 2, за несколькими исключениями. Использование возможностей, которые были изменены или объявлены устаревшими во Vue 3 будут выдавать предупреждения во время выполнения. Совместимость для определённых возможностей также можно включать/выключать на уровне конкретного компонента.

### Предполагаемые сценарии использования

- Обновление приложения Vue 2 на Vue 3 (с некоторыми [ограничениями](#известные-ограничения))
- Миграция библиотеки для поддержки Vue 3
- Для опытных разработчиков Vue 2, которые ещё не пробовали Vue 3, сборка для миграции может быть использована вместо Vue 3, чтобы помочь в изучении различий между версиями.

### Известные ограничения

Хотя сборку для миграции старались сделать максимально похожей поведением на Vue 2, есть некоторые ограничения, которые могут препятствовать обновлению приложения:

- Зависимости, которые полагаются на внутренние API Vue 2 или недокументированное поведение. Наиболее распространённым случаем является использование приватных свойств на `VNodes`. Если проект полагается на такие библиотеки компонентов, как [Vuetify](https://vuetifyjs.com/en/), [Quasar](https://quasar.dev/) или [ElementUI](https://element.eleme.io/#/en-US), то лучше сначала дождаться их новых версий, совместимых с Vue 3.

- Поддержка Internet Explorer 11: [Vue 3 официально отказался от поддержки IE11](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0038-vue3-ie11-support.md). Если всё ещё нужна поддержка IE11 или ниже, то придётся остаться на Vue 2.

- Отрисовка на стороне сервера (SSR): сборка для миграции может быть использована для SSR, но миграция пользовательской конфигурации SSR намного сложнее. В общем, идея заключается в замене `vue-server-renderer` на [`@vue/server-renderer`](https://github.com/vuejs/vue-next/tree/master/packages/server-renderer). Vue 3 больше не предоставляет bundle renderer и рекомендуется использовать Vue 3 SSR с [Vite](https://vitejs.dev/guide/ssr.html). Если используете [Nuxt.js](https://nuxtjs.org/), то вероятно лучше подождать Nuxt 3.

### Ожидания

Обратите внимание, что сборка для миграции нацелена только на публично документированные API и поведение Vue 2. Если приложение не работает со сборкой для миграции из-за использования недокументированного поведения, маловероятно, что мы изменим сборку для миграции, чтобы удовлетворить ваш конкретный случай. Вместо этого рассмотрите возможность рефакторинга, чтобы устранить зависимость от данного поведения.

Предостережение: если приложение большое и сложное, то миграция, скорее всего, также будет сложной даже со сборкой для миграции. Если приложение, к сожалению, не подходит для обновления, обратите внимание, что планируется сделать портировать Composition API и некоторые другие функции Vue 3 в релизе 2.7 (предположительно в конце третьего квартала 2021).

Если удалось запустить приложение на сборке для миграции, то его **можно** опубликовать в production до завершения миграции. Несмотря на небольшие накладные расходы на производительность/размер, это не должно заметно повлиять на UX в production. Это может потребоваться, если есть зависимости, которые зависят от поведения Vue 2 и не могут быть обновлены/заменены.

Сборка для миграции станет предоставляться, начиная с версии 3.1 и будет продолжать обновляться вместе с релизами 3.2. Публикация обновлений сборки для миграции будет прекращена в одной из будущих минорных версий (не ранее конца 2021 года), поэтому до этого времени стоит запланировать переход на стандартную сборку.

## Процесс обновления

Следующий процесс обновления описывает шаги по миграции фактического приложения Vue 2 (Vue HackerNews 2.0) на Vue 3. Полные коммиты можно найти [здесь](https://github.com/vuejs/vue-hackernews-2.0/compare/migration). Обратите внимание, что фактические шаги для собственного проекта могут отличаться, а описанные здесь шаги следует рассматривать как общее руководство, а не как строгую инструкцию.

### Подготовка

- При использовании [устаревшего синтаксиса именованных слотов / слотов с ограниченной областью видимости](https://ru.vuejs.org/v2/guide/components-slots.html#%D0%A3%D1%81%D1%82%D0%B0%D1%80%D0%B5%D0%B2%D1%88%D0%B8%D0%B9-%D1%81%D0%B8%D0%BD%D1%82%D0%B0%D0%BA%D1%81%D0%B8%D1%81), сначала обновите его до нового синтаксиса (который уже поддерживается в версии 2.6).

### Установка

1. Обновите инструментарий, если это возможно.

   - При использовании пользовательской настройки webpack: Обновите `vue-loader` до версии `^16.0.0`.
   - При использовании `vue-cli`: обновите `@vue/cli-service` до последней версии с помощью команды `vue upgrade`
   - (Альтернатива) Мигрируйте на [Vite](https://vitejs.dev/) + [vite-plugin-vue2](https://github.com/underfin/vite-plugin-vue2). [[Пример коммита](https://github.com/vuejs/vue-hackernews-2.0/commit/565b948919eb58f22a32afca7e321b490cb3b074)]

2. В файле `package.json`, обновите `vue` до 3.1. Установите `@vue/compat` такой же версии, замените `vue-template-compiler` (если присутствует) на `@vue/compiler-sfc`:

   ```diff
   "dependencies": {
   -  "vue": "^2.6.12",
   +  "vue": "^3.1.0",
   +  "@vue/compat": "^3.1.0"
      ...
   },
   "devDependencies": {
   -  "vue-template-compiler": "^2.6.12"
   +  "@vue/compiler-sfc": "^3.1.0"
   }
   ```

   [Пример коммита](https://github.com/vuejs/vue-hackernews-2.0/commit/14f6f1879b43f8610add60342661bf915f5c4b20)

3. В конфигурации сборки установите псевдоним для `vue` на `@vue/compat` и включите режим совместимости через опции компилятора Vue.

    **Примеры конфигураций**

    <details>
      <summary><b>vue-cli</b></summary>

    ```js
    // vue.config.js
    module.exports = {
      chainWebpack: config => {
        config.resolve.alias.set('vue', '@vue/compat')

        config.module
          .rule('vue')
          .use('vue-loader')
          .tap(options => {
            return {
              ...options,
              compilerOptions: {
                compatConfig: {
                  MODE: 2
                }
              }
            }
          })
      }
    }
    ```

    </details>

    <details>
      <summary><b>Чистый webpack</b></summary>

    ```js
    // webpack.config.js
    module.exports = {
      resolve: {
        alias: {
          vue: '@vue/compat'
        }
      },
      module: {
        rules: [
          {
            test: /\.vue$/,
            loader: 'vue-loader',
            options: {
              compilerOptions: {
                compatConfig: {
                  MODE: 2
                }
              }
            }
          }
        ]
      }
    }
    ```

    </details>

    <details>
      <summary><b>Vite</b></summary>

    ```js
    // vite.config.js
    export default {
      resolve: {
        alias: {
          vue: '@vue/compat'
        }
      },
      plugins: [
        vue({
          template: {
            compilerOptions: {
              compatConfig: {
                MODE: 2
              }
            }
          }
        })
      ]
    }
    ```

    </details>

4. На этом этапе приложение может столкнуться с некоторыми ошибками / предупреждениями на этапе компиляции (например, при использование фильтров). Исправьте их в первую очередь. Когда все предупреждения компилятора исчезнут, можно переключить компилятор в режим Vue 3.

   [Пример коммита](https://github.com/vuejs/vue-hackernews-2.0/commit/b05d9555f6e115dea7016d7e5a1a80e8f825be52)

5. After fixing the errors, the app should be able to run if it is not subject to the [limitations](#известные-ограничения) mentioned above.

   You will likely see a LOT of warnings from both the command line and the browser console. Here are some general tips:

   - You can filter for specific warnings in the browser console. It's a good idea to use the filter and focus on fixing one item at a time. You can also use negated filters like `-GLOBAL_MOUNT`.

   - You can suppress specific deprecations via [compat configuration](#compat-configuration).

   - Some warnings may be caused by a dependency that you use (e.g. `vue-router`). You can check this from the warning's component trace or stack trace (expanded on click). Focus on fixing the warnings that originate from your own source code first.

   - If you are using `vue-router`, note `<transition>` and `<keep-alive>` will not work with `<router-view>` until you upgrade to `vue-router` v4.

6. Update [`<transition>` class names](transition.html). This is the only feature that does not have a runtime warning. You can do a project-wide search for `.*-enter` and `.*-leave` CSS class names.

   [Пример коммита](https://github.com/vuejs/vue-hackernews-2.0/commit/d300103ba622ae26ac26a82cd688e0f70b6c1d8f)

7. Update app entry to use [new global mounting API](global-api.html#a-new-global-api-createapp).

   [Пример коммита](https://github.com/vuejs/vue-hackernews-2.0/commit/a6e0c9ac7b1f4131908a4b1e43641f608593f714)

8. [Upgrade `vuex` to v4](https://next.vuex.vuejs.org/guide/migrating-to-4-0-from-3-x.html).

   [Пример коммита](https://github.com/vuejs/vue-hackernews-2.0/commit/5bfd4c61ee50f358cd5daebaa584f2c3f91e0205)

9. [Upgrade `vue-router` to v4](https://next.router.vuejs.org/guide/migration/index.html). If you also use `vuex-router-sync`, you can replace it with a store getter.

   After the upgrade, to use `<transition>` and `<keep-alive>` with `<router-view>` requires using the new [scoped-slot based syntax](https://next.router.vuejs.org/guide/migration/index.html#router-view-keep-alive-and-transition).

   [Пример коммита](https://github.com/vuejs/vue-hackernews-2.0/commit/758961e73ac4089890079d4ce14996741cf9344b)

10. Pick off individual warnings. Note some features have conflicting behavior between Vue 2 and Vue 3 - for example, the render function API, or the functional component vs. async component change. To migrate to Vue 3 API without affecting the rest of the application, you can opt-in to Vue 3 behavior on a per-component basis with the [`compatConfig` option](#per-component-config).

    [Пример коммита](https://github.com/vuejs/vue-hackernews-2.0/commit/d0c7d3ae789be71b8fd56ce79cb4cb1f921f893b)

11. When all warnings are fixed, you can remove the migration build and switch to Vue 3 proper. Note you may not be able to do so if you still have dependencies that rely on Vue 2 behavior.

    [Пример коммита](https://github.com/vuejs/vue-hackernews-2.0/commit/9beb45490bc5f938c9e87b4ac1357cfb799565bd)

## Конфигурация совместимости

### Глобальная конфигурация

Compat features can be disabled individually:

```js
import { configureCompat } from 'vue'

// disable compat for certain features
configureCompat({
  FEATURE_ID_A: false,
  FEATURE_ID_B: false
})
```

Alternatively, the entire application can default to Vue 3 behavior, with only certain compat features enabled:

```js
import { configureCompat } from 'vue'

// default everything to Vue 3 behavior, and only enable compat
// for certain features
configureCompat({
  MODE: 3,
  FEATURE_ID_A: true,
  FEATURE_ID_B: true
})
```

### Конфигурация для компонента

A component can use the `compatConfig` option, which expects the same options as the global `configureCompat` method:

```js
export default {
  compatConfig: {
    MODE: 3, // opt-in to Vue 3 behavior for this component only
    FEATURE_ID_A: true // features can also be toggled at component level
  }
  // ...
}
```

### Конфигурация компиляции

Features that start with `COMPILER_` are compiler-specific: if you are using the full build (with in-browser compiler), they can be configured at runtime. However if using a build setup, they must be configured via the `compilerOptions` in the build config instead (see example configs above).

## Перечень возможностей

### Типы совместимости

- ✔ Полная совместимость
- ◐ Частичная совместимость (с ограничениями)
- ⨂ Нет совместимости (только предупреждения)
- ⭘ Только для совместимости (нет предупреждений)

### Нет совместимости

> Требуется исправить заранее или может приводить к ошибкам

| ID                                    | Тип | Описание                                                                | Документация                                                                                     |
|---------------------------------------|-----|-------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------|
| GLOBAL_MOUNT_CONTAINER                | ⨂   | Mounted application does not replace the element it's mounted to        | [ссылка](mount-changes.md)                                                                       |
| CONFIG_DEVTOOLS                       | ⨂   | production devtools is now a build-time flag                            | [ссылка](https://github.com/vuejs/vue-next/tree/master/packages/vue#bundler-build-feature-flags) |
| COMPILER_V_IF_V_FOR_PRECEDENCE        | ⨂   | `v-if` and `v-for` precedence when used on the same element has changed | [ссылка](v-if-v-for.md)                                                                          |
| COMPILER_V_IF_SAME_KEY                | ⨂   | `v-if` branches can no longer have the same key                         | [ссылка](key-attribute.md#использование-на-ветках-с-условием)                                              |
| COMPILER_V_FOR_TEMPLATE_KEY_PLACEMENT | ⨂   | `<template v-for>` key should now be placed on `<template>`             | [ссылка](key-attribute.md#использование-с-template-v-for)                                                  |
| COMPILER_SFC_FUNCTIONAL               | ⨂   | `<template functional>` is no longer supported in SFCs                  | [ссылка](functional-components.md#однофаиловые-компоненты-sfc)                                  |

### Частичная совместимость (с ограничениями)

| ID                       | Тип | Описание                                                                                                                                                                                   | Документация                                                                                  |
|--------------------------|-----|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------|
| CONFIG_IGNORED_ELEMENTS  | ◐   | `config.ignoredElements` is now `config.compilerOptions.isCustomElement` (only in browser compiler build). If using build setup, `isCustomElement` must be passed via build configuration. | [ссылка](global-api.md#своиство-config-ignoredelements-теперь-config-compileroptions-iscustomelement) |
| COMPILER_INLINE_TEMPLATE | ◐   | `inline-template` removed (compat only supported in browser compiler build)                                                                                                                | [ссылка](inline-template-attribute.md)                                                        |
| PROPS_DEFAULT_THIS       | ◐   | props default factory no longer have access to `this` (in compat mode, `this` is not a real instance - it only exposes props, `$options` and injections)                                   | [ссылка](props-default-this.md)                                                               |
| INSTANCE_DESTROY         | ◐   | `$destroy` instance method removed (in compat mode, only supported on root instance)                                                                                                       |                                                                                               |
| GLOBAL_PRIVATE_UTIL      | ◐   | `Vue.util` is private and no longer available                                                                                                                                              |                                                                                               |
| CONFIG_PRODUCTION_TIP    | ◐   | `config.productionTip` no longer necessary                                                                                                                                                 | [ссылка](global-api.md#удалено-своиство-config-productiontip)                                         |
| CONFIG_SILENT            | ◐   | `config.silent` removed                                                                                                                                                                    |                                                                                               |

### Только для совместимости (нет предупреждений)

| ID                 | Тип | Описание                               | Документация            |
|--------------------|-----|----------------------------------------|-------------------------|
| TRANSITION_CLASSES | ⭘   | Transition enter/leave classes changed | [ссылка](transition.md) |

### Полная совместимость

| ID                           | Тип | Описание                                                              | Документация                                                               |
|------------------------------|-----|-----------------------------------------------------------------------|----------------------------------------------------------------------------|
| GLOBAL_MOUNT                 | ✔   | new Vue() -> createApp                                                | [ссылка](global-api.md#монтирование-экземпляра-приложения)                             |
| GLOBAL_EXTEND                | ✔   | Vue.extend removed (use `defineComponent` or `extends` option)        | [ссылка](global-api.md#удален-метод-vue-extend)                                |
| GLOBAL_PROTOTYPE             | ✔   | `Vue.prototype` -> `app.config.globalProperties`                      | [ссылка](global-api.md#своиство-vue-prototype-заменено-на-config-globalproperties) |
| GLOBAL_SET                   | ✔   | `Vue.set` removed (no longer needed)                                  |                                                                            |
| GLOBAL_DELETE                | ✔   | `Vue.delete` removed (no longer needed)                               |                                                                            |
| GLOBAL_OBSERVABLE            | ✔   | `Vue.observable` removed (use `reactive`)                             | [ссылка](../../api/basic-reactivity.md)                                    |
| CONFIG_KEY_CODES             | ✔   | config.keyCodes removed                                               | [ссылка](keycode-modifiers.md)                                             |
| CONFIG_WHITESPACE            | ✔   | In Vue 3 whitespace defaults to `"condense"`                          |                                                                            |
| INSTANCE_SET                 | ✔   | `vm.$set` removed (no longer needed)                                  |                                                                            |
| INSTANCE_DELETE              | ✔   | `vm.$delete` removed (no longer needed)                               |                                                                            |
| INSTANCE_EVENT_EMITTER       | ✔   | `vm.$on`, `vm.$off`, `vm.$once` removed                               | [ссылка](events-api.md)                                                    |
| INSTANCE_EVENT_HOOKS         | ✔   | Instance no longer emits `hook:x` events                              | [ссылка](vnode-lifecycle-events.md)                                        |
| INSTANCE_CHILDREN            | ✔   | `vm.$children` removed                                                | [ссылка](children.md)                                                      |
| INSTANCE_LISTENERS           | ✔   | `vm.$listeners` removed                                               | [ссылка](listeners-removed.md)                                             |
| INSTANCE_SCOPED_SLOTS        | ✔   | `vm.$scopedSlots` removed; `vm.$slots` now exposes functions          | [ссылка](slots-unification.md)                                             |
| INSTANCE_ATTRS_CLASS_STYLE   | ✔   | `$attrs` now includes `class` and `style`                             | [ссылка](attrs-includes-class-style.md)                                    |
| OPTIONS_DATA_FN              | ✔   | `data` must be a function in all cases                                | [ссылка](data-option.md)                                                   |
| OPTIONS_DATA_MERGE           | ✔   | `data` from mixin or extension is now shallow merged                  | [ссылка](data-option.md)                                                   |
| OPTIONS_BEFORE_DESTROY       | ✔   | `beforeDestroy` -> `beforeUnmount`                                    |                                                                            |
| OPTIONS_DESTROYED            | ✔   | `destroyed` -> `unmounted`                                            |                                                                            |
| WATCH_ARRAY                  | ✔   | watching an array no longer triggers on mutation unless deep          | [ссылка](watch.md)                                                         |
| V_FOR_REF                    | ✔   | `ref` inside `v-for` no longer registers array of refs                | [ссылка](array-refs.md)                                                    |
| V_ON_KEYCODE_MODIFIER        | ✔   | `v-on` no longer supports keyCode modifiers                           | [ссылка](keycode-modifiers.md)                                             |
| CUSTOM_DIR                   | ✔   | Custom directive hook names changed                                   | [ссылка](custom-directives.md)                                             |
| ATTR_FALSE_VALUE             | ✔   | No longer removes attribute if binding value is boolean `false`       | [ссылка](attribute-coercion.md)                                            |
| ATTR_ENUMERATED_COERSION     | ✔   | No longer special case enumerated attributes                          | [ссылка](attribute-coercion.md)                                            |
| TRANSITION_GROUP_ROOT        | ✔   | `<transition-group>` no longer renders a root element by default      | [ссылка](transition-group.md)                                              |
| COMPONENT_ASYNC              | ✔   | Async component API changed (now requires `defineAsyncComponent`)     | [ссылка](async-components.md)                                              |
| COMPONENT_FUNCTIONAL         | ✔   | Functional component API changed (now must be plain functions)        | [ссылка](functional-components.md)                                         |
| COMPONENT_V_MODEL            | ✔   | Component v-model reworked                                            | [ссылка](v-model.md)                                                       |
| RENDER_FUNCTION              | ✔   | Render function API changed                                           | [ссылка](render-function-api.md)                                           |
| FILTERS                      | ✔   | Filters removed (this option affects only runtime filter APIs)        | [ссылка](filters.md)                                                       |
| COMPILER_IS_ON_ELEMENT       | ✔   | `is` usage is now restricted to `<component>` only                    | [ссылка](custom-elements-interop.md)                                       |
| COMPILER_V_BIND_SYNC         | ✔   | `v-bind.sync` replaced by `v-model` with arguments                    | [ссылка](v-model.md)                                                       |
| COMPILER_V_BIND_PROP         | ✔   | `v-bind.prop` modifier removed                                        |                                                                            |
| COMPILER_V_BIND_OBJECT_ORDER | ✔   | `v-bind="object"` is now order sensitive                              | [ссылка](v-bind.md)                                                        |
| COMPILER_V_ON_NATIVE         | ✔   | `v-on.native` modifier removed                                        | [ссылка](v-on-native-modifier-removed.md)                                  |
| COMPILER_V_FOR_REF           | ✔   | `ref` in `v-for` (compiler support)                                   |                                                                            |
| COMPILER_NATIVE_TEMPLATE     | ✔   | `<template>` with no special directives now renders as native element |                                                                            |
| COMPILER_FILTERS             | ✔   | filters (compiler support)                                            |                                                                            |
