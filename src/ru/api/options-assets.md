# Доступные ресурсы

## directives

- **Тип:** `Object`

- **Подробности:**

  Хэш директив, которые доступны в экземпляре компонента.

- **Использование:**

  ```js
  const app = createApp({})

  app.component('focused-input', {
    directives: {
      focus: {
        mounted(el) {
          el.focus()
        }
      }
    },
    template: `<input v-focus>`
  })
  ```

- **См. также:** [Пользовательские директивы](../guide/custom-directive.md)

## components

- **Тип:** `Object`

- **Подробности:**

  Хэш компонентов, которые доступны в экземпляре компонента.

- **Использование:**

  ```js
  const Foo = {
    template: `<div>Foo</div>`
  }

  const app = createApp({
    components: {
      Foo
    },
    template: `<Foo />`
  })
  ```

- **См. также:** [Компоненты](../guide/component-basics.md)
