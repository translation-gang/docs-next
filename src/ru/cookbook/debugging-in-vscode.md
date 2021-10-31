# Отладка в VS Code

Со временем каждое приложение достигает отметки когда становится необходимо разбираться в ошибках, от больших до малых. В этом рецепте рассмотрим несколько возможностей для пользователей VS Code как отлаживать приложения в браузере.

В рецепте рассматривается отладка приложения на [Vue CLI](https://github.com/vuejs/vue-cli) в паре с VS Code при их запуске в браузере.

## Подготовка

Убедитесь что установлена VS Code и выбранный браузер.

Разверните проект с помощью [vue-cli](https://github.com/vuejs/vue-cli), следуя инструкциям по установке в [руководстве Vue CLI](https://cli.vuejs.org/ru/). Перейдите в каталог новосозданного приложения и откройте VS Code.

### Отображение исходного кода в браузере

Прежде чем отлаживать компоненты Vue из VS Code нужно обновить сгенерированную конфигурацию Webpack для генерации source maps. Это делается для того, чтобы отладчик смог сопоставлять код из минифицированных файлов со строками в оригинальных. Это обеспечит возможность отладки приложения даже после того, как все ресурсы были оптимизированы с помощью Webpack.

При использовании Vue CLI 2 нужно установить или обновить свойство `devtool` в файле `config/index.js`:

```js
devtool: 'source-map',
```

Если используете Vue CLI 3, то нужно установить или обновить свойство `devtool` в файле `vue.config.js` в корне проекта:

```js
module.exports = {
  configureWebpack: {
    devtool: 'source-map',
  },
}
```

### Запуск приложения из VS Code

:::info Информация
Предполагается, что приложение запущено на порту `8080`. Если это не так (например, если порт `8080` был уже занят и поэтому Vue CLI автоматически выбрал другой порт), то не забудьте изменить соответствующим образом конфигурацию.
:::

Нажмите на значок отладки (слева, узкая боковая панель действий) чтобы открыть представление «Отладка», затем на значок шестерёнки (сверху), чтобы настроить файл launch.json, выберите **Chrome/Firefox: Launch** в качестве окружения. Замените содержимое сгенерированного `launch.json` указанной ниже конфигурацией:

<div style="padding: 10px 25px 30px"><img src="/images/config_add.png" alt="Добавление конфигурации Chrome" style="width: 690px; border-radius: 3px; box-shadow: 0 10px 15px rgb(0 0 0 / 50%)"></div>

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "chrome",
      "request": "launch",
      "name": "vuejs: chrome",
      "url": "http://localhost:8080",
      "webRoot": "${workspaceFolder}/src",
      "breakOnLoad": true,
      "sourceMapPathOverrides": {
        "webpack:///src/*": "${webRoot}/*"
      }
    },
    {
      "type": "firefox",
      "request": "launch",
      "name": "vuejs: firefox",
      "url": "http://localhost:8080",
      "webRoot": "${workspaceFolder}/src",
      "pathMappings": [{ "url": "webpack:///src/", "path": "${webRoot}/" }]
    }
  ]
}
```

## Добавление точки останова

1. Добавьте точку останова, например в **src/components/HelloWorld.vue** на строке `90`.

<div style="padding: 10px 25px 30px"><img src="/images/breakpoint_set.png" alt="Добавление точки останова" style="width: 690px; border-radius: 3px; box-shadow: 0 10px 15px rgb(0 0 0 / 50%)"></div>

2. Откройте консоль в корневом каталоге приложения и запустите его с помощью Vue CLI:

```bash
npm run serve
```

3. Перейдите в режим отладки, выберите конфигурацию **'vuejs: chrome/firefox'**, затем нажмите F5 или зелёную кнопку для старта.

4. Эта точка останова должна быть достигнута, когда новый экземпляр браузера откроет `http://localhost:8080`.

<div style="padding: 10px 25px 30px"><img src="/images/breakpoint_hit.png" alt="Достижение точки останова" style="width: 690px; border-radius: 3px; box-shadow: 0 10px 15px rgb(0 0 0 / 50%)"></div>

## Альтернативные варианты

### Vue devtools

Есть и другие методы отладки, отличающиеся по сложности. Самый популярный и простой из них — использовать Vue.js devtools [для Chrome](https://chrome.google.com/webstore/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd) и [для Firefox](https://addons.mozilla.org/en-US/firefox/addon/vue-js-devtools/). Главное преимущество работы с инструментами разработки состоит в том, что они позволяют в реальном времени менять данные в свойствах и сразу видеть изменения на странице. Другим важным плюсом является возможность путешествий во времени (time travel debugging) для Vuex.

![Devtools Timetravel Debugger](/images/devtools-timetravel.gif)

Обратите внимание, если на странице используется production/минифицированная сборка Vue.js (например подключаете стандартную ссылка на CDN), то интеграция инструментов разработки по умолчанию отключена и панель Vue не будет отображаться в инструментах разработчика. При переключении на не-минифицированную версию может потребоваться обновить страницу, чтобы увидеть изменения.

### Использование debugger в коде

Примеры выше — отличные рабочие варианты для отладки. Однако, есть и альтернативный вариант, когда можно просто использовать [нативное выражение debugger](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Statements/debugger) прямо в коде. При их использовании важно не забыть удалить эти выражения после окончания отладки.

```vue
<script>
export default {
  data() {
    return {
      message: ''
    }
  },
  mounted() {
    const hello = 'Hello World!'
    debugger
    this.message = hello
  }
};
</script>
```

## Благодарности

Этот рецепт был основан на примере [Kenneth Auchenberg](https://twitter.com/auchenberg), который [доступен здесь](https://github.com/Microsoft/VSCode-recipes/tree/master/vuejs-cli).
