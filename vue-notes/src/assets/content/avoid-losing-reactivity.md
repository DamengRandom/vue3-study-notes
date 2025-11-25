# Avoid Losing Reactivity

- Example

```html
<!-- Component which has title as a v-model bound based ref value -->
<script setup lang="ts">
  const title = ref('App Title')

  useSeoMeta({
    // title: title.value, // this is just a value, not reactive ...
    title, // this is reactive
  });
</script>
<template>
  <div>
    <AppTitle :title="title" />
    <input v-model="title" />
  </div>
</template>
```

```html
<!-- We will have another component which calling title -->
 <script setup lang="ts">
  // const { title } = defineProps({ // usually keep using props, up to you ..
  const props = defineProps({
    title: {
      type: String,
      default: 'App Title',
    },
  });

  // const titleRef = toRef(() => props.title); // between component, if we need to keep the reactivity of previous data, we need to use toRef to create a new ref value based on current props value 🥐🥐🥐 !!!!

  useSeoMeta({
    // title: titleRef, // this is reactive
    title: () => props.title, // creates a lazy getter; each call returns the current props.title !!!
    // simple getter function
  });
</script>
<template>
  <div>
    <!-- {{ titleRef }} -->
    {{ props.title }}  
  </div>
</template>
```

Summary

Keep variable reactive by:
- using `title: () => props.title` getter function
- using `toRef` to create a new ref value
