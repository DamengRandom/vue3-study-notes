# `watch` vs `watchEffect`

## What?

- `watch`: developer tells exactly/explicit data resources to watch !! 

- `watchEffect`: function itself will automatically track the reactive dependencies used inside its callback

```ts
// With watch - explicit dependencies
watch([userId, filters], ([newUserId, newFilters]) => {
  fetchData(newUserId, newFilters)
})

// With watchEffect - automatic dependency tracking
watchEffect(() => {
  fetchData(userId.value, filters.value)
})
```

Start with `watchEffect` for simple side effects and when immediate execution is desired

Use `watch` when you need old values, lazy execution, or explicit dependency control


## When to use which?

### When to use `watch`?

(1). Watching Specific Reactive Sources

```ts
// You explicitly state what to watch
watch(userId, (newId, oldId) => {
  fetchUser(newId)
})

// Watching multiple specific sources
watch([userId, projectId], ([newUserId, newProjectId], [oldUserId, oldProjectId]) => {
  // React to changes in either userId OR projectId
})
```

(2). Need Access to Previous Values

```ts
watch(searchQuery, (newQuery, oldQuery) => {
  // You can compare what changed
  if (newQuery !== oldQuery) {
    searchProducts(newQuery)
  }
})
```

(3). Lazy Execution (Don't Run Immediately)

```ts
// This won't run until userId actually changes
watch(userId, (newId) => {
  fetchUser(newId)
})
```

(4). Conditional Triggering Based on Values

```ts
watch(data, (newData) => {
  if (newData.isValid) {
    submitToServer(newData)
  }
})
```

### When to use `watchEffect`?

(1). Multiple Dependencies That Change Together

```ts
// watchEffect automatically tracks both 'filters' and 'pagination'
watchEffect(() => {
  // If ANY reactive property used here changes, this runs
  const results = searchAPI(filters.search, filters.category, pagination.page)
  results.value = results
})
```

(2). Immediate Execution Needed

```ts
// Runs immediately and whenever dependencies change
watchEffect(() => {
  document.title = `Messages (${unreadCount.value})`
})
```

(3). Simple Side Effects

```ts
// Simple reactions like logging, analytics, etc.
watchEffect(() => {
  console.log(`User ${user.value.name} is viewing page ${route.path}`)
})
```

(4). Complex Computed Side Effects 

```ts
// When your side effect depends on multiple reactive values
watchEffect(() => {
  const shouldLoad = isAuthenticated.value && hasPermission.value && !data.value
  if (shouldLoad) {
    loadUserData()
  }
})
```
