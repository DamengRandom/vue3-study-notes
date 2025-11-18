# How reactivity work

Remember: we have two parts in the reactivity system:
0. We have effect() function to save the value we want it to be reactive
1. track() - add effect function to the dependency set
2. trigger() - call effect function in the dependency set

## Code example

```js
// Basic reactivity example
let price = 10;
let quantity = 1;
let total = 0;
let dep = new Set();

function track() { // add effect function
  dep.add(effect);
}

function effect() { // save latest value
  total = price * quantity;
}

function trigger() { // call effect function
  dep.forEach(effect => effect());
}

track(); // add latest effect()
effect(); // update total
console.log(total);
console.log(quantity);
quantity = 3; // update value now
trigger(); // call effect()
console.log(total); // reactive update ~
```


```js
// To handle the case we have multiple properties in one object which needs to be reactive
const depMap = new Map();

function track(key) {
  let dep = depMap.get(key);

  if (!dep) { // if no dep, lets create a dep !!
    depMap.set(key, (dep = new Set()));
  }

  dep.add(effect);
}

function trigger(key) {
  let dep = depMap.get(key);

  if (dep) {
    dep.forEach(effect => { effect() });
  }
}

let product = {
  price: 10,
  quantity: 1,
}

let total = 0;

let effect = () => {
  total = product.price * product.quantity;
}

track('quantity');
effect();

// Type:
total // 10
product.quantity = 4; // update quantity value // 4
trigger('quantity'); // trigger effect() based on the latest quantity value !! // undefined
total // See,m now the latest value will be 40
```

```js
// To handle the case we have multiple properties in one object which needs to be reactive
const targetMap = new WeakMap(); // Why? because we have multiple objects which needs to be reactive, and we need key to be an object

// WeakMap vsd Map: WeakMap can save the key as an object, Map can't !!
// WeakMap: key is weak, if there is no reference to the key, it will be garbage collected
// Map: key is strong, if there is no reference to the key, it will not be garbage collected

function track(target, key) {
  let depsMap = targetMap.get(target);

  if (!depsMap) { // if no depsMap, lets create a depsMap !!
    targetMap.set(target, (depsMap = new Map()));
  }

  let dep = depsMap.get(key);

  if (!dep) { // if no dep, lets create a dep !!
    depsMap.set(key, (dep = new Set()));
  }

  dep.add(effect); // where the effect comes from?
}

function trigger(target, key) {
  const depsMap = targetMap.get(target)

  if (!depsMap) {
    return ;
  }

  let dep = depsMap.get(key);

  if (dep) {
    dep.forEach(effect => { effect() });
  }
}

let product = {
  price: 10,
  quantity: 1,
}

let total = 0;

let effect = () => {
  total = product.price * product.quantity;
}

track(product, 'quantity');
effect();

// Type:
total // 10
product.quantity = 4; // update quantity value // 4
trigger(product, 'quantity'); // trigger effect() based on the latest quantity value !! // undefined
total // See,m now the latest value will be 40
```
