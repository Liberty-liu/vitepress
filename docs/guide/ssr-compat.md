---
outline: deep
---

# SSR Compatibility

VitePress pre-renders the app in Node.js during the production build, using Vue's Server-Side Rendering (SSR) capabilities. This means all custom code in theme components are subject to SSR Compatibility.

The [SSR section in official Vue docs](https://vuejs.org/guide/scaling-up/ssr.html) provides more context on what is SSR, the relationship between SSR / SSG, and common notes on writing SSR-friendly code. The rule of thumb is to only access browser / DOM APIs in `beforeMount` or `mounted` hooks of Vue components.

## `<ClientOnly>`

If you are using or demoing components that are not SSR-friendly (for example, contain custom directives), you can wrap them inside the built-in `<ClientOnly>` component:

```md
<ClientOnly>
  <NonSSRFriendlyComponent />
</ClientOnly>
```

## Libraries that Access Browser API on Import

Some components or libraries access browser APIs **on import**. To use code that assumes a browser environment on import, you need to dynamically import them.

### Importing in Mounted Hook

```vue
<script setup>
import { onMounted } from 'vue'

onMounted(() => {
  import('./lib-that-access-window-on-import').then((module) => {
    // use code
  })
})
</script>
```

### Conditional Import

You can also conditionally import a dependency using the `import.meta.env.SSR` flag (part of [Vite env variables](https://vitejs.dev/guide/env-and-mode.html#env-variables)):

```js
if (!import.meta.env.SSR) {
  import('./lib-that-access-window-on-import').then((module) => {
    // use code
  })
}
```

Since [`Theme.enhanceApp`](/guide/custom-theme#theme-interface) can be async, you can conditionally import and register Vue plugins that access browser APIs on import:

```js
// .vitepress/theme/index.js
export default {
  // ...
  async enhanceApp({ app }) {
    if (!import.meta.env.SSR) {
      const plugin = await import('plugin-that-access-window-on-import')
      app.use(plugin)
    }
  }
}
```

### `defineClientComponent`

VitePress provides a convenience helper for importing Vue components that access browser APIs on import.

```vue
<script setup>
import { defineClientComponent } from 'vitepress'

const ClientComp = defineClientComponent(() => {
  return import('component-that-access-window-on-import')
})
</script>

<template>
  <ClientComp />
</template>
```

The target component will only be imported in the mounted hook of the wrapper component.