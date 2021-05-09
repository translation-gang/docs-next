# Введение

## Почему появился Composition API?

:::tip Примечание
Прежде чем приступать к изучению этого раздела документации необходимо понимать как базовые [основы Vue](introduction.md), так и принципы [создания компонентов](component-basics.md).
:::

<VideoLesson href="https://www.vuemastery.com/courses/vue-3-essentials/why-the-composition-api" title="Посмотрите бесплатное видео о Composition API на Vue Mastery">Посмотрите бесплатное видео о Composition API на Vue Mastery</VideoLesson>

Создание компонентов Vue позволяет извлекать повторяющиеся части интерфейса, вместе со связанной функциональностью, в переиспользуемые части кода. Этот подход добавляет приложению достаточно много с точки зрения удобства обслуживания и гибкости. Однако, коллективный опыт показал, что этого всё ещё может быть недостаточно, если приложение становится действительно большим — когда счёт идёт на несколько сотен компонентов. Когда приходится работать с такими большими приложениями — возможность разделения и переиспользования кода становится крайне важна.

Представим, что в приложении есть компонент, который отображает список репозиториев конкретного пользователя. Поверх него, нужно реализовать функциональность поиска и фильтрации. Компонент, управляющим подобным отображением, может выглядеть так:

```js
// src/components/UserRepositories.vue

export default {
  components: { RepositoriesFilters, RepositoriesSortBy, RepositoriesList },
  props: {
    user: {
      type: String,
      required: true
    }
  },
  data () {
    return {
      repositories: [], // 1
      filters: { ... }, // 3
      searchQuery: '' // 2
    }
  },
  computed: {
    filteredRepositories () { ... }, // 3
    repositoriesMatchingSearchQuery () { ... }, // 2
  },
  watch: {
    user: 'getUserRepositories' // 1
  },
  methods: {
    getUserRepositories () {
      // использование `this.user` для загрузки пользовательских репозиториев
    }, // 1
    updateFilters () { ... }, // 3
  },
  mounted () {
    this.getUserRepositories() // 1
  }
}
```

Этот компонент имеет несколько обязанностей:

1. Загрузка списка репозиториев из какого-либо внешнего API для этого имени пользователя и его обновление при изменении пользователя
2. Поиск репозиториев с помощью строки `searchQuery`
3. Фильтрация репозиториев с помощью объекта `filters`

Организация логики в опцииях компонента (`data`, `computed`, `methods`, `watch`) отлично работает в большинстве случаев. Однако, чем больше становятся компоненты, тем больше разрастётся список **логических блоков**. Что может привести к появлению компонентов, которые сложно изучать и понимать, особенно для тех, кто не занимался их разработкой.

![Options API во Vue: Код сгруппированный по типу опции](/images/options-api.png)

Пример большого компонента, в котором сгруппированы по цвету его **логические блоки**.

Подобная фрагментация усложняет понимание и поддержание таких сложных компонентов. Разделение по опциям делает менее заметными логические блоки используемые в них. Кроме того, при работе над одной логической задачей приходится постоянно «прыгать» между блоками в поисках соответствующего кода.

Было бы удобнее, если соответствующий логическому блоку код можно разместить рядом. И это именно то, чего позволяет добиться Composition API.

## Основы Composition API

Разобравшись с вопросом **почему появился**, можно перейти к вопросу **как использовать**. Прежде чем начинать работу с Composition API должно быть место, где его можно использовать. В компоненте Vue это место называется `setup`.

### Опция компонента `setup`

<VideoLesson href="https://www.vuemastery.com/courses/vue-3-essentials/setup-and-reactive-references" title="Посмотрите бесплатное видео об опции setup на Vue Mastery">Посмотрите бесплатное видео об опции setup на Vue Mastery</VideoLesson>

Новая опция компонента `setup` выполняется **перед созданием компонента**, сразу после разрешения входных параметров `props`, и служит точкой старта для Composition API.

:::warning ВНИМАНИЕ
Так как на этапе выполнения `setup` ещё не создан экземпляр компонента, то внутри опции `setup` не будет доступа к `this`. Это значит, что кроме входных параметров `props`, будет нельзя обратиться ни к каким свойствам, объявленным в компоненте, а именно — **локальному состоянию**, **вычисляемым свойствам** или **методам**.
:::

