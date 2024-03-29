# Методы экземпляра

## $watch

- **Аргументы:**

  - `{string | Function} source`
  - `{Function | Object} callback`
  - `{Object} options (опционально)`
    - `{boolean} deep`
    - `{boolean} immediate`
    - `{string} flush`

- **Возвращает:** `{Function} unwatch`

- **Использование:**

  Отслеживание изменений реактивного свойства или функции вычисляемого свойства на экземпляре компонента. Коллбэк будет вызываться с новым и старым значениями. В виде строки можно указывать только имена свойств корневого уровня для `data`, `props` или `computed`. Для сложных выражений или вложенных свойств стоит определять функцию.

- **Пример:**

  ```js
  const app = createApp({
    data() {
      return {
        a: 1,
        b: 2,
        c: {
          d: 3,
          e: 4
        }
      }
    },
    created() {
      // имя свойства корневого уровня
      this.$watch('a', (newVal, oldVal) => {
        // сделать что-нибудь
      })

      // функция для отслеживания одного вложенного свойства
      this.$watch(
        () => this.c.d,
        (newVal, oldVal) => {
          // сделать что-нибудь
        }
      )

      // функция для отслеживания сложного выражения
      this.$watch(
        // каждый раз, когда выражение `this.a + this.b` получит другой
        // результат — будет вызываться коллбэк. Это похоже на отслеживание
        // вычисляемого свойства, но без необходимости его объявлять
        () => this.a + this.b,
        (newVal, oldVal) => {
          // сделать что-нибудь
        }
      )
    }
  })
  ```

  При отслеживании объекта или массива, любые изменения свойств или элементов в них не будут вызывать коллбэк, потому что они ссылаются на один и тот же объект/массив:

  ```js
  const app = createApp({
    data() {
      return {
        article: {
          text: 'Vue крутой!'
        },
        comments: ['Да!', 'Согласен']
      }
    },
    created() {
      this.$watch('article', () => {
        console.log('Заметка обновилась!')
      })

      this.$watch('comments', () => {
        console.log('Комментарии обновились!')
      })
    },
    methods: {
      // Такие методы НЕ ВЫЗОВУТ коллбэк метода-наблюдателя, потому что
      // изменится только свойство объекта/массива, но не объект/массив целиком
      changeArticleText() {
        this.article.text = 'Vue 3 крутой'
      },
      addComment() {
        this.comments.push('Новый комментарий')
      },

      // Такие методы ВЫЗОВУТ коллбэк метода-наблюдателя, потому что
      // заменяется объект/массив целиком
      changeWholeArticle() {
        this.article = { text: 'Vue 3 крутой' }
      },
      clearComments() {
        this.comments = []
      }
    }
  })
  ```

  Метод `$watch` также возвращает функцию для возможности остановить отслеживание:

  ```js
  const app = createApp({
    data() {
      return {
        a: 1
      }
    }
  })

  const vm = app.mount('#app')

  const unwatch = vm.$watch('a', cb)

  // ... когда-то позднее, останавливаем дальнейшее отслеживание
  unwatch()
  ```

- **Опция: deep**

  Для отслеживания изменений вложенных значений внутри объектов нужно передавать в опциях `deep: true`. Её можно также использовать и для отслеживания мутаций массива.

  > Обратите внимание: **при мутации (а не замене)** объекта/массива и их отслеживании с помощью опции deep, старое значение будет таким же, как и новое, потому что они ссылаются на один и тот же объект/массив. Vue не хранит копию значения до мутации.

  ```js
  vm.$watch('someObject', callback, {
    deep: true
  })

  vm.someObject.nestedValue = 123
  // коллбэк будет вызван
  ```

- **Опция: immediate**

  При передаче в опциях `immediate: true`, коллбэк будет вызываться сразу же, с текущим значением отслеживаемого выражения:

  ```js
  vm.$watch('a', callback, {
    immediate: true
  })

  // `callback` будет вызван сразу же, с текущим значением `a`
  ```

  Обратите внимание, при использовании опции `immediate` нет возможности остановить отслеживание при первом вызове коллбэка.

  ```js{6,8}
  // подобное выбросит ОШИБКУ
  const unwatch = vm.$watch(
    'value',
    function() {
      doSomething()
      unwatch()
    },
    { immediate: true }
  )
  ```

  При необходимости останавливать отслеживание в коллбэке следует сначала проверять его доступность:

  ```js{1,7-9}
  let unwatch = null

  unwatch = vm.$watch(
    'value',
    function() {
      doSomething()
      if (unwatch) {
        unwatch()
      }
    },
    { immediate: true }
  )
  ```

