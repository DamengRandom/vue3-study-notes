# Common VueJS mistakes to avoid

## Don not use `index` as `:key` when using `v-for`

```html
<!-- NEVER DO THIS PLEASE -->
<div v-for="item in items" :key="item.index">
  {{ item.name }}
</div>
<!-- KEY must be unique -->
```

## Not only for loop is using :key, as long as we need re-render happening, we use :key

```html
<template>
  <div class="flex h-[90vh] justify-center items-center">
    <Transition>
      <div class="text-9xl" :key="counter">
        <!-- :key must be used, otherwise, we can't get the transition effect !!! -->
        {{ counter }}
      </div>
    </Transition>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const counter = ref(0)

// small demo increment so you can see the transition in action
setInterval(() => {
  counter.value++
}, 1000)
</script>

<style scoped>
.v-enter-active,
.v-leave-active {
  transition: opacity 0.5s ease;
}

.v-leave-active {
  transition: opacity 0.5s ease;
  position: absolute;
}

.v-enter-from,
.v-leave-to {
  opacity: 0;
}
</style>
```

## Not use computed for localStorage which is not going to trigger value update

```js
// BAD: this is not going to work as expected
const cookieAccepted = computed(() => {
  return localStorage.getItem('cookieAccepted')
})
```

### The browser APIs which also not working (non-reactive value as dep), such as:

```html
localStorage
Date()
DOM element properties (eg: clientWidth, clientHeight and etc)
etc
```

### Whats the solution? Using `VueUse`

- `VueUse`: is a handy reactive wrapper around browser APIs
- My understanding: convert browser APIs in Vue way to handle reactive value

Example

```html
<script setup>
import { useLocalStorage } from '@vueuse/core' // VueUse example

const cookiesAccepted = useLocalStorage('cookiesAccepted', false)
</script>

<template>
  <div class="flex h-[90vh] justify-center items-center scale-150">
    <label>
      <input type="checkbox" v-model="cookiesAccepted" />
      {{ cookiesAccepted }}
    </label>
  </div>
```

## Do not replace reactive values with new values

```html
<script>
  let posts = reactive([
    'Post 1',
    'Post 2'
  ]);

  function loadMore() {
    // fetch more posts from server
    posts = newPosts; // NEVER replace reactive value with a new value directly !!
  }
</script>
```

### Cue: Using `ref` instead please (better using `ref` everywhere, make life easier ~)

```html
<script>
  let posts = ref([
    'Post 1',
    'Post 2'
  ]);

  function loadMore() {
    // fetch more posts from server
    posts.value = newPosts; // using ref.value to update reactive value
  }
</script>
```

(**One quick not: default watcher depth is different between `ref()` and `reactive()`)

## Do not mutate props value via `v-model`

```html
<!-- Please do not mutate props value via `v-model` -->
<script setup lang="ts">
  defineProps<{
    modelValue: string,
  }>();
</script>
<template>
  <input id="username" v-model="modelValue" />
</template>
```

### Cue: using `defineModel()` to handle props value mutation

```html
<script setup lang="ts">
  const modelValue = defineModel();
</script>
<template>
  <input id="username" v-model="modelValue" />
</template>
```

## Be careful to accidently update the parent state from child level, because passing props as an `object` format !!!

```html
<!-- This is parent component, which pass data as an object to child level, and child component will update the data.username & password values accidentally to trigger parent level component re-rendered -->
<script setup lang="ts">
import { ref } from 'vue'

const data = ref({
  username: '',
  password: '',
})
</script>

<template>
  <PageHeading>Mutating Props</PageHeading>
  <section class="mutating-props">
    <h1>Data in Parent</h1>
    <pre>{{ data }}</pre>
    <hr class="my-10 block" />
    <h1 class="mt-10">Child Component</h1>
    <FormComponent :data="data" @submit="$event => data = $event"></FormComponent>
  </section>
</template>

<style>
.mutating-props * {
  font-size: 2rem;
}

.mutating-props {
  width: 980px;
  margin: 0 auto;
}
</style>
```

```html
<!-- This is Child component, which passes props.data from parent component to child component -->
<script setup lang="ts">
type Props = {
  data: {
    username: string;
    password: string;
  };
};

const props = defineProps<Props>();
const emit = defineEmits<{
  submit: [payload: Props['data']];
}>();

// still not resolved if you convert to local data
// const _data = ref(props.data); // because data.username is object based value, which will not work.

// Another word to explain: when we update the local _data.username, the parent _data as an object will also get updated, this is why when we start to type username, above display field username also get updated, because fo9r reference type, its mutable, which will update the whole object instead of specific value (Remember: Passing by reference, which shares same memory address !!!!!!!!)

// So how to fix it?
const newPropsData = JSON.parse(JSON.stringify(props.data)); // deep clone object !!!
const _data = ref(newPropsData);
// Or using lodash deep clone instead
// import _ from 'lodash';
// const newPropsData = _.cloneDeep(props.data);

// Remember `v-model` get triggered for updating the data username & password values, which will also update parent level value updated if we are not using deep clone to create a new object ⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️ !!!
</script>

<template>
  <form @submit.prevent="$event => $emit('submit', _data)">
    <fieldset>
      <label for="username" class="block">Username</label>
      <input id="username" v-model="_data.username" />
    </fieldset>

    <fieldset>
      <label class="block" for="password">Password</label>
      <input id="password" type="password" v-model="_data.password" />
    </fieldset>

    <UButton size="xl" class="mt-5" type="submit">Login</UButton>
  </form>
</template>
```


## Do not use composable function outside the script `setup` tag or `setup()` function

```html
<script>
// Outside setup - BAD!
import { useUser } from '@/composables/useUser'

// This won't work as expected
const { user, login } = useUser()

export default {
  setup() {
    // ...
  }
}
</script>
```

```html
<script setup>
// Inside setup - GOOD!
import { useUser } from '@/composables/useUser'

const { user, login } = useUser()
</script>
```

## Please always use UI component libs for building your application, instead of building your own UI component libs from scratch 🩸🩸🩸 lessons

- At moment: PrimeVue or Reka-UI, quite famous after `npm trends` website search

## No Fat Pania store, keep it lean and tidy ~

- Do not put too much code logics inside Pania store, and keep it SRP (Single Responsibility Principle), shorter (eg: separate into multiple smaller functions, create some selectors and etc) ~

- When we write code, always re-check the code and ask ourselves a question: WHAT IS THE MEANING FOR ME TO WRITE THIS CODE? And find the answer for yourself, if answer is blurred, it means, this code we actually don't understand (Be mindful: Alway be prepared for questions from your colleagues about the code you write) !!!!
