# Введение

## Зачем Composition API?

:::tip Примечание
Прежде чем начинать изучение этого раздела документации необходимо знать как базовые [основы Vue](introduction.md), так и принципы [создания компонентов](component-basics.md).
:::

<VideoLesson href="https://www.vuemastery.com/courses/vue-3-essentials/why-the-composition-api" title="Посмотрите бесплатное видео о Composition API на Vue Mastery">Посмотрите бесплатное видео о Composition API на Vue Mastery</VideoLesson>

Создание компонентов Vue позволяет извлечь повторяющиеся части интерфейса вместе со связанной функциональностью в переиспользуемые части кода. Подобное уже добавляет приложению довольно много с точки зрения удобства обслуживания и гибкости. Однако, коллективный опыт показал, что этого всё ещё может быть недостаточно, если приложение становится действительно большим — представьте, когда счёт идёт на несколько сотен компонентов. Когда приходится иметь дело с такими большими приложениями — возможность разделения и переиспользования кода становится крайне важна.

Представим, что в приложении есть компонент с отображением списка репозиториев определённого пользователя. Поверх него, необходимо добавить возможность поиска и фильтрации. Компонент, управляющим подобным отображением, может выглядеть так:

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

Такой компонент имеет несколько обязанностей:

1. Загрузка репозиториев для этого имени пользователя из какого-либо внешнего API и обновление списка при изменениях пользователя
2. Поиск репозиториев с помощью строки `searchQuery`
3. Фильтрация репозиториев с помощью объекта `filters`

Организация логики через опции компонента (`data`, `computed`, `methods`, `watch`) будет прекрасно работать в большинстве случаев. Однако, когда компоненты становятся больше, список **логических блоков** также растёт. Это может привести к появлению компонентов, которые сложно изучать и понимать, особенно для тех, кто не занимался их написанием.

![Options API во Vue: Код сгруппированный по типу опции](/images/options-api.png)

Пример большого компонента, где его **логические блоки** сгруппированы по цвету.

Подобная фрагментация усложняет понимание и поддержание таких сложных компонентов. Разделение по опциям делает менее заметными логические блоки используемые в них. Кроме того, при работе над одним логическим блоком постоянно приходится «прыгать» между блоками в поисках соответствующего кода.

Было бы гораздо удобнее, если соответствующий логическому блоку код можно разместить рядом. И это именно то, что позволяет сделать Composition API.

## Основы Composition API

Ответив на вопрос **зачем**, теперь можно перейти к разделу **как**. Прежде чем начать работу с Composition API нужно место, где его можно использовать. В компоненте Vue это место называется `setup`.

### Опция компонента `setup`

<VideoLesson href="https://www.vuemastery.com/courses/vue-3-essentials/setup-and-reactive-references" title="Посмотрите бесплатное видео об опции setup на Vue Mastery">Посмотрите бесплатное видео об опции setup на Vue Mastery</VideoLesson>

Новая опция компонента `setup` выполняется **перед** созданием компонента, как только разрешатся входные параметры `props`, и случит точкой входа для Composition API.

:::warning ВНИМАНИЕ
Так как на этапе выполнения `setup` экземпляр компонента ещё не создан, то внутри опции `setup` доступа к `this` не будет. Это значит, что кроме входных параметров `props`, нельзя будет обратиться ни к каким свойствам, объявленным в компоненте, а именно — **локальному состоянию**, **вычисляемым свойствам** или **методам**.
:::

