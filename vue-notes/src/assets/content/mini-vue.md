<!-- /Users/damonwu/Works/2025/concepts/vue/vue-starter-app/src/assets/content/mini-vue.md -->
```html
<div id="app"></div>
<script>
  function h(type, props, children) {
    return { type, props, children };
  }

  function mount(vnode, container) {
    const el = document.createElement(vnode.type);
    if (vnode.props) {
      for (const key in vnode.props) {
        const value = vnode.props[key];
        if (key.startsWith('on')) {
          el.addEventListener(key.slice(2).toLowerCase(), value);
        } else {
          el.setAttribute(key, value);
        }
      }
    }
    if (vnode.children) {
      if (typeof vnode.children === 'string') {
        el.textContent = vnode.children;
      } else {
        vnode.children.forEach(child => mount(child, el));
      }
    }
    vnode.el = el;
    container.appendChild(el);
  }

  function patch(n1, n2) {
    if (n1.type === n2.type) {
      const el = (n2.el = n1.el);
      const oldProps = n1.props || {};
      const newProps = n2.props || {};

      for (const key in newProps) {
        const oldValue = oldProps[key];
        const newValue = newProps[key];
        if (newValue !== oldValue) el.setAttribute(key, newValue);
      }
      for (const key in oldProps) {
        if (!(key in oldProps)) el.removeAttribute(key);
      }

      const oldChildren = n1.children || [];
      const newChildren = n2.children || [];

      if (typeof newChildren === 'string') {
        if (typeof oldChildren === 'string') {
          if (newChildren !== oldChildren) el.textContent = newChildren;
        } else {
          el.textContent = newChildren;
        }
      } else {
        if (typeof oldChildren === 'string') {
          el.innerHTML = '';
          newChildren.forEach(child => mount(child, el));
        } else {
          const commonLength = Math.min(oldChildren.length, newChildren.length);
          for (let i = 0; i < commonLength; i++) {
            patch(oldChildren[i], newChildren[i]);
            if (newChildren.length > oldChildren.length) {
              newChildren.slice(oldChildren.length).forEach(child => mount(child, el));
            } else if (oldChildren.length > newChildren.length) {
              oldChildren.slice(newChildren.length).forEach(child => el.removeChild(child.el));
            }
          }
        }
      }
    } else {
      // replace logic can be added here if needed
    }
  }

  let activeEffect;
  class Dep {
    subscribers = new Set();
    depend() { if (activeEffect) this.subscribers.add(activeEffect); }
    notify() { this.subscribers.forEach(effect => effect()); }
  }

  function watchEffect(effect) {
    activeEffect = effect;
    effect();
    activeEffect = null;
  }

  const targetMap = new WeakMap();
  function getDep(target, key) {
    let depsMap = targetMap.get(target);
    if (!depsMap) { depsMap = new Map(); targetMap.set(target, depsMap); }
    let dep = depsMap.get(key);
    if (!dep) { dep = new Dep(); depsMap.set(key, dep); }
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
  };

  function reactive(raw) {
    return new Proxy(raw, reactiveHandlers);
  }

  const App = {
    data: reactive({ count: 0 }),
    render() {
      return h('div', { onClick: () => { this.data.count++; } }, String(this.data.count));
    }
  };

  function mountApp(component, container) {
    let isMounted = false;
    let prevDom;
    watchEffect(() => {
      if (!isMounted) {
        prevDom = component.render();
        mount(prevDom, container);
        isMounted = true;
      } else {
        const newVdom = component.render();
        patch(prevDom, newVdom);
        prevDom = newVdom;
      }
    });
  }

  mountApp(App, document.getElementById('app'));
</script>
```