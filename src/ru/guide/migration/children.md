---
badges:
  - removed
---

# $children <MigrationBadges :badges="$frontmatter.badges" />

## Обзор

`$children` instance property removed from Vue 3.0 and no longer supported.

## Синтаксис в 2.x

In 2.x, developers could access direct child components of the current instance with `this.$children`:

```html
<div ref="app" id="app">
  <img alt="Vue logo" src="./assets/logo.png" width="25%" />
  <my-button>Change logo</my-button>
</div>
```

```js
export default {
  name: "App",
  components: {
    MyButton,
  },
  mounted() {
    console.log(this.$children); // [VueComponent]
  },
};
</script>
```

## Изменения в 3.x

In 3.x, `$children` property is removed and no longer supported. Instead, if you need to access a child component instance, we recommend using [$refs](../component-template-refs.md#template-refs).
