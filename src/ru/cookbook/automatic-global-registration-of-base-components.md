# Автоматическая глобальная регистрация базовых компонентов

## Базовый пример

Многие из компонентов будут относительно общими, возможно, оборачивающими лишь элемент, например поле ввода или кнопку. Их обычно называют [базовыми компонентами](../style-guide/#именование-базовых-компонентов-настоятельно-рекомендуется) и, как правило, они очень часто используются в компонентах.

В итоге во многих компонентах появляется длинный список импортов базовых компонентов:

```js
import BaseButton from './BaseButton.vue'
import BaseIcon from './BaseIcon.vue'
import BaseInput from './BaseInput.vue'

export default {
  components: {
    BaseButton,
    BaseIcon,
    BaseInput
  }
}
```

Только чтобы реализовать относительно небольшую разметку в шаблоне:

```html
<BaseInput v-model="searchText" @keydown.enter="search" />
<BaseButton @click="search">
  <BaseIcon name="search" />
</BaseButton>
```

К счастью, при использовании webpack (или [Vue CLI](https://github.com/vuejs/vue-cli), которая использует webpack под капотом), можно использовать `require.context` для глобальной регистрации только этих очень распространённых базовых компонентов. Вот небольшой пример кода, который можно использовать для глобального импорта базовых компонентов в файле точки входа приложения (например, `src/main.js`):

```js
import { createApp } from 'vue'
import upperFirst from 'lodash/upperFirst'
import camelCase from 'lodash/camelCase'
import App from './App.vue'

const app = createApp(App)

const requireComponent = require.context(
  // Относительный путь к каталогу с компонентами
  './components',
  // Выполнять (или нет) ли поиск во вложенных каталогах
  false,
  // Регулярное выражение для сопоставления имён файлов базовых компонентов
  /Base[A-Z]\w+\.(vue|js)$/
)

requireComponent.keys().forEach(fileName => {
  // Получение конфигурации компонента
  const componentConfig = requireComponent(fileName)

  // Получение имени компонента в PascalCase
  const componentName = upperFirst(
    camelCase(
      // Получение имени файла, независимо от глубины каталога
      fileName
        .split('/')
        .pop()
        .replace(/\.\w+$/, '')
    )
  )

  app.component(
    componentName,
    // Поиск опций компонента в `.default`, который будет существовать,
    // если компонент экспортируется с помощью `export default`,
    // а в противном случае — возврат к корню модуля.
    componentConfig.default || componentConfig
  )
})

app.mount('#app')
```
