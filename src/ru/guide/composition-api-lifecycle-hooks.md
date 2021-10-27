# Хуки жизненного цикла

> В разделе в примерах кода используется синтаксис [однофайловых компонентов](single-file-component.md)

> Подразумевается, что уже изучили и разобрались с разделами [Введение в Composition API](composition-api-introduction.md) и [Основы реактивности](reactivity-fundamentals.md). Если нет — прочитайте их сначала.

<VideoLesson href="https://www.vuemastery.com/courses/vue-3-essentials/lifecycle-hooks" title="Посмотрите бесплатное видео о том как работать с хуками жизненного цикла">Посмотрите бесплатное видео о том как работать с хуками жизненного цикла на Vue Mastery</VideoLesson>

Хуки жизненного цикла компонента доступны по тем же именам, но с префиксом `on`.

В таблице ниже указано, как нужно именовать хуки жизненного цикла внутри [setup()](composition-api-setup.md):

| Options API       | Хук внутри `setup`  |
|-------------------|---------------------|
| `beforeCreate`    | Не нужен\*          |
| `created`         | Не нужен\*          |
| `beforeMount`     | `onBeforeMount`     |
| `mounted`         | `onMounted`         |
| `beforeUpdate`    | `onBeforeUpdate`    |
| `updated`         | `onUpdated`         |
| `beforeUnmount`   | `onBeforeUnmount`   |
| `unmounted`       | `onUnmounted`       |
| `errorCaptured`   | `onErrorCaptured`   |
| `renderTracked`   | `onRenderTracked`   |
| `renderTriggered` | `onRenderTriggered` |
| `activated`       | `onActivated`       |
| `deactivated`     | `onDeactivated`     |

:::tip Совет
Поскольку `setup` запускается приблизительно как и хуки `beforeCreate` и `created`, то и не требуется их явно определять. Другими словами, любой код для этих хуков можно указать непосредственно в функции `setup`.
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
