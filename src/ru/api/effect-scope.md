# API области действия эффектов <Badge text="3.2+" />

:::info ИНФОРМАЦИЯ
Область действия эффектов — продвинутый API, предназначенный в первую очередь для авторов библиотек. Узнать подробнее о том, как использовать этот API, можно в соответствующем [RFC](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0041-reactivity-effect-scope.md).
:::

## `effectScope`

Создаёт объект области действия эффекта, который может захватывать другие реактивные эффекты (например, вычисляемые свойства и наблюдатели), созданные внутри него, чтобы иметь возможность уничтожить все эти эффекты вместе.

**Типизация:**

```ts
function effectScope(detached?: boolean): EffectScope

interface EffectScope {
  run<T>(fn: () => T): T | undefined // undefined если область действия неактивна
  stop(): void
}
```

**Пример:**

```js
const scope = effectScope()

scope.run(() => {
  const doubled = computed(() => counter.value * 2)

  watch(doubled, () => console.log(doubled.value))

  watchEffect(() => console.log('Счётчик: ', doubled.value))
})

// для уничтожения всех эффектов в области действия
scope.stop()
```

## `getCurrentScope`

Возвращает текущую активную [область действия эффекта](#effectscope), если таковая есть.

**Типизация:**

```ts
function getCurrentScope(): EffectScope | undefined
```

## `onScopeDispose`

Регистрация коллбэка для текущей активной [области действия эффекта](#effectscope). Коллбэк будет вызван, когда связанная с ним область действия эффекта будет остановлена.

Этот метод можно использовать как не связанную с компонентами замену `onUnmounted` в переиспользуемых функциях композиций, поскольку функция `setup()` каждого компонента Vue также вызывается в области действия эффекта.

**Типизация:**

```ts
function onScopeDispose(fn: () => void): void
```
