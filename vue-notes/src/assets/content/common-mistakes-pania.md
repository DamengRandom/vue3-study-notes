# Avoid common mistakes when using Pania

- Some examples to show Pania store mistakes we might be able to overcome during code writing ~

## Using composition API styled for writing Pania store code logics ~

```ts
// This is Non-composition API styled pania store
import { defineStore } from 'pinia'

export const useUserStore = defineStore('user', {
  state: () => ({
    userId: null as string | null,
    name: '',
    email: '',
    isLoggedIn: false,
    preferences: {
      theme: 'light' as 'light' | 'dark',
      notifications: true,
    },
  }),

  getters: {
    displayName: (state) => state.name || 'Guest User',
    hasCompleteProfile: (state) => Boolean(state.name) && Boolean(state.email),
    userPreferences: (state) => state.preferences,
    isDarkTheme: (state) => state.preferences.theme === 'dark',
  },

  actions: {
    login(payload: { userId: string; name: string; email: string }) {
      this.userId = payload.userId
      this.name = payload.name
      this.email = payload.email
      this.isLoggedIn = true
    },
    logout() {
      this.userId = null
      this.name = ''
      this.email = ''
      this.isLoggedIn = false
    },
    updateProfile(payload: { name?: string; email?: string }) {
      if (payload.name !== undefined) this.name = payload.name
      if (payload.email !== undefined) this.email = payload.email
    },
    setTheme(theme: 'light' | 'dark') {
      this.preferences.theme = theme
    },
    toggleTheme() {
      this.preferences.theme = this.preferences.theme === 'light' ? 'dark' : 'light'
    },
    setNotifications(enabled: boolean) {
      this.preferences.notifications = enabled
    },
    setPreferences(payload: Partial<{ theme: 'light' | 'dark'; notifications: boolean }>) {
      if (payload.theme !== undefined) this.preferences.theme = payload.theme
      if (payload.notifications !== undefined) this.preferences.notifications = payload.notifications
    },
    saveToStorage() {
      const data = {
        userId: this.userId,
        name: this.name,
        email: this.email,
        isLoggedIn: this.isLoggedIn,
        preferences: this.preferences,
      }
      localStorage.setItem('user-store', JSON.stringify(data))
    },
    loadFromStorage() {
      const raw = localStorage.getItem('user-store')
      if (!raw) return
      const data = JSON.parse(raw)
      this.userId = data.userId ?? null
      this.name = data.name ?? ''
      this.email = data.email ?? ''
      this.isLoggedIn = Boolean(data.isLoggedIn)
      if (data.preferences) {
        this.preferences.theme = data.preferences.theme === 'dark' ? 'dark' : 'light'
        this.preferences.notifications = Boolean(data.preferences.notifications)
      }
    },
    reset() {
      this.$reset()
    },
  },
})
```

```ts
// Composition API styled Pania store - Recommended to use this way ✅✅✅ !!!
import { defineStore } from 'pinia'

export const useUserStore = defineStore('user', () => {
  // convert to ref values
  const userId = ref<string | null>(null)
  const name = ref('')
  const email = ref('')
  const isLoggedIn = ref(false)
  const preferences = ref({
    theme: 'light' as 'light' | 'dark',
    notifications: true,
  });

  // convert getters to computed properties
  const displayName = computed(() => name.value || 'Guest User')
  const hasCompleteProfile = computed(() => Boolean(name.value) && Boolean(email.value))
  const userPreferences = computed(() => preferences.value)
  const isDarkTheme = computed(() => preferences.value.theme === 'dark')
  
  // remove actions wrapper
  function login(payload: { userId: string; name: string; email: string }) {
    userId.value = payload.userId
    name.value = payload.name
    email.value = payload.email
    isLoggedIn.value = true
  }

  function logout() {
    userId.value = null
    name.value = ''
    email.value = ''
    isLoggedIn.value = false
  }

  function updateProfile(payload: { name?: string; email?: string }) {
    if (payload.name !== undefined) name.value = payload.name
    if (payload.email !== undefined) email.value = payload.email
  }

  function setTheme(theme: 'light' | 'dark') {
    preferences.value.theme = theme
  }

  function toggleTheme() {
    preferences.value.theme = preferences.value.theme === 'light' ? 'dark' : 'light'
  }

  function setNotifications(enabled: boolean) {
    preferences.value.notifications = enabled
  }

  function setPreferences(payload: Partial<{ theme: 'light' | 'dark'; notifications: boolean }>) {
    if (payload.theme !== undefined) preferences.value.theme = payload.theme
    if (payload.notifications !== undefined) preferences.value.notifications = payload.notifications
  }

  function saveToStorage() {
    const data = {
      userId: userId.value,
      name: name.value,
      email: email.value,
      isLoggedIn: isLoggedIn.value,
      preferences: preferences.value,
    }
    localStorage.setItem('user-store', JSON.stringify(data))
  }

  function loadFromStorage() {
    const raw = localStorage.getItem('user-store')
    if (!raw) return
    const data = JSON.parse(raw)
    userId.value = data.userId ?? null
    name.value = data.name ?? ''
    email.value = data.email ?? ''
    isLoggedIn.value = Boolean(data.isLoggedIn)
    if (data.preferences) {
      preferences.value.theme = data.preferences.theme === 'dark' ? 'dark' : 'light'
      preferences.value.notifications = Boolean(data.preferences.notifications)
    }
  }

  function reset() {
    userId.value = null
    name.value = ''
    email.value = ''
    isLoggedIn.value = false
    preferences.value = {
      theme: 'light',
      notifications: true,
    }
  }

  return {
    // expose ref values
    userId,
    name,
    email,
    isLoggedIn,
    preferences,
    // expose computed properties
    displayName,
    hasCompleteProfile,
    userPreferences,
    isDarkTheme,
    // expose actions as methods
    login,
    logout,
    updateProfile,
    setTheme,
    toggleTheme,
    setNotifications,
    setPreferences,
    saveToStorage,
    loadFromStorage,
    reset,
  }
})
```

