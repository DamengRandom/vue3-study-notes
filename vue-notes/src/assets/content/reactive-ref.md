# Ref & Reactive

In many docs, we know we use ref for primitive + reference type data and reactive only for reference type.

## When to use `ref`

When we need direct assign value, 

Assuming when we set an object value after 1 second, like:

```js
const reactiveNotUpdate = reactive({});

setTimeout(() => {
  reactiveNotUpdate = { a: 1, b: 2 };
}, 1000);

console.log(reactiveNotUpdate.value); // {}
// As you can see result still {} not { a: 1, b: 2 }. Why?
// Because after setTimeout process, reactiveNotUpdate turns into an object, but before assign { a: 1, b: 2 } to reactiveNotUpdate, reactiveNotUpdate was an proxy object, which was reactive !!!
```

In above case, better use `ref`.

(Always remember we need to use `ref.value = 'assign a value here'`)
always `.value` assign value, because .value reflects current ref object is proxy object, will be reactive !!!

When modify data case, we can use either `ref` or `reactive`.

In my view, better use ref, just got one more `.value` difference, thats it.

## `shallowRef` (some basics)

`shallowRef` is a ref that only tracks its own value, not the values of its properties.

Example:

```js
const shallowRefEg = shallowRef({ count: 0 });

shallowRefEg.value.count = 1; // this will NOT work, because shallowRef only tracks its own value, not the values of its properties.
// How to make it reactive? Using .value then,
shallowRefEg.value = { count: 2 };
```

## `customRef` (one example: debounce)

a Vue API which receives `trigger()` and `track()` as arguments, and return `get()` and `set()` methods

```js
import { customRef } from 'vue';

export function useDebounceRef(value, delay = 200) {
  let timeout;

  return customRef((trigger, track) => {
    return {
      get() {
        track();
        return value;
      },
      set(newValue) {
        clearTimeout(timeout);
        timeout = setTimeout(() => {
          value = newValue;
          trigger();
        }, delay);
      }
    }
  });
}
```

In Component level how to use it?

```html
<script setup>
  import { useDebounceRef } from '@utils/useDebounceRef';
  const debounceText = useDebounceRef('');
</script>
<template>
  <input v-model="debounceText" />
</template>
```

## `ref` vs `shallowRef`

- `ref`: create a `deeply reactive reference` that tracks `nested object values`

```ts
const deepRef = ref({
  user: {
    name: "John",
    profile: {
      age: 26
    }
  }
});

deepRef.value.user.profile.age = 24; // ✅ Triggers the reactivity ~

console.log(deepRef.value); // { user: { name: "John", profile: { age: 24 } } }
```

- `shallowRef`: create a `shallow reactive reference` that only tracks its own value, not the values of its properties (Another word: create a reactive reference that only tracks `direct value changes`)

```ts
const shallowRefEg = shallowRef({
  user: {
    name: "Doe",
    profile: {
      age: 36
    }
  }
});

shallowRefEg.value.user.profile.age = 33; // this will NOT work ❌, because shallowRef only tracks its own value, not the values of its properties.

// How to make it reactive? Using .value then,
shallowRefEg.value = { user: { name: "Doe", profile: { age: 33 } } }; // this will work ✅, because shallowRefEg.value is a new object, not the same object as before.
```

- `ref` could be slower then `shallowRef` since shallowRef only track the direct value changes, not nested object values ~
- `ref` is a bit `safer` choice and which is also predictable, better document because of `performance`, we choose to use `shallowRef` for certain cases ~

### When to use which ~

(1). When to sue `ShallowRef`?

```ts
// Large objects where you only care about replacement
const largeDataset = shallowRef(/* huge dataset */)

// Only triggers reactivity when replaced entirely
largeDataset.value = newLargeDataset
```

```ts
const immutableState = shallowRef(initialState)

const updateState = (newState) => {
  immutableState.value = { ...immutableState.value, ...newState }
  // Always replace, never mutate
}
```

(2). When to use `Ref`?

```ts
// Small objects where you need deep reactivity
const smallState = ref({
  count: 0,
  nested: {
    value: 10
  }
})

// Triggers reactivity for all levels ~
smallState.value.count = 1
smallState.value.nested.value = 20
```