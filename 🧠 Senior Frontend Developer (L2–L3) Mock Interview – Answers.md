## Q1: TypeScript Utility Types

### a) Make all fields required and readonly:
```ts
type ReadonlyRequiredUser = Readonly<Required<User>>;
```

### b) Make `email` required and all others optional:
```ts
type EmailRequiredUser = Required<Pick<User, "email">> & Partial<Omit<User, "email">>;
```

---

## Q2: Vue 3 Pattern Design – Modal System

- Use a global modal service (via `provide/inject` or `pinia`) to track open modals.
- Render modals using `<Teleport to="body" />` for stacking.
- Store modal state as an array of modal instances with metadata.
- Ensure each modal traps focus, responds to `Escape`, and handles ARIA.
- Use `v-model:visible`, expose events like `onClose`, `onOpen`, and allow slot-based content injection.

---

## Q3: Frontend Architecture – Multi-Tenant Dashboard

### Strategy:
- **Client Configs**: Store per-tenant config in a `tenants` folder, loaded dynamically.
- **Runtime Theming**: Use CSS variables and `provide/inject` or a global store (e.g., Pinia).
- **Dynamic Routes/Widgets**: Code split using async components and conditional registration via `router.addRoute`.
- **Structure**:
  ```
  src/
    tenants/
      tenantA/config.ts
      tenantB/config.ts
    themes/
    widgets/
  ```
- Inject tenant ID via env or auth context, load config on boot.

---

## Q4: Composable – `useDebouncedSearch`

```ts
import { ref, watch } from "vue";

export function useDebouncedSearch(searchTerm, searchFn, delay = 500) {
  const results = ref([]);
  const isLoading = ref(false);
  let timeout, controller;

  watch(searchTerm, (term) => {
    if (timeout) clearTimeout(timeout);
    if (controller) controller.abort();

    timeout = setTimeout(async () => {
      isLoading.value = true;
      controller = new AbortController();

      try {
        const data = await searchFn(term, controller.signal);
        results.value = data;
      } catch (e) {
        if (e.name !== "AbortError") console.error(e);
      } finally {
        isLoading.value = false;
      }
    }, delay);
  });

  return { results, isLoading };
}
```

---

## Q5: Performance and Web Vitals

### To improve FCP & TTI:
- **Code-split**: lazy load non-critical routes/components.
- **Preload critical assets**: fonts, hero images.
- **Defer 3rd-party scripts** using `async` or `defer`.
- **Compress assets** with Brotli/gzip.
- **Use skeleton loaders** or content placeholders.
- **Optimize fonts**: subset, preload, font-display: swap.
- **Critical CSS inlined**, rest lazy-loaded.
- **Lazy load images** using `loading="lazy"`.

---

## Q6: Testing Strategy for Legacy Codebase

- **Start with E2E tests** (Playwright/Cypress) to cover user flows quickly.
- **Unit tests** for new components using Vitest + Testing Library.
- **Visual regression** with Chromatic or Percy.
- **Use test coverage tools** like `c8` or `nyc` to track progress.
- Gradually refactor legacy code **only when touched**, covered by snapshot/E2E first.
- Hold **testing workshops** to train team.
- Enforce test presence in PRs via CI pipeline.

---

## Q7: System Design – Real-Time Collaborative Whiteboard

### Architecture Overview:
- **Canvas**: Use `<canvas>` or SVG with `Konva.js` or `Fabric.js`.
- **Realtime Sync**: Use WebSockets or CRDTs (e.g. Yjs) with backend (e.g., WebSocket server or Firebase/Liveblocks).
- **Component Structure**: Split into `Canvas`, `Toolbar`, `UserCursors`, `LayerManager`, etc.
- **Undo/Redo**: Maintain action history stack and pointer.
- **Offline Support**: Store local copy using IndexedDB. Use CRDTs to resolve conflicts upon reconnect.
- **Performance**:
  - Throttle updates (cursor/move).
  - Virtualize DOM overlays.
  - Use bitmap caching for static layers.