Vue 3.4 introduced a shiny new compiler macro called `defineModel`, and it’s one of those features that makes you wonder: _why did we ever write `props` + `emits` manually?_

If you’ve ever built components that need `v-model`, you know the drill: props for `modelValue`, an emit for `update:modelValue`, and a bit of boilerplate to wire them together. `defineModel` takes all of that and turns it into a single, clean line of code.

Let’s peel back the curtain and see what’s really happening.

---

## ✨ The Simple Example

Here’s a minimal parent + child setup:

**App.vue**

```vue
<script setup>
import { ref } from 'vue'
import TextInput from './TextInput.vue'

const textModel = ref('')
</script>

<template>
  <TextInput v-model="textModel" />
  <p>Name: {{ textModel }}</p>
</template>
```

**TextInput.vue**

```vue
<script setup>
const modelValue = defineModel()
</script>

<template>
  <input v-model="modelValue" />
</template>
```

That’s it. No props. No emits. Just `defineModel`.

---

## 🛠️ What Vue Compiles It Into

When you hit save, Vue’s compiler does the heavy lifting. `defineModel()` gets expanded into a `useModel` call plus generated props + emits.

**TextInput.vue (source)** | **Compiled (JS)**

```vue
<script setup>
const modelValue = defineModel()
</script>

<template>
  <input v-model="modelValue" />
</template>
```

```js
import { useModel as _useModel } from 'vue'

const __sfc__ = {
  __name: 'TextInput',
  props: {
    "modelValue": {},
    "modelModifiers": {},
  },
  emits: ["update:modelValue"],
  setup(__props, { expose: __expose }) {
    __expose();

    // 👇 defineModel turns into useModel
    const modelValue = _useModel(__props, "modelValue")

    return { modelValue }
  }
}

function render(_ctx, _cache, $props, $setup) {
  return _withDirectives(
    _openBlock(),
    _createElementBlock("input", {
      "onUpdate:modelValue": $event => ($setup.modelValue = $event)
    }, null, 512),
    [[_vModelText, $setup.modelValue]]
  )
}
```

---

## 🔍 Breaking It Down

So what exactly is happening here?

1. **⚡ Props & Emits are generated for you:**
    
    ```js
    props: { modelValue: {}, modelModifiers: {} },
    emits: ["update:modelValue"]
    ```
    
2. **⚡ `defineModel` turns into `useModel`:**
    
    ```js
    const modelValue = useModel(__props, "modelValue")
    ```
    
    This creates a local ref that:
    
    - Reads from `props.modelValue`
        
    - Emits `update:modelValue` when assigned to
        
3. **⚡ Template reactivity stays clean:**  
    The `v-model="modelValue"` in your template is just sugar. Under the hood, it compiles into:
    
    ```js
    onUpdate:modelValue: $event => ($setup.modelValue = $event)
    ```
    
    Which calls the setter from `useModel` and triggers the parent update.
    

---

## 🚀 Why This Matters

- **Less boilerplate** → No more writing `props` + `emits` manually.
    
- **Cleaner mental model** → Treat your model like a local `ref`.
    
- **Type safety** → Works with TypeScript generics (`defineModel<string>()`).
    
- **Multiple models** → Add more with `defineModel('name')`, `defineModel('content')`.
    

---

## 🧠 The Mental Model

Think of `defineModel` like this:

```
Parent v-model  ⇄  Child’s local ref (via useModel)
```

- The child sees a normal reactive ref (`modelValue`).
    
- The parent gets updates automatically, without extra emits.
    
- Both stay in sync.
    

---

## 💡 Final Thoughts

Vue’s `defineModel` is one of those quality-of-life upgrades that make your codebase cleaner and your brain lighter. Instead of wiring up props and emits by hand, you just declare a reactive ref, and Vue does the rest.

It’s reactivity the way it should feel: **local when you need it, two-way when you want it.**