## When and Where to Avoid Composables inside Pania store code logics ~

- 🚫 Danger: At the Root Level or Inside state

```ts
// 🚫 BAD: Composable called at root level
import { useLocalStorage } from '@/composables/useLocalStorage';
import { useRouter } from 'vue-router'; // This is also a composable!

// ❌ This will break! `useRouter()` requires a component context.
const router = useRouter();

// ❌ DANGEROUS: Lifecycle hooks in this composable would be tied to the store, not a component.
const { storedValue } = useLocalStorage('token');

export const useAuthStore = defineStore('auth', {
  state: () => ({
    // ❌ ALSO BAD: Called inside state initializer
    token: useLocalStorage('token').storedValue // Don't do this!
  }),
});
```

- 🚫 Danger: Composables with Lifecycle Hooks

```ts
// composables/useMouse.js
export function useMouse() {
  const x = ref(0);
  const y = ref(0);
  
  function update(e) {
    x.value = e.pageX;
    y.value = e.pageY;
  }
  
  // This composable relies on lifecycle hooks
  onMounted(() => window.addEventListener('mousemove', update));
  onUnmounted(() => window.removeEventListener('mousemove', update));
  // ❌ Using this in a Pinia store would be disastrous.
  
  return { x, y };
}
```

- ✅ Safe: Inside Actions

```ts
// stores/userStore.js
import { defineStore } from 'pinia';
import { useFetch } from '@/composables/useFetch'; // A composable for API calls

export const useUserStore = defineStore('user', {
  actions: {
    async fetchUser(userId) {
      // ✅ GOOD: Using a composable inside an action
      const { data, error, execute } = useFetch(`/api/users/${userId}`);
      
      await execute(); // Execute the API call
      
      if (!error.value) {
        // Update the store's state with the result
        this.user = data.value;
      } else {
        // Handle error, maybe using another composable
        useToast().error('Failed to fetch user!');
      }
    },
    
    async updateUser(userData) {
      // ✅ Also good: Composables for specific utilities
      const validatedData = useDataValidator().validateUser(userData);
      // ... rest of the action
    }
  }
});
```

- ✅ Safe: Inside Getters

```ts
// stores/productStore.js
import { useCurrencyFormatter } from '@/composables/useCurrencyFormatter';

export const useProductStore = defineStore('product', {
  state: () => ({
    products: [],
    currency: 'USD'
  }),
  getters: {
    formattedProducts: (state) => {
      // ✅ GOOD: Using a functional, side-effect-free composable inside a getter
      const { formatPrice } = useCurrencyFormatter(state.currency);
      
      return state.products.map(product => ({
        ...product,
        formattedPrice: formatPrice(product.price)
      }));
    }
  }
});
```

### What About the Composition API Syntax for Pinia?

```ts
export const useUserStore = defineStore('user', () => {
  // State
  const user = ref(null);
  
  // ❌ STILL BAD: Called at root level of setup function
  // const { data } = useFetch('/api/user');
  
  // Actions
  const fetchUser = async (userId) => {
    // ✅ GOOD: Called inside an action (which is a function)
    const { data, error, execute } = useFetch(`/api/users/${userId}`);
    await execute();
    user.value = data.value;
  };
  
  return {
    user,
    fetchUser
  };
});
```
