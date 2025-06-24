# Vue concepts (Version 3 focus)

## 1. How does Vue work?

### Concepts

Vue.js is a progressive JavaScript framework for building user interfaces that focuses on the view layer and uses a reactive data-binding system to efficiently update the DOM when application state changes.

### Modules

- Reactive Module: allow to create javascript `reactive objects` which can be `watched` for changes
- Compiler Module: convert `template` to `render function`

  - What is `render functions` in Vue?

    Render functions are the underlying mechanism that Vue uses to generate Virtual DOM (VDOM) tree.

    ```js
    // Using render function
    const MyComponent = {
      render() {
        return h('div', { class: 'container' }, [
          h('h1', 'Hello World'),
          h('p', 'This is a paragraph')
        ])
      }
    }

    // Equivalent template
    // <template>
    //   <div class="container">
    //     <h1>Hello World</h1>
    //     <p>This is a paragraph</p>
    //   </div>
    // </template>
    ```

- Renderer Module: responsible for rendering the virtual DOM tree to the actual DOM, which consists of 3 phases:

  - Render Phase: render function get called and which returns virtual dom node
  - Mount Phase: render function takes virtual dom node to create web page
  - Patch Phase: compare the specific node(s) change and update the virtual dom node and then update the webpage eventually

<!-- Will watch & understand the video before interview: https://www.bilibili.com/video/BV1d4411v7UX/?spm_id_from=333.788.recommend_more_video.4&vd_source=eb4aa6845c77100317af9f94c35c4ed1 -->

### How to update the dynamic node

(1). In Vue, normally VueJS will create `_openBlock()` & `_createBlock()` for tracking the `dynamic` content changes, when changes detected, eg:

`_openBlock()`: starts collecting dynamic children in the current rendering context

`_createBlock()`: creates a block that contains references to all dynamic descendants

This optimization will update this specific virtual DOM element instead of updating the whole virtual DOM tree.

(2). Patch Flags for Granular Updates

Vue 3 uses Patch Flags to mark what type of dynamic content each node contains:

```js
// Compiled output example
_createElementVNode("p", null, _toDisplayString(dynamic), 1 /* TEXT */)
```
Common patch flags:

- `1` (TEXT) - dynamic text content
- `2` (CLASS) - dynamic class
- `4` (STYLE) - dynamic style
- `8` (PROPS) - dynamic props

One of `TEXT` dynamic change examples: 
```html
<div>
  <div><p>static content</p></div>
  <div>
    <p>{{ dynamic }}</p> <!-- dynamic change element -->
  </div>
</div>
```

## 2. Render function -> `h` (hyperscript) function

### Basic form of `h` function

Alternative is using `template` (synatx sugar)

```js
import { h } from 'vue';
// h: for hyperscript, which is used for creating virtual DOM node elements !!
const App = {
  render() {
    return h('div', {
      id: 'hello'
    }, [
      h('span', 'world') // This is nested DOM elements, can be defined as array format since it could have multiple 
    ])
  }
}
// Final output in the real DOM tree will be:
// <div id="hello"><span>world</span></div>

// The virtual DOM tree would be (Virtual DOM structure):
const vDom = {
  type: 'div',
  props: {
    id: 'hello'
  },
  children: [
    {
      type: 'span',
      props: {},
      children: ['world']
    }
  ]
}
```

### v-for directive example

```js
import { h } from 'vue';
// v-for would be a list of data returned
// eg: v-for="item in items"
const App = {
  render() {
    return this.items.map(item => h('div', {
      id: 'hello'
    }, [
      h('span', item)
    ]))
  }
}
```

### `this.$slots` practical usage

<!-- `this.$slots` is used to render the slot content passed in the component. -->

