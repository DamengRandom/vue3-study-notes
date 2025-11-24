# Common Good Practices when writing VueJS code

## 1. Always use `:key` when using `v-for`

```html
<ul>
  <li v-for="item in items" :key="item.id">
    {{ item.name }}
  </li>
</ul>
```


## 2. Using kebab-case for events

```html
<!-- Inside PopupWindow.vue -->
this.$emit("close-window");
<!-- Inside ParentComponent.vue -->
<template>
<popup-window @close-window="closeWindow()" />
</template>
```

## 3. Using `camelCase` for script and `kebab-case` for template

```html

<script>
export default {
  props: {
    titleText: String,
  },
}
</script>
<template>
  <popup-window title-text="hello world" /> 
  <!-- See: titleText becomes title-text in template -->
</template>
```

## 4. Do not use `v-if` inside `v-for`

```html
<!-- NEVER DO THIS PLEASE -->
<div v-for="product in products" v-if="product.price < 500"></div>
```

## 5. Use `PascalCase` or `kebab-case` for component names

```txt
# BAD

mycomponent.vue  
myComponent.vue  
Mycomponent.vue

# GOOD

MyComponent.vue
my-component.vue
```

## 6. Component only used ONCE should have prefix "The"

```txt
# BAD

PopupWindow.vue

# GOOD

ThePopupWindow.vue
```

## 7. Should be consistent to use vue directive shorthand symbols

```txt
: for v-bind
@ for v-on
# for v-slot
```

## 8. Template expression should only have basic JavaScript expressions

```html
<template>
  <!--BAD-->
  {{
    fullName
      .split(' ')
      .map(function (word) {
        return word[0].toUpperCase() + word.slice(1)
      })
      .join(' ')
  }}
  <!-- too long, not recommended -->
</template>
```

```html
<script>
export default {
  // The complex expression has been moved to a computed property
  computed: {
    normalizedFullName () {
      return this.fullName
        .split(' ')
        .map((word) => {
          return word[0].toUpperCase() + word.slice(1)
        })
        .join(' ')
    },
  },
}
</script>
<!-- Recommended !! -->
<template>{{ normalizedFullName }}</template>
```

## 9. Using `app.config.errorHandler` level for error handlers, eg: Sentry

```js
import { createApp } from 'vue'
import * as Sentry from '@sentry/vue'

const app = createApp(...)

app.config.errorHandler = (err, instance, info) => {
  // Example code: report error to tracking services
  Sentry.captureException(err, {
    extra: {
      componentName: instance.$options.name,
      info,
    },
  })
}
```

## 10. Using composables for encapsulating logic

```js
import { ref, onMounted } from 'vue'

export function useMousePosition() {
  const x = ref(0)
  const y = ref(0)

  function updatePosition(event) {
    x.value = event.clientX
    y.value = event.clientY
  }

  onMounted(() => {
    window.addEventListener('mousemove', updatePosition)
  })

  return {
    x,
    y,
  }
}
```

## 11. Using `v-memo` for optimizing performance

```html
<div v-for="item in list" :key="item.id" v-memo="[item.id === selected]">
  <p>ID: {{ item.id }} - selected: {{ item.id === selected }}</p>
  <p>...more child nodes</p>
</div>
```

## 12. Using `v-once` to skip future updates

```html
<!-- single element -->
<span v-once>This will never change: {{msg}}</span>
<!-- the element have children -->
<div v-once>
  <h1>Comment</h1>
  <p>{{msg}}</p>
</div>
<!-- component -->
<MyComponent v-once :comment="msg"></MyComponent>
<!-- `v-for` directive -->
<ul>
  <li v-for="i in list" v-once>{{i}}</li>
</ul>
```

## 13. Using virtual list foir large amount of list data rendering

Link [here](https://github.com/Akryum/vue-virtual-scroller/blob/master/packages/vue-virtual-scroller/README.md)

## 14. Never use Non-trusted templates

```html
<!-- HTML Injection -->
<div v-html="userProvidedHtml"></div>
<div innerHTML={this.userProvidedHtml}></div>
<!-- URL Injection -->
<a :href="userProvidedUrl">
  click me
</a>
<!-- Style Injection -->
<a
  :href="sanitizedUrl"
  :style="userProvidedStyles"
>
  click me
</a>
```

```js
// <!-- HTML Injection -->
h('div', {
  innerHTML: this.userProvidedHtml
})
```

## 15. Implement lazy loading with Suspense

```html
<template>
  <Suspense>
    <template #default>
      <UserList />
    </template>
    <template #fallback>
      <div class="loading">
        <p>Loading users...</p>
        <div class="spinner"></div>
      </div>
    </template>
  </Suspense>
</template>

<script setup>
import { defineAsyncComponent } from 'vue'

const UserList = defineAsyncComponent(() =>
  import('./components/UserList.vue')
)
</script>
```


## `useFetch` & `$fetch`

- `useFetch`: need to use useFetch in page level of vue components (When doing data fetching for that component)
(making data fetching for support Server Side Rendering (SSR))
- `$fetch`: When doing event triggering, eg: onSubmit, onClick, etc, we can use $fetch to do the data fetching ~
(Client side API call support)

## When to not use `.value`?

```html
<template>
  <!-- ❌ NO .value in template -->
  <div>{{ count }}</div>
  
  <!-- ❌ NO .value for methods -->
  <button @click="increment">Increment</button>
  
  <!-- ❌ NO .value for reactive objects -->
  <div>{{ user.name }}</div>
  
  <!-- ❌ NO .value in v-model -->
  <input v-model="username">
</template>
```

```html
<script setup>
const props = defineProps({
  userId: Number,
  userData: Object
})

// ❌ NO .value for props (they're already reactive)
console.log(props.userId)
console.log(props.userData.name)
</script>
```

Summary

| Context | Need .value? | Example |
| --- | --- | --- |
| `<script setup>` | ✅ Yes | `count.value++` |
| `<template>` | ❌ No | `{{ count }}` |
| `computed()` | ✅ Yes | `return count.value * 2` |
| `watch()` | ✅ Yes | `watch(count, (val) => ...)` |
| `Lifecycle hooks` | ✅ Yes | `onMounted(() => count.value)` |
| `Template refs` | ✅ Yes | `inputRef.value.focus()` |
| `Reactive props` | ❌ No | `props.userId` |
| `Reactive destructure` | ❌ No | `const { count } = state` |

## Check bundle files size ~

```bash
pnpm dlx vite-bundle-visualizer
```

## use `onMounted()` for calling function during page rendered stage

```html
<script setup lang="ts">
import { useUserStore } from '@/stores/userStore';
import { onMounted } from 'vue';

const userStore = useUserStore();

// This will only run once when the component mounts
onMounted(() => {
  userStore.fetchUser(); // similar like call once function, only run once
});
</script>
```
