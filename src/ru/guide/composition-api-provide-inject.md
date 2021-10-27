# Provide / Inject

> В разделе в примерах кода используется синтаксис [однофайловых компонентов](single-file-component.md)

> Подразумевается, что уже изучили и разобрались с разделами [Provide / Inject](component-provide-inject.md), [Введение в Composition API](composition-api-introduction.md) и [Основы реактивности](reactivity-fundamentals.md). Если нет — прочитайте их сначала.

Можно использовать [provide / inject](component-provide-inject.md) и вместе с Composition API. Они могут быть вызваны во время [`setup()`](composition-api-setup.md) с текущим активным экземпляром.

## Предыстория сценария использования

Предположим, что нужно переписать следующий код с использованием Composition API, где компонент `MyMap` предоставляет текущее местоположение компоненту `MyMarker`.

```vue
<!-- src/components/MyMap.vue -->
<template>
  <MyMarker />
</template>

<script>
import MyMarker from './MyMarker.vue'

export default {
  components: {
    MyMarker
  },
  provide: {
    location: 'Северный полюс',
    geolocation: {
      longitude: 90,
      latitude: 135
    }
  }
}
</script>
```

```vue
<!-- src/components/MyMarker.vue -->
<script>
export default {
  inject: ['location', 'geolocation']
}
</script>
```

## Использование Provide

Для использования `provide` в `setup()` потребуется сначала явно импортировать метод из `vue`. Это позволяет определить каждое свойство своим собственным вызовом `provide`.

Функция `provide` позволяет определить свойство с помощью двух аргументов:

1. Имя свойства (тип `<String>`)
2. Значение свойства

Внедряемые значения в компоненте `MyMap` можно переписать следующим образом:

```vue{7,14-20}
<!-- src/components/MyMap.vue -->
<template>
  <MyMarker />
</template>

<script>
import { provide } from 'vue'
import MyMarker from './MyMarker.vue'

export default {
  components: {
    MyMarker
  },
  setup() {
    provide('location', 'Северный полюс')
    provide('geolocation', {
      longitude: 90,
      latitude: 135
    })
  }
}
</script>
```

## Использование Inject

Для использования `inject` в `setup()` также потребуется сначала явно импортировать метод из `vue`. После этого его можно будет вызывать, чтобы определить какие свойства требуется внедрить в компоненте.

Функция `inject` принимает два аргумента:

1. Имя внедряемого свойства
2. Значение по умолчанию **(опционально)**

Код в компоненте `MyMarker` можно переписать следующим образом:

```vue{3,6-14}
<!-- src/components/MyMarker.vue -->
<script>
import { inject } from 'vue'

export default {
  setup() {
    const userLocation = inject('location', 'Вселенная')
    const userGeolocation = inject('geolocation')

    return {
      userLocation,
      userGeolocation
    }
  }
}
</script>
```

## Реактивность

### Добавление реактивности

Чтобы сохранить реактивность между предоставляемыми и внедряемыми значениями можно воспользоваться [ref](reactivity-fundamentals.md#создание-автономных-ссылок-на-реактивные-значения) или [reactive](reactivity-fundamentals.md#объявление-реактивного-состояния) при предоставлении значения.

Код компонента `MyMap` можно обновить следующим образом:

```vue{7,15-22}
<!-- src/components/MyMap.vue -->
<template>
  <MyMarker />
</template>

<script>
import { provide, reactive, ref } from 'vue'
import MyMarker from './MyMarker.vue'

export default {
  components: {
    MyMarker
  },
  setup() {
    const location = ref('Северный полюс')
    const geolocation = reactive({
      longitude: 90,
      latitude: 135
    })

    provide('location', location)
    provide('geolocation', geolocation)
  }
}
</script>
```

Теперь компонент `MyMarker` будет обновляться автоматически при изменении какого-либо из свойств!

### Изменение реактивных свойств

При использовании реактивных значений в provide / inject **рекомендуется сохранять любые мутации реактивных свойств внутри _провайдера_ во всех случаях, когда это возможно**.

Например, если потребуется изменять местоположение пользователя, то лучшим способом будет это делать внутри компонента `MyMap`.

```vue{28-32}
<!-- src/components/MyMap.vue -->
<template>
  <MyMarker />
</template>

<script>
import { provide, reactive, ref } from 'vue'
import MyMarker from './MyMarker.vue'

export default {
  components: {
    MyMarker
  },
  setup() {
    const location = ref('Северный полюс')
    const geolocation = reactive({
      longitude: 90,
      latitude: 135
    })

    provide('location', location)
    provide('geolocation', geolocation)

    return {
      location
    }
  },
  methods: {
    updateLocation() {
      this.location = 'Южный полюс'
    }
  }
}
</script>
```

Но иногда требуется обновлять данные внутри компонента, в который внедряются данные. В таком случае лучше предоставлять метод, который будет изменять реактивное свойство.

```vue{21-23,27}
<!-- src/components/MyMap.vue -->
<template>
  <MyMarker />
</template>

<script>
import { provide, reactive, ref } from 'vue'
import MyMarker from './MyMarker.vue'

export default {
  components: {
    MyMarker
  },
  setup() {
    const location = ref('Северный полюс')
    const geolocation = reactive({
      longitude: 90,
      latitude: 135
    })

    const updateLocation = () => {
      location.value = 'Южный полюс'
    }

    provide('location', location)
    provide('geolocation', geolocation)
    provide('updateLocation', updateLocation)
  }
}
</script>
```

```vue{9,14}
<!-- src/components/MyMarker.vue -->
<script>
import { inject } from 'vue'

export default {
  setup() {
    const userLocation = inject('location', 'Вселенная')
    const userGeolocation = inject('geolocation')
    const updateUserLocation = inject('updateLocation')

    return {
      userLocation,
      userGeolocation,
      updateUserLocation
    }
  }
}
</script>
```

Также рекомендуется использовать `readonly` на предоставляемых свойствах, если нужно гарантировать, чтобы передаваемые через `provide` данные не изменялись компонентом, в который внедряются.

```vue{7,25-26}
<!-- src/components/MyMap.vue -->
<template>
  <MyMarker />
</template>

<script>
import { provide, reactive, readonly, ref } from 'vue'
import MyMarker from './MyMarker.vue'

export default {
  components: {
    MyMarker
  },
  setup() {
    const location = ref('Северный полюс')
    const geolocation = reactive({
      longitude: 90,
      latitude: 135
    })

    const updateLocation = () => {
      location.value = 'Южный полюс'
    }

    provide('location', readonly(location))
    provide('geolocation', readonly(geolocation))
    provide('updateLocation', updateLocation)
  }
}
</script>
```
