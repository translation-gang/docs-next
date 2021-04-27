# Реактивные ref-ссылки

> Этот раздел использует синтаксис [однофайловых компонентов](../guide/single-file-component.md) в примерах кода

## `ref`

Получает внутреннее значение и возвращает реактивный и мутируемый ref-объект. В этом ref-объекте есть только одно свойство `.value`, которое указывает на внутреннее значение.

**Пример:**

```js
const count = ref(0)
console.log(count.value) // 0

count.value++
console.log(count.value) // 1
```

Если в объект присваивается значение реактивной ref-ссылки, то объект становится глубоко реактивным с помощью метода [reactive](basic-reactivity.md#reactive).

**Типы:**

```ts
interface Ref<T> {
  value: T
}

function ref<T>(value: T): Ref<T>
```

Иногда может потребоваться определить сложный тип для внутреннего значения ref-ссылки. Это реализуется лаконично, с помощью передачи дженерика аргументом при вызове `ref` для переопределения вывода типа по умолчанию:

```ts
const foo = ref<string | number>('foo') // тип foo: Ref<string | number>

foo.value = 123 // ok!
```

Если тип дженерика неизвестен, рекомендуется приводить `ref` к `Ref<T>`:

```ts
function useState<State extends string>(initial: State) {
  const state = ref(initial) as Ref<State> // state.value -> State extends string
  return state
}
```

## `unref`

Возвращает внутреннее значение, если аргумент является [`ref`](#ref), в противном случае возвращает сам аргумент. Данная функция всего лишь синтаксический сахар для `val = isRef(val) ? val.value : val`.

```ts
function useFoo(x: number | Ref<number>) {
  const unwrapped = unref(x) // значение unwrapped гарантированно будет числом
}
```

## `toRef`

Можно использоваться для создания [`ref`](#ref) для свойства на исходном реактивном объекте. После этого ref-ссылку можно передавать, сохраняя реактивную связь с исходным свойством.

```js
const state = reactive({
  foo: 1,
  bar: 2
})

const fooRef = toRef(state, 'foo')

fooRef.value++
console.log(state.foo) // 2

state.foo++
console.log(fooRef.value) // 3
```

`toRef` пригодится, когда требуется передать ref-ссылку из входного параметра в функцию композиции:

```js
export default {
  setup(props) {
    useSomeFeature(toRef(props, 'foo'))
  }
}
```

`toRef` возвращает ref-ссылку пригодную для использования, даже если свойство в источнике в настоящее время не существует. Это особенно полезно при работе с необязательными входными параметрами, которые не будут подхвачены [`toRefs`](#torefs).

## `toRefs`

Преобразует реактивный объект в обычный объект, в котором каждое свойство будет [`ref`](#ref), указывающей на соответствующее свойство исходного объекта.

```js
const state = reactive({
  foo: 1,
  bar: 2
})

const stateAsRefs = toRefs(state)
/*
Тип stateAsRefs:

{
  foo: Ref<number>,
  bar: Ref<number>
}
*/

// Реактивная ref-ссылка и оригинальное свойство "связаны"
state.foo++
console.log(stateAsRefs.foo.value) // 2

stateAsRefs.foo.value++
console.log(state.foo) // 3
```

`toRefs` полезен при возвращении реактивного объекта из функции композиции, чтобы использующий компонент мог использовать деструктуризацию/оператор разложения к возвращаемому объекту без потери реактивности:

```js
function useFeatureX() {
  const state = reactive({
    foo: 1,
    bar: 2
  })

  // логика, работающая с состоянием

  // при возвращении преобразуем к refs
  return toRefs(state)
}

export default {
  setup() {
    // теперь можно использовать деструктуризацию без потери реактивности
    const { foo, bar } = useFeatureX()

    return {
      foo,
      bar
    }
  }
}
```

`toRefs` будет генерировать ref-ссылки только для свойств, включённых в исходный объект. Для создания ref-ссылки для конкретного свойства следует использовать [`toRef`](#toref).

## `isRef`

Проверяет является ли значение ref-объектом.

## `customRef`

Создаёт пользовательскую ref-ссылку с возможностью явно контролировать отслеживание зависимостей и управлять вызовом обновлений. Ожидает фабричную функцию, которая получает в качестве аргументов функции `track` и `trigger`, и должна возвращать объект с `get` и `set`.

- Пример использования пользовательской ref-ссылки для реализации debounce вместе с `v-model`:

  ```html
  <input v-model="text" />
  ```

  ```js
  function useDebouncedRef(value, delay = 200) {
    let timeout
    return customRef((track, trigger) => {
      return {
        get() {
          track()
          return value
        },
        set(newValue) {
          clearTimeout(timeout)
          timeout = setTimeout(() => {
            value = newValue
            trigger()
          }, delay)
        }
      }
    })
  }

  export default {
    setup() {
      return {
        text: useDebouncedRef('hello')
      }
    }
  }
  ```

**Типы:**

```ts
function customRef<T>(factory: CustomRefFactory<T>): Ref<T>

type CustomRefFactory<T> = (
  track: () => void,
  trigger: () => void
) => {
  get: () => T
  set: (value: T) => void
}
```

## `shallowRef`

Создаёт ref-ссылку, которая отслеживает изменения своего `.value`, но не делает это значение реактивным.

```js
const foo = shallowRef({})
// изменение значения ref-ссылки реактивно
foo.value = {}
// но значение не преобразуется в реактивное.
isReactive(foo.value) // false
```

**См. также**: [Создание автономных ссылок на реактивные значения](../guide/reactivity-fundamentals.md#создание-автономных-ссылок-на-реактивные-значения)

## `triggerRef`

Выполняет любые эффекты, привязанные вручную к [`shallowRef`](#shallowref).

```js
const shallow = shallowRef({
  greet: 'Привет, мир'
})

// Выведет "Привет, мир" один раз при первом проходе
watchEffect(() => {
  console.log(shallow.value.greet)
})

// Это не вызовет эффект, потому что ref-ссылка неглубокая
shallow.value.greet = 'Привет, вселенная'

// Выведет "Привет, вселенная"
triggerRef(shallow)
```

**См. также:** [Вычисляемые свойства и методы-наблюдатели — watchEffect](computed-watch-api.md#watcheffect)
