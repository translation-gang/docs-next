---
title: Корневой элемент transition-group больше не создаётся
badges:
  - breaking
---

# Корневой элемент `<transition-group>` больше не создаётся <MigrationBadges :badges="$frontmatter.badges" />

## Обзор

Теперь `<transition-group>` по умолчанию больше не создаёт корневой элемент при отрисовке, но может его создавать при указании атрибута `tag`.

## Синтаксис в 2.x

Во Vue 2, `<transition-group>` как и другие пользовательские компоненты, нуждался в корневом элементе, который по умолчанию был `<span>`, но настраивался через атрибут `tag`.

```html
<transition-group tag="ul">
  <li v-for="item in items" :key="item">
    {{ item }}
  </li>
</transition-group>
```

## Синтаксис в 3.x

Во Vue 3 теперь появилась [поддержка фрагментов](fragments.md), поэтому корневой тег больше _не нужен_ компонентам. Поэтому по умолчанию `<transition-group>` и не отрисовывает его.

- Если уже указан атрибут `tag` в коде Vue 2, как в примере выше, то всё будет работать как и раньше
- Если атрибут не указывался _и_ стилизация или другое поведение полагается на наличие корневого элемента `<span>`, то добавьте `tag="span"` в `<transition-group>`:

```html
<transition-group tag="span">
  <!-- -->
</transition-group>
```

## Стратегия миграции

[Флаг сборки для миграции: `TRANSITION_GROUP_ROOT`](migration-build.md#конфигурация-совместимости)

## См. также

- [Переименованы некоторые классы `<transition>`](transition.md)
- [Использование `<transition>` корневым элементом](transition-as-root.md)