```html
<!-- Assuming that we need to build a Stack component for rendering the slot content passed in. -->
<Stack size="4">
  <div>hi</div>
  <Stack size="2">
    <div>hey</div>
    <div>hey</div>
  </Stack>
</Satck>
<!-- In Plain HTML will be represent like this: -->
<div class="stack">
  <div class="mt-4">
    <div>hi</div>
  </div>
  <div class="mt-4">
    <div class="stack">
      <div class="mt-2">
        <div>hey</div>
        <div>hey</div>
      </div>
    </div>
  </div>
</div>
```
<!-- Now, we need to convert above version to Vue code by using `slots` concept ~ -->
```js
import { h } from 'vue';

const Stack = {
  render() {
    const slot = this.$slots.default ? this.$slots.default() : [];

    return h('div', { class: 'stack' },  slot.map(childSlot => h('div', { class: `mt-${this.$props.size}` }, [childSlot])));
  }
}
```
<!-- Now, lets try with a complete version in browser and see if it works properly ~ -->
```html
<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
<style>
  .stack {
    border: 1px solid #ddd;
    padding: 10px;
    margin: 5px;
    background-color: #f9f9f9;
  }
  .mt-4 {
    margin: 16px
  }
  .mt-2 {
    margin: 8px
  }
  .mt-1 {
    margin: 4px
  }
</style>

<div id="app"></div>
  
<script>
  const { createApp, h } = Vue;

  const Stack = {
    render() {
      const slot = this.$slots.default ? this.$slots.default() : [];

      return h('div', { class: 'stack' },  slot.map(childSlot => h('div', { class: `mt-${this.$props.size}` }, [childSlot])));
    }
  };

  const App = {
    components: {
      Stack
    },
    template: `
      <Stack size="4">
        <div class="mt-4">hi</div>
        <Stack size="2">
          <div class="mt-2">hey</div>
          <div class="mt-2">hey</div>
          <Stack size="1">
            <div class="mt-1">aloha</div>
          </Stack>
        </Stack>
      </Stack>
    `
  };

  createApp(App).mount('#app');
</script>
```

## 3. Mount & Patch function

### Concepts

`mount()` in Vue is a function which attachs vue application instance (`createApp()`) to a DOM element.

`patch()` is a function which is used for updating the virtual DOM tree.

(1). How `mount()` works? Steps are:

- using `createApp()` to create application instance
- using `mount()` to mount application instance to a DOM element
- Vue renders root component into the target element
- Now, the app becomes reactive and interactive

(2). Quick example of utilizing `mount()`:

```js
import { createApp } from 'vue';
import App from './App.vue';

const app = createApp(App);

app.mount('#app');
```


### How to write a mount & patch function (example)

```js
<style>
  .red {
    color: red;
  }
  .green {
    color: green;
  }
</style>

<div id="app"></div>

<script>
function h(type, props, children) {
  return {
    type,
    props,
    children
  }
}

function mount(vnode, container) {
  const el = document.createElement(vnode.type); // when call mount function, need to copy the real dom element to represent as virtual dom ~
  // props
  if (vnode.props) {
    for (const key in vnode.props) {
      const value = vnode.props[key];
      el.setAttribute(key, value);
    }
  }
  // children
  if (vnode.children) {
    if (typeof vnode.children === 'string') {
      el.textContent = vnode.children;
    } else {
      vnode.children.forEach(child => {
        mount(child, el);
      });
    }
  }

  vnode.el = el; // assign el into vnode

  container.appendChild(el);
}

const vdom = h('div', { class: 'red' }, [
  h('span', null, ['hello'])
]);

mount(vdom, document.getElementById('app'));


function patch(n1, n2) { // Now, move to the patch part (Update the virtual DOM) -> same logic: compare old and new dom, and then update new dom as desired state
// n1 -> old dom (first node), n2 -> new dom (second node)
  // type
  if (n1.type === n2.type) {
    // compare props
    const el = n2.el = n1.el;
    const oldProps = n1.props || {};
    const newProps = n2.props || {};

    for (const key in newProps) {
      const oldValue = oldProps[key];
      const newValue = newProps[key];
      console.log('whats el: ', n2);
      if (newValue !== oldValue) {
        el.setAttribute(key, newValue);
      }
    }

    for (const key in oldProps) {
      if (!(key in oldProps)) {
        el.removeAttribute(key);
      }
    }

    // compare children
    const oldChildren = n1.children || [];
    const newChildren = n2.children || [];

    if (typeof newChildren === 'string') {
      if (typeof oldChildren === 'string') {
        if (newChildren !== oldChildren) {
          el.textContent = newChildren;
        }
      } else {
        // oldChildren else:
        el.textContent = newChildren;
      }
    } else {
      // newChildren else:
      if (typeof oldChildren === 'string') {
        el.innerHTML = '';

        newChildren.forEach(child => {
          mount(child, el); // mount for the new updated HTML DOM element
        });
      } else {
        // handling the case both oldChildren and newChildren are array have same 
        const commonLength = Math.min(oldChildren.length, newChildren.length);
        // can be reused for compare 2 arrays
        for (let i = 0; i < commonLength; i++) {
          patch(oldChildren[i], newChildren[i]); // update virtual DOM for the children elements they both (oldChildren & newChildren) have

          if (newChildren.length > oldChildren.length) { // means we need to add some new children elements into the vdom
            newChildren.slice(oldChildren.length).forEach(child => { // adding new elements into the current vdom tree
              mount(child, el);
            });
          } else if (oldChildren.length > newChildren.length) { // remove the old elements from the vdom tree
            oldChildren.slice(newChildren.length).forEach(child => {
              el.removeChild(child.el);
            });
          }
        }
      }
    }
  } else {
    // more logics TBA (To be added) - replace logics
  }
}

const vdom2 = h('div', { class: 'green' }, [
  h('span', null, 'changed ~')
]);

patch(vdom, vdom2);
</script>
```