- **Опция: flush**

  Опция `flush` позволяет точнее контролировать время вызова коллбэка. Значением может быть `'pre'`, `'post'` или `'sync'`.

  По умолчанию значение `'pre'`. Это значит, что коллбэк должен быть вызван перед отрисовкой. Это позволяет в коллбэке обновить другие значения перед обновлением шаблона.

  Значение `'post'` можно использовать для отложенного вызова коллбэка по окончанию отрисовки. Его следует использовать, если в коллбэке требуется доступ к обновлённому DOM или дочерним компонентам через `$refs`.

  При установке опции `flush` в значение `'sync'`, коллбэк будет вызываться синхронно, как только значение изменится.

  Для значений `'pre'` и `'post'` коллбэк будет буферизироваться с использованием очереди. Он будет добавляться в очередь только один раз, даже если отслеживаемое значение изменилось несколько раз. Промежуточные значения будут пропущены и не будут переданы в коллбэк.

  Буферизация коллбэка позволяет не только улучшить производительность, но и помогает обеспечить консистентность данных. Методы-наблюдатели не будут вызываться до тех пор, пока не завершится код, занимающийся обновлением данных.

  Значение `'sync'` следует применять редко и очень аккуратно, потому что у него нет этих преимуществ.

  Дополнительную информацию об опции `flush` можно прочитать в разделе [синхронизации времени очистки эффектов](../guide/reactivity-computed-watchers.md#синхронизация-времени-очистки-эффектов).

- **См. также:** [Методы-наблюдатели](../guide/computed.md#методы-наблюдатели)

## $emit

- **Аргументы:**

  - `{string} eventName`
  - `...args (опционально)`

  Генерирует событие на текущем экземпляре. Любые дополнительные аргументы будут переданы в коллбэк функции прослушивания события.

- **Примеры:**

  Использование `$emit` только определяя имя события:

  ```html
  <div id="emit-example-simple">
    <welcome-button v-on:welcome="sayHi"></welcome-button>
  </div>
  ```

  ```js
  const app = createApp({
    methods: {
      sayHi() {
        console.log('Привет!')
      }
    }
  })

  app.component('welcome-button', {
    emits: ['welcome'],
    template: `
      <button v-on:click="$emit('welcome')">
        Кликни для приветствия
      </button>
    `
  })

  app.mount('#emit-example-simple')
  ```

  Использование `$emit` с передачей дополнительных аргументов:

  ```html
  <div id="emit-example-argument">
    <advice-component v-on:advice="showAdvice"></advice-component>
  </div>
  ```

  ```js
  const app = createApp({
    methods: {
      showAdvice(advice) {
        alert(advice)
      }
    }
  })

  app.component('advice-component', {
    emits: ['advise'],
    data() {
      return {
        adviceText: 'Какой-то совет'
      }
    },
    template: `
      <div>
        <input type="text" v-model="adviceText">
        <button v-on:click="$emit('advice', adviceText)">
          Кликни для получения какого-нибудь совета
        </button>
      </div>
    `
  })

  app.mount('#emit-example-argument')
  ```

- **См. также:**
  - [Опция `emits`](options-data.md#emits)
  - [Компоненты — Передача данных вместе с событием](../guide/component-basics.md#передача-данных-вместе-с-событием)

## $forceUpdate

- **Использование:**

  Вызывает принудительную перерисовку экземпляра компонента. Обратите внимание, что он не затрагивает все дочерние компоненты, а только сам экземпляр и дочерние компоненты с содержимым подставленным в слот.

## $nextTick

- **Аргументы:**

  - `{Function} callback (опционально)`

- **Использование:**

  Откладывает вызов коллбэка до следующего цикла обновления DOM. Используйте его сразу после изменения данных, чтобы дождаться обновления DOM. Аналогичен глобальному `nextTick`, за исключением того, что контекст `this` автоматически привязывается к экземпляру, вызвавшему этот метод.

- **Пример:**

  ```js
  createApp({
    // ...
    methods: {
      // ...
      example() {
        // изменяем данные
        this.message = 'changed'
        // DOM ещё не обновлён
        this.$nextTick(function() {
          // теперь DOM обновлён
          // `this` привязан к текущему экземпляру
          this.doSomethingElse()
        })
      }
    }
  })
  ```

- **См. также:** [nextTick](global-api.md#nexttick)