Опция `setup` должна быть функцией, которая принимает аргументами `props` и `context`, о которых подробнее поговорим [дальше](composition-api-setup.md#arguments). Кроме того, всё что возвращает `setup` станет доступным для остальных частей компонента (вычисляемые свойства, методы, хуки жизненного цикла и т.д.), а также в шаблоне компонента.

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

    return {} // всё что возвращается здесь, станет доступно остальной части компонента
  }
  // ... остальная часть компонента
}
```

Теперь извлечём логику первого логического блока (отмечена как «1» в исходном примере).

> 1. Загрузка репозиториев для этого имени пользователя из какого-либо внешнего API и обновление списка при изменениях пользователя

Начнём с самых очевидных частей:

- Списка репозиториев
- Функции для обновления списка репозиториев
- Возвращение как списка, так и функции, чтобы сделать их доступными для других опций компонента

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
    getUserRepositories // возвращаемые функции ведут себя как методы
  }
}
```

Это стартовая точка, за исключением того, что ещё не всё работает, потому что переменная `repositories` пока не реактивна. С точки зрения пользователя, список репозиториев будет оставаться пустым. Давайте это исправим!

### Реактивные переменные с помощью `ref`

Во Vue 3.0 теперь где угодно любую переменную можно сделать реактивной с помощью новой функции `ref`, например так:

```js
import { ref } from 'vue'

const counter = ref(0)
```

Аргумент `ref` возвращается обёрнутым в объект со свойством `value`, которое затем можно использовать для доступа или изменения значения реактивной переменной:

```js
import { ref } from 'vue'

const counter = ref(0)

console.log(counter) // { value: 0 }
console.log(counter.value) // 0

counter.value++
console.log(counter.value) // 1
```

Оборачивание значения в объект может показаться лишним, но это нужно для одинакового поведения для разных типов данных в JavaScript. Связано это с тем, что примитивные типы в JavaScript, такие как `Number` или `String`, передаются по значению, а не по ссылке:

