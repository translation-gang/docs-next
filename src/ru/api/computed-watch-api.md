# Вычисляемые свойства и методы-наблюдатели

> В этом разделе в примерах кода используется синтаксис [однофайловых компонентов](../guide/single-file-component.md)

## `computed`

Принимает геттер-функцию и возвращает иммутабельный реактивный [ref-объект](refs-api.md#ref) для возвращаемого значения из геттера.

```js
const count = ref(1)
const plusOne = computed(() => count.value + 1)

console.log(plusOne.value) // 2

plusOne.value++ // ошибка
```

Может быть объект с функциями `get` и `set` для создания ref-объекта для чтения/записи.

```js
const count = ref(1)
const plusOne = computed({
  get: () => count.value + 1,
  set: val => {
    count.value = val - 1
  }
})

plusOne.value = 1
console.log(count.value) // 0
```

**Типизация:**

```ts
// только для чтения
function computed<T>(getter: () => T): Readonly<Ref<Readonly<T>>>

// чтение/запись
function computed<T>(options: { get: () => T; set: (value: T) => void }): Ref<T>
```

## `watchEffect`

Немедленно запускает функцию, отслеживая её зависимости с помощью системы реактивности, а затем повторно вызывает лишь при изменении этих зависимостей.

```js
const count = ref(0)

watchEffect(() => console.log(count.value))
// -> выведет 0

setTimeout(() => {
  count.value++
  // -> выведет 1
}, 100)
```

**Типизация:**

```ts
function watchEffect(
  effect: (onInvalidate: InvalidateCbRegistrator) => void,
  options?: WatchEffectOptions
): StopHandle

interface WatchEffectOptions {
  flush?: 'pre' | 'post' | 'sync'
  onTrack?: (event: DebuggerEvent) => void
  onTrigger?: (event: DebuggerEvent) => void
}

interface DebuggerEvent {
  effect: ReactiveEffect
  target: any
  type: OperationTypes
  key: string | symbol | undefined
}

type InvalidateCbRegistrator = (invalidate: () => void) => void

type StopHandle = () => void
```

**См. также**: [Руководство по `watchEffect`](../guide/reactivity-computed-watchers.md#watcheffect)

## `watch`

API `watch` является точным эквивалентом [this.$watch](instance-methods.md#watch) Options API (и соответствующей опции [watch](options-data.md#watch)). Для `watch` требуется отслеживание конкретного источника данных и применяет побочные эффекты в отдельном коллбэке. По умолчанию она также является «ленивой» — коллбэк вызывается только тогда, когда наблюдаемый источник изменился.

- В сравнении с [watchEffect](#watcheffect), `watch` позволяет:

  - Выполнять побочный эффект лениво;
  - Конкретнее определять какое состояние должно вызывать повторный запуск метода-наблюдателя;
  - Предоставляет доступ к предыдущему и текущему значениям отслеживаемого состояния.

### Отслеживание единственного источника данных

Источником данных для наблюдателя может быть функция геттер, возвращающая значение, или непосредственно реактивная [ref-ссылка](refs-api.md#ref):

```js
// наблюдение за геттер-функцией
const state = reactive({ count: 0 })
watch(
  () => state.count,
  (count, prevCount) => {
    /* ... */
  }
)

// наблюдение за ref-ссылкой
const count = ref(0)
watch(count, (count, prevCount) => {
  /* ... */
})
```

### Отслеживание нескольких источников данных

Можно отслеживать несколько источников одновременно, используя массив при записи:

```js
watch([fooRef, barRef], ([foo, bar], [prevFoo, prevBar]) => {
  /* ... */
})
```

### Общее поведение с `watchEffect`

Общее поведение `watch` и [`watchEffect`](#watcheffect) будет в возможностях [остановки отслеживания](../guide/reactivity-computed-watchers.md#остановка-отслеживания), [аннулировании побочных эффектов](../guide/reactivity-computed-watchers.md#аннулирование-побочных-эффектов) (с передачей коллбэка `onInvalidate` третьим аргументом), [синхронизации времени очистки эффектов](../guide/reactivity-computed-watchers.md#синхронизация-времени-очистки-эффектов) и инструментов [отладки](../guide/reactivity-computed-watchers.md#отладка-наблюдателеи).


**Типизация:**

```ts
// отслеживание единственного источника данных
function watch<T>(
  source: WatcherSource<T>,
  callback: (
    value: T,
    oldValue: T,
    onInvalidate: InvalidateCbRegistrator
  ) => void,
  options?: WatchOptions
): StopHandle

// отслеживание нескольких источников данных
function watch<T extends WatcherSource<unknown>[]>(
  sources: T
  callback: (
    values: MapSources<T>,
    oldValues: MapSources<T>,
    onInvalidate: InvalidateCbRegistrator
  ) => void,
  options? : WatchOptions
): StopHandle

type WatcherSource<T> = Ref<T> | (() => T)

type MapSources<T> = {
  [K in keyof T]: T[K] extends WatcherSource<infer V> ? V : never
}

// обратитесь к типам `watchEffect` для общих опций
interface WatchOptions extends WatchEffectOptions {
  immediate?: boolean // по умолчанию: false
  deep?: boolean
}
```

**См. также**: [Руководство по `watch`](../guide/reactivity-computed-watchers.md#watch)