Опция `setup` должна быть функцией, которая принимает аргументами `props` и `context` (о которых подробнее поговорим [дальше](composition-api-setup.md#аргументы)). Кроме того, всё что возвращается из функции `setup` будет доступно для остальных частей компонента (вычисляемых свойств, методов, хуков жизненного цикла и т.д.), а также в шаблоне компонента.

Добавим `setup` в компонент:

```js
// src/components/UserRepositories.vue

export default {
  components: { RepositoriesFilters, RepositoriesSortBy, RepositoriesList },
  props: {
    user: {
      type: String,
      required: true
    }
  },
  setup(props) {
    console.log(props) // { user: '' }

    return {} // всё что возвращается, станет доступно в остальной части компонента
  }
  // ... остальная часть компонента
}
```

Извлечём логику первого логического блока (отмеченной как «1» в исходном примере).

> 1. Загрузка списка репозиториев из какого-либо внешнего API для этого имени пользователя и его обновление при изменении пользователя

Начнём с самых очевидных частей:

- Списка репозиториев
- Функции для обновления списка репозиториев
- Возвращение как списка репозиториев, так и функции обновления, чтобы использовать их в других опциях компонента

```js
// src/components/UserRepositories.vue; функция `setup`
import { fetchUserRepositories } from '@/api/repositories'

// в компоненте
setup (props) {
  let repositories = []
  const getUserRepositories = async () => {
    repositories = await fetchUserRepositories(props.user)
  }

  return {
    repositories,
    getUserRepositories // возвращаемые функции ведут себя также, как и методы
  }
}
```

Начало положено! Пока что ещё не всё работает, потому что переменная `repositories` не реактивна. С точки зрения пользователя, список репозиториев будет оставаться пустым. Давайте это исправим!

### Реактивные переменные с помощью `ref`

Во Vue 3 теперь можно сделать реактивную переменную где угодно с помощью новой функции `ref`, например так:

```js
import { ref } from 'vue'

const counter = ref(0)
```

`ref` принимает аргумент и возвращает его обёрнутым в объект со свойством `value`, которое затем можно использовать для чтения или изменения значения реактивной переменной:

```js
import { ref } from 'vue'

const counter = ref(0)

console.log(counter) // { value: 0 }
console.log(counter.value) // 0

counter.value++
console.log(counter.value) // 1
```

Оборачивание значения в объект может показаться лишним, но это нужно для одинакового поведения разных типов данных в JavaScript. Это связано с тем, что примитивные типы в JavaScript, такие как `Number` или `String`, передаются по значению, а не по ссылке:

![Различия передачи по ссылке и по значению](https://blog.penjee.com/wp-content/uploads/2015/02/pass-by-reference-vs-pass-by-value-animation.gif)

Наличие объекта-обёртки вокруг любого значения позволяет безопасно использовать его в любой части приложения, не беспокоясь о потере реактивности где-то по пути.

:::tip Примечание
Другими словами, `ref` создаёт **реактивную ссылку** к значению. Концепция работы со **ссылками** используется повсеместно в Composition API.
:::

Возвращаясь к примеру, создадим реактивную переменную `repositories`:

```js{3,7,9}
// src/components/UserRepositories.vue; функция `setup`
import { fetchUserRepositories } from '@/api/repositories'
import { ref } from 'vue'

// в компоненте
setup (props) {
  const repositories = ref([])
  const getUserRepositories = async () => {
    repositories.value = await fetchUserRepositories(props.user)
  }

  return {
    repositories,
    getUserRepositories
  }
}
```

Готово! Теперь каждый вызов `getUserRepositories` будет изменять `repositories` и обновлять вид блока, чтобы отобразить изменения. Компонент станет выглядеть так:

```js
// src/components/UserRepositories.vue
import { fetchUserRepositories } from '@/api/repositories'
import { ref } from 'vue'

export default {
  components: { RepositoriesFilters, RepositoriesSortBy, RepositoriesList },
  props: {
    user: {
      type: String,
      required: true
    }
  },
  setup (props) {
    const repositories = ref([])
    const getUserRepositories = async () => {
      repositories.value = await fetchUserRepositories(props.user)
    }

    return {
      repositories,
      getUserRepositories
    }
  },
  data () {
    return {
      filters: { ... }, // 3
      searchQuery: '' // 2
    }
  },
  computed: {
    filteredRepositories () { ... }, // 3
    repositoriesMatchingSearchQuery () { ... }, // 2
  },
  watch: {
    user: 'getUserRepositories' // 1
  },
  methods: {
    updateFilters () { ... }, // 3
  },
  mounted () {
    this.getUserRepositories() // 1
  }
}
```

Несколько частей первого логического блока теперь переместились в метод `setup` и удобно располагаются рядом друг с другом. Остался вызов `getUserRepositories` в хуке `mounted` и метод-наблюдатель, отслеживающий изменения входного параметра `user`.

Начнём с хука жизненного цикла.

### Использование хуков жизненного цикла внутри `setup`

Для паритета возможностей между Composition API и Options API, также требовался способ использовать хуки жизненного цикла внутри `setup`. Это стало возможно благодаря новым функциям, экспортируемым Vue. Хуки жизненного цикла в Composition API именуются как и в Options API, но с префиксом `on`: т.е. `mounted` станет `onMounted`.

Эти функции принимают аргументом коллбэк, который выполнится при вызове компонентом хука жизненного цикла.

Добавим его в функцию `setup`:

```js{3,12}
// src/components/UserRepositories.vue; функция `setup`
import { fetchUserRepositories } from '@/api/repositories'
import { ref, onMounted } from 'vue'

// в компоненте
setup (props) {
  const repositories = ref([])
  const getUserRepositories = async () => {
    repositories.value = await fetchUserRepositories(props.user)
  }

  onMounted(getUserRepositories) // хук `mounted` вызовет `getUserRepositories`

  return {
    repositories,
    getUserRepositories
  }
}
```

Осталось реализовать отслеживание изменений входного параметра `user` и реагирование на них. Для этого воспользуемся автономной функцией `watch`.

### Отслеживание изменений с помощью `watch`

Аналогично тому, как реализуем отслеживание изменений свойства `user` через опцию `watch`  внутри компонента — тоже самое можно сделать и через функцию `watch`, импортированную из Vue. Она принимает 3 аргумента:

- **Реактивная ссылка** или геттер-функция, которую требуется отслеживать
- Коллбэк
- Опциональные настройки

**Простой пример, чтобы понять как это работает:**

```js
import { ref, watch } from 'vue'

const counter = ref(0)
watch(counter, (newValue, oldValue) => {
  console.log('Новое значение counter: ' + counter.value)
})
```

При каждом изменении `counter`, например после `counter.value = 5`, будет срабатывать наблюдатель и вызовется коллбэк (второй аргумент), который в данном случае выведет в консоль `'Новое значение counter: 5'`.

**Эквивалент при использовании Options API:**

```js
export default {
  data() {
    return {
      counter: 0
    }
  },
  watch: {
    counter(newValue, oldValue) {
      console.log('Новое значение counter: ' + this.counter)
    }
  }
}
```

Более подробную информацию о `watch` можно найти в [продвинутом руководстве](reactivity-computed-watchers.md#watch).

**Возвращаемся к примеру:**

```js{7-8,12-13,18-19}
// src/components/UserRepositories.vue; функция `setup`
import { fetchUserRepositories } from '@/api/repositories'
import { ref, onMounted, watch, toRefs } from 'vue'

// в компоненте
setup (props) {
  // `toRefs` создаёт реактивную ссылку для входного параметра `user`
  const { user } = toRefs(props)

  const repositories = ref([])
  const getUserRepositories = async () => {
    // меняем `props.user` на `user.value` для получения значения по реактивной ссылке
    repositories.value = await fetchUserRepositories(user.value)
  }

  onMounted(getUserRepositories)

  // устанавливаем наблюдатель на реактивную ссылку входного параметра `user`
  watch(user, getUserRepositories)

  return {
    repositories,
    getUserRepositories
  }
}
```

Использование `toRefs` в самом начале `setup` необходимо для того, чтобы убедиться, что метод-наблюдатель будет реагировать на изменения входного параметра `user`.

Благодаря всем этим изменениям получилось вынести весь первый логический блок в одно место. Теперь можно сделать тоже самое и со вторым логическим блоком — фильтрацией по свойству `searchQuery`, но на этот раз воспользовавшись вычисляемым свойством.

### Автономные `computed`-свойства

Вне компонента Vue, как `ref` или `watch`, также можно создавать вычисляемые свойства с помощью функции `computed`, импортированной из Vue. Вернёмся к примеру со счётчиком:

```js
import { ref, computed } from 'vue'

const counter = ref(0)
const twiceTheCounter = computed(() => counter.value * 2)

counter.value++
console.log(counter.value) // 1
console.log(twiceTheCounter.value) // 2
```

Функция `computed` вернёт **реактивную ссылку** _только для чтения_ с результатом коллбэка, который был передан первым аргументом в `computed`. Для доступа к **значению** такого вычисляемого свойства, нужно обращаться к свойству `.value`, как и в случае с `ref`.

Перенесём функциональность поиска в `setup`:

```js{21-26,31-32}
// src/components/UserRepositories.vue; функция `setup`
import { fetchUserRepositories } from '@/api/repositories'
import { ref, onMounted, watch, toRefs, computed } from 'vue'

// в компоненте
setup (props) {
  // `toRefs` создаёт реактивную ссылку для входного параметра `user`
  const { user } = toRefs(props)

  const repositories = ref([])
  const getUserRepositories = async () => {
    // меняем `props.user` на `user.value` для получения значения по реактивной ссылке
    repositories.value = await fetchUserRepositories(user.value)
  }

  onMounted(getUserRepositories)

  // устанавливаем наблюдатель на реактивную ссылку входного параметра `user`
  watch(user, getUserRepositories)

  const searchQuery = ref('')
  const repositoriesMatchingSearchQuery = computed(() => {
    return repositories.value.filter(
      repository => repository.name.includes(searchQuery.value)
    )
  })

  return {
    repositories,
    getUserRepositories,
    searchQuery,
    repositoriesMatchingSearchQuery
  }
}
```

Можно продолжать делать аналогичное и для других **логических блоков**, но возможно уже раздаётся вопрос — _Разве это не перемещение кода в опцию `setup`, что теперь сделает её гигантской?_ Что ж, это не так. Поэтому, прежде чем продолжить, вынесем приведённый код выше в автономную **функцию композиции**. Начнём с создания `useUserRepositories.js`:

```js
// src/composables/useUserRepositories.js

import { fetchUserRepositories } from '@/api/repositories'
import { ref, onMounted, watch } from 'vue'

export default function useUserRepositories(user) {
  const repositories = ref([])
  const getUserRepositories = async () => {
    repositories.value = await fetchUserRepositories(user.value)
  }

  onMounted(getUserRepositories)
  watch(user, getUserRepositories)

  return {
    repositories,
    getUserRepositories
  }
}
```

Функциональность поиска вынесем в собственную функцию композиции:

```js
// src/composables/useRepositoryNameSearch.js

import { ref, computed } from 'vue'

export default function useRepositoryNameSearch(repositories) {
  const searchQuery = ref('')
  const repositoriesMatchingSearchQuery = computed(() => {
    return repositories.value.filter(repository => {
      return repository.name.includes(searchQuery.value)
    })
  })

  return {
    searchQuery,
    repositoriesMatchingSearchQuery
  }
}
```

**Теперь, когда эти две функциональности разнесены по отдельным файлам, можно начать использовать их в компоненте вот таким образом:**

```js{2,3,17,19-22}
// src/components/UserRepositories.vue
import useUserRepositories from '@/composables/useUserRepositories'
import useRepositoryNameSearch from '@/composables/useRepositoryNameSearch'
import { toRefs } from 'vue'

export default {
  components: { RepositoriesFilters, RepositoriesSortBy, RepositoriesList },
  props: {
    user: {
      type: String,
      required: true
    }
  },
  setup (props) {
    const { user } = toRefs(props)

    const { repositories, getUserRepositories } = useUserRepositories(user)

    const {
      searchQuery,
      repositoriesMatchingSearchQuery
    } = useRepositoryNameSearch(repositories)

    return {
      // Поскольку репозитории без фильтрации не используются, можно
      // объявить отфильтрованные результаты под именем `repositories`
      repositories: repositoriesMatchingSearchQuery,
      getUserRepositories,
      searchQuery,
    }
  },
  data () {
    return {
      filters: { ... }, // 3
    }
  },
  computed: {
    filteredRepositories () { ... }, // 3
  },
  methods: {
    updateFilters () { ... }, // 3
  }
}
```

Уже должно быть понятно, что и как делается, поэтому перейдём к концу и переместим оставшуюся функциональность фильтрации. Вдаваться в детали её реализации не будем, это не важно для данного руководства.

```js
// src/components/UserRepositories.vue
import { toRefs } from 'vue'
import useUserRepositories from '@/composables/useUserRepositories'
import useRepositoryNameSearch from '@/composables/useRepositoryNameSearch'
import useRepositoryFilters from '@/composables/useRepositoryFilters'

export default {
  components: { RepositoriesFilters, RepositoriesSortBy, RepositoriesList },
  props: {
    user: {
      type: String,
      required: true
    }
  },
  setup(props) {
    const { user } = toRefs(props)

    const { repositories, getUserRepositories } = useUserRepositories(user)

    const {
      searchQuery,
      repositoriesMatchingSearchQuery
    } = useRepositoryNameSearch(repositories)

    const {
      filters,
      updateFilters,
      filteredRepositories
    } = useRepositoryFilters(repositoriesMatchingSearchQuery)

    return {
      // Поскольку репозитории без фильтрации не используются, можно
      // объявить отфильтрованные результаты под именем `repositories`
      repositories: filteredRepositories,
      getUserRepositories,
      searchQuery,
      filters,
      updateFilters
    }
  }
}
```

Вот и всё!

И это лишь видимая часть айсберга Composition API и того, что он позволяет сделать. Более подробную информацию можно узнать в продвинутом руководстве.
