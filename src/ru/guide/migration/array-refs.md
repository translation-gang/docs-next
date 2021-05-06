---
title: Массив ref-ссылок при использовании с v-for
badges:
  - breaking
---

# {{ $frontmatter.title }} <MigrationBadges :badges="$frontmatter.badges" />

Во Vue 2, использование атрибута `ref` внутри `v-for` заполнит соответствующее свойство `$refs` массивом ссылок на элементы. Такое поведение становится двусмысленным и неэффективным при наличии вложенных `v-for`.

Во Vue 3, такое использование больше не будет автоматически создавать массив в `$refs`. Чтобы получить несколько ссылок из одной привязки, следует привязать `ref` к функции, которая обеспечит большую гибкость (это новая возможность):

```html
<div v-for="item in list" :ref="setItemRef"></div>
```

С использованием Options API:

```js
export default {
  data() {
    return {
      itemRefs: []
    }
  },
  methods: {
    setItemRef(el) {
      if (el) {
        this.itemRefs.push(el)
      }
    }
  },
  beforeUpdate() {
    this.itemRefs = []
  },
  updated() {
    console.log(this.itemRefs)
  }
}
```

С использованием Composition API:

```js
import { onBeforeUpdate, onUpdated } from 'vue'

export default {
  setup() {
    let itemRefs = []
    const setItemRef = el => {
      if (el) {
        itemRefs.push(el)
      }
    }
    onBeforeUpdate(() => {
      itemRefs = []
    })
    onUpdated(() => {
      console.log(itemRefs)
    })
    return {
      setItemRef
    }
  }
}
```

Обратите внимание:

- `itemRefs` не обязательно должно быть массивом: это может быть и объект, где ссылки задаются их ключами итерации.

- Это также позволяет сделать `itemRefs` реактивным и отслеживать в нём изменения при необходимости.