![Отличия в передаче по ссылки и передаче по значению](https://blog.penjee.com/wp-content/uploads/2015/02/pass-by-reference-vs-pass-by-value-animation.gif)

Наличие вокруг любого значения объекта-обёртки позволяет безопасно использовать его во всём приложении, не беспокоясь о потере реактивности где-то по пути.

:::tip Примечание
Другими словами, `ref` создаёт **реактивную ссылку** к значению. Концепция работы со **ссылками** будет повсюду использоваться в Composition API.
:::

Возвращаясь к примеру, создадим реактивную переменную `repositories`:

```js
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

Готово! Теперь при каждом вызове `getUserRepositories` будет изменяться `repositories` и обновляться вид блока, чтобы отобразить изменения. Теперь компонент выглядит так:

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

Несколько кусочков первого логического блока теперь переместились в метод `setup` и удобно располагаются рядом друг с другом. Остался вызов `getUserRepositories` в хуке `mounted` и метод-наблюдатель, отслеживающий изменения входного параметра `user`.

Начнём с хука жизненного цикла.

### Использование хуков жизненного цикла внутри `setup`

Для паритета возможностей между Composition API и Options API, также требуется способ использования хуков жизненного цикла внутри `setup`. Это стало возможным благодаря новым функциям, экспортируемым Vue. Хуки жизненного цикла в Composition API имеют такое же имя, как и в Options API, но с префиксом `on`: т.е. `mounted` станет `onMounted`.

Эти функции принимают коллбэк, который выполнится при вызове хука жизненного цикла компонентом.

Добавим его в функцию `setup`:

```js{12}
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

Теперь осталось реагировать на изменения входного параметра `user`. Для этого потребуется использовать автономную функцию `watch`.

### Отслеживание изменений с помощью `watch`

Подобно тому, как добавляем внутри компонента отслеживание изменений свойства `user` через опцию `watch`, можно сделать тоже самое через функцию `watch`, импортированную из Vue. Она принимает 3 аргумента:

- **Реактивная ссылка** или геттер-функция, которую требуется отслеживать
- Коллбэк
- Опциональные настройки

**Простой пример как это работает:**

```js
import { ref, watch } from 'vue'

const counter = ref(0)
watch(counter, (newValue, oldValue) => {
  console.log('Новое значение counter: ' + counter.value)
})
```

Каждый раз, когда изменяется `counter`, например после `counter.value = 5`, наблюдатель сработает и вызовет коллбэк (второй аргумент), который в данном случае выведет в консоль `'Новое значение counter: 5'`.

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

Более подробную информацию о `watch` можно найти   в [продвинутом руководстве](reactivity-computed-watchers.md#watch).

**Возвращаемся к примеру:**

```js{7-8,12-13,18-19}
// src/components/UserRepositories.vue; функция `setup`
import { fetchUserRepositories } from '@/api/repositories'
import { ref, onMounted, watch, toRefs } from 'vue'

// в компоненте
setup (props) {
  // `toRefs` создаёт реактивную ссылку для свойству `user` входных параметров
  const { user } = toRefs(props)

  const repositories = ref([])
  const getUserRepositories = async () => {
    // меняем `props.user` на `user.value` для доступа к значению по ссылке
    repositories.value = await fetchUserRepositories(user.value)
  }

  onMounted(getUserRepositories)

  // установка наблюдателя на реактивную ссылку входного параметра user
  watch(user, getUserRepositories)

  return {
    repositories,
    getUserRepositories
  }
}
```

Использование `toRefs` в самом начале `setup` требуется для того, чтобы убедиться, что метод-наблюдатель будет реагировать на изменения входного параметра `user`.

Со всеми этими изменениями получилось вынести весь первый логический блок в одно место. Теперь можно сделать тоже самое и со вторым логическим блоком — фильтрацией по свойству `searchQuery`, на этот раз с использованием вычисляемого свойства.

### Автономные `computed`-свойства

Вычисляемые свойства, как `ref` или `watch`, также можно создавать вне компонента Vue с помощью функции `computed`, импортированной из Vue. Вернёмся к примеру со счётчиком:

```js
import { ref, computed } from 'vue'

const counter = ref(0)
const twiceTheCounter = computed(() => counter.value * 2)

counter.value++
console.log(counter.value) // 1
console.log(twiceTheCounter.value) // 2
```

Функция `computed` возвращает **реактивную ссылку** _только для чтения_ вывода результата коллбэка, переданного первым аргументом в `computed`. Для доступа к **значению** нового вычисляемого свойства, нужно обращаться к свойству `.value`, как и в случае с `ref`.

Перенесём функциональность поиска в `setup`:

```js{21-26,31-32}
// src/components/UserRepositories.vue; функция `setup`
import { fetchUserRepositories } from '@/api/repositories'
import { ref, onMounted, watch, toRefs, computed } from 'vue'

// в компоненте
setup (props) {
  // `toRefs` создаёт реактивную ссылку для свойству `user` входных параметров
  const { user } = toRefs(props)

  const repositories = ref([])
  const getUserRepositories = async () => {
    // меняем `props.user` на `user.value` для доступа к значению по ссылке
    repositories.value = await fetchUserRepositories(user.value)
  }

  onMounted(getUserRepositories)

  // установка наблюдателя на реактивную ссылку входного параметра user
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

Можно продолжать делать то же самое и для других **логических блоков**, но возможно уже раздаётся вопрос — _Разве это не просто перемещение кода в опцию `setup`, что сделает её гигантской?_ Что ж, это не так. Поэтому, прежде чем продолжить, вынесем приведённый код выше в автономную **функцию композиции**. Начнём с создания `useUserRepositories.js`:

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

Функциональность поиска вынесем в свою функцию композиции:

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

**Теперь, имея эти две функциональности в отдельных файлах, можно начать использовать их в компоненте. Вот как это можно сделать:**

```js
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

На данный момент уже должно быть понятно, как это делается, поэтому перейдём к концу и переместим оставшуюся функциональность фильтрации. Вдаваться в детали её реализации не будем, это не важно для данного руководства.

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

Учтите, это лишь видимая часть айсберга Composition API и того, что он позволяет сделать. Чтобы узнать больше, обратитесь к продвинутому руководству.
