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
