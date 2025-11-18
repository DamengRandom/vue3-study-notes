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
