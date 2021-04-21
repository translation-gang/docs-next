# Хуки жизненного цикла

> Подразумевается, что уже изучили и разобрались с разделами [Введение в Composition API](composition-api-introduction.md) и [Основы реактивности](reactivity-fundamentals.md). Если нет — прочитайте их сначала.

<VideoLesson href="https://www.vuemastery.com/courses/vue-3-essentials/lifecycle-hooks" title="Посмотрите бесплатное видео о том как работать с хуками жизненного цикла">Посмотрите бесплатное видео о том как работать с хуками жизненного цикла на Vue Mastery</VideoLesson>

Хуки жизненного цикла компонента доступны с префиксом «on» к имени хука.

В таблице ниже перечислено, как нужно именовать хуки жизненного цикла внутри [setup()](composition-api-setup.md):

| Options API       | Хук внутри `setup`         |
| ----------------- | -------------------------- |
| `beforeCreate`    | Не нужен\*                 |
| `created`         | Не нужен\*                 |
| `beforeMount`     | `onBeforeMount`            |
| `mounted`         | `onMounted`                |
| `beforeUpdate`    | `onBeforeUpdate`           |
| `updated`         | `onUpdated`                |
| `beforeUnmount`   | `onBeforeUnmount`          |
| `unmounted`       | `onUnmounted`              |
| `errorCaptured`   | `onErrorCaptured`          |
| `renderTracked`   | `onRenderTracked`          |
| `renderTriggered` | `onRenderTriggered`        |

:::tip Совет
Поскольку `setup` запускается приблизительно как и хуки `beforeCreate` и `created`, то не требуется явно их определять. Другими словами, любой код для этих хуков стоит писать непосредственно в функции `setup`.
:::

Эти функции принимают коллбэк, который будет выполнен при вызове хука компонентом:

```js
// MyBook.vue

export default {
  setup() {
    // mounted
    onMounted(() => {
      console.log('Компонент примонтирован!')
    })
  }
}
```