## 4.`reactivity` in Vue

### Concept

Reactivity: when a program is able to react to the changes (Think about using Excel, 2 cells plus together, and if one cell value changed, the result cell value also get updated ~)

In Vue, its using `JavaScript Proxy` to observe the changes of the object.

`ref()` is a common way for Vue to handle state value changes ~

```js
const data = {
  name: 'damon'
};

const observedData = new Proxy(data, { // instantiate the proxy instance for tracking the data object changes
  set(target, key, value) {
    document.querySelector("#name").innerText = value;
    target[key] = value;
    console.log('target: ', target);
  } // set the update when value get changed
});

observedData.name = "unknown"; // update the name value

console.log(observedData.name); // unknown
```

```html
<div id="name"></div>
```

### Building `reactivity` from scratch example

```html
<script>
  let activeEffect;

  class Dep {
    subscribers = new Set();

    depend() {
      if (activeEffect) {
        this.subscribers.add(activeEffect);
      }
    }

    notify() {
      this.subscribers.forEach(effect => { effect() });
    }
  }

  function watchEffect(effect) {
    activeEffect = effect;
    effect();
    activeEffect = null;
  }

  // Basic JS way to implmenet reactive method

  // function reactive(raw) {
  //   Object.keys(raw).forEach(key => {
  //     const dep = new Dep();
  //     let value = raw[key];

  //     Object.defineProperty(raw, key, {
  //       get() {
  //         dep.depend();
  //         return value;
  //       },
  //       set(newValue) {
  //         value = newValue;
  //         dep.notify();
  //       }
  //     });
  //   });

  //   return raw;
  // }

  const targetMap = new WeakMap();

  function getDep(target, key) {
    let depsMap = targetMap.get(target);

    if (!depsMap) {
      depsMap = new Map();
      targetMap.set(target, depsMap);
    }

    let dep = depsMap.get(key);

    if (!dep) {
      dep = new Dep();
      depsMap.set(key, dep);
    }

    return dep;
  }

  const reactiveHandlers = {
    get(target, key, receiver) {
      const dep = getDep(target, key);

      dep.depend();

      return Reflect.get(target, key, receiver);
    },
    set(target, key, value, receiver) {
      const dep = getDep(target, key);
      const result = Reflect.set(target, key, value, receiver);

      dep.notify();

      return result;
    }
  }

  function reactive(raw) {
    return new Proxy(raw, reactiveHandlers);
  }

  const state = reactive({
    count: 0,
  });

  watchEffect(() => {
    console.log(state.count);
  });

  state.count++;
</script>
```

### Vue `reactivity` based (`ref()`) example

```html
<template>
  <main>
    <h1 v-if="isShow">Welcome ~</h1>
    <h1 v-else>See ya ~</h1>
    <button @click="toggleTitle">{{ isShow.value ? 'Hide' : 'Show' }}</button>
  </main>
</template>

<style scoped>
</style>

<script setup>
import { ref } from 'vue';

let isShow = ref(false);

function toggleTitle() {
  isShow.value = !isShow.value;
}
</script>
```

## 5.  

### Concept