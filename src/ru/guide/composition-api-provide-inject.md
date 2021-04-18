# Provide / Inject

> Подразумевается, что уже изучили и разобрались с разделами [Provide / Inject](component-provide-inject.md), [Введение в Composition API](composition-api-introduction.md) и [Основы реактивности](reactivity-fundamentals.md). Если нет — прочитайте их сначала.

Также можно использовать [provide / inject](component-provide-inject.md) и вместе с Composition API. Их можно вызывать только во время [`setup()`](composition-api-setup.md) с текущим активным экземпляром.

## Предыстория сценария использования

Предположим, что нужно переписать с использованием Composition API следующий код, где компонент `MyMap` предоставляет компоненту `MyMarker` текущее местоположение.

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

Для использования `provide` в `setup()` сначала потребуется явно импортировать метод из `vue`. Это позволяет определить каждое свойство своим собственным вызовом `provide`.

Функция `provide` позволяет определить свойство с помощью двух параметров:

1. Имя свойства (тип `<String>`)
2. Значение свойства

В компоненте `MyMap` внедряемые значения можно переписать следующим образом:

```vue{7,14-20}
<!-- src/components/MyMap.vue -->
<template>
  <MyMarker />
</template>

<script>
import { provide } from 'vue'
import MyMarker from './MyMarker.vue

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

Для использования `inject` в `setup()` также сначала потребуется явно импортировать метод из `vue`. После этого можно будет вызывать его, чтобы определить что требуется внедрить в компоненте.

Функция `inject` получает два параметра:

1. Имя внедряемого свойства
2. Значение по умолчанию (**Опционально**)

В компоненте `MyMarker` код можно переписать следующим образом:

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

Для добавления реактивности между предоставляемыми и внедряемыми значениями можно использовать [ref](reactivity-fundamentals.md#creating-standalone-reactive-values-as-refs) или [reactive](reactivity-fundamentals.md#declaring-reactive-state) при предоставлении значения.

Используя компонент `MyMap`, код можно обновить следующим образом:

```vue{7,15-22}
<!-- src/components/MyMap.vue -->
<template>
  <MyMarker />
</template>

<script>
import { provide, reactive, ref } from 'vue'
import MyMarker from './MyMarker.vue

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

Теперь, при изменении какого-либо из свойств, компонент `MyMarker` будет автоматически обновлён!

### Изменение реактивных свойств

При использовании реактивных значений для provide / inject **рекомендуется сохранять любые мутации реактивных свойств внутри _провайдера_ всегда когда это возможно**.

Например, в случае необходимости изменить местоположение пользователя, идеально будет сделать это внутри компонента `MyMap`.

```vue{28-32}
<!-- src/components/MyMap.vue -->
<template>
  <MyMarker />
</template>

<script>
import { provide, reactive, ref } from 'vue'
import MyMarker from './MyMarker.vue

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

Однако бывает когда требуется обновить данные внутри компонента, в который внедряются данные. В таком случае рекомендуется предоставлять метод, который ответственен за изменение реактивного свойства.

```vue{21-23,27}
<!-- src/components/MyMap.vue -->
<template>
  <MyMarker />
</template>

<script>
import { provide, reactive, ref } from 'vue'
import MyMarker from './MyMarker.vue

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

Наконец, рекомендуется использовать `readonly` на предоставляемых свойствах, если требуется гарантировать, что данные, передаваемые через `provide` не должны изменяться компонентом, в который внедряются.

```vue{7,25-26}
<!-- src/components/MyMap.vue -->
<template>
  <MyMarker />
</template>

<script>
import { provide, reactive, readonly, ref } from 'vue'
import MyMarker from './MyMarker.vue

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
