---
title: "Vector: The easiest way to plug Vue in Blade"
slug: vector
publishDate: 3 Feb 2026
description: Laravel package that lets you write Vue's <script setup> syntax directly in Blade templates. Is it cursed? Maybe. Does it work? Absolutely
---

<!-- @format -->

You know that feeling when you're building a Laravel app and you just need a tiny bit of reactivity? A counter. A toggle. Something that feels overkill for a full Vue component but too annoying for vanilla JavaScript?

I kept reaching for Alpine.js, which is great, but I wanted Vue's Composition API. The `ref()`, the `computed()`, the familiar syntax I already know. So I built Vector.

## What Even Is This?

Vector is a Laravel package that lets you write Vue's `<script setup>` syntax directly in your Blade templates:

```blade
@vector
    <script setup>
        const count = ref(0);
    </script>
@endvector

<div>
    <button @click="count++">Clicked @{{ count }} times</button>
</div>
```

That's it. No build step for your components. No separate `.vue` files. Just Blade with a sprinkle of Vue.

## Why Though?

I was tired of the mental gymnastics:

1. "This needs reactivity"
2. "Should I make a Vue component?"
3. "But it's just a counter..."
4. "Fine, I'll use Alpine"
5. "Wait, how do I do computed properties in Alpine again?"

With Vector, the answer is always: write it like you would in Vue, because it _is_ Vue.

## How It Works

The `@vector` directive does a few things:

1. Captures your `<script setup>` block
2. Strips the Vue imports (they're provided globally)
3. Extracts your variable declarations
4. Generates a script that mounts Vue on the next sibling element

```php
// What @vector generates
<script data-vector="vector-abc123">
(function(__script) {
    function __mount() {
        const { createApp, ref } = window.Vue;

        const count = ref(0);

        createApp({
            setup() {
                return { count };
            }
        }).mount(__script.nextElementSibling);
    }
    // ... waits for Vue to be available
})(document.currentScript);
</script>
```

The magic is in the variable extraction. It parses `const`, `let`, and `var` declarations and auto-returns them to the template. You write normal code, it figures out the rest.

## Installation

```bash
composer require brunoabpinto/vector
```

Then expose Vue globally in your `app.js`:

```javascript
import * as Vue from "vue";
window.Vue = Vue;
```

And update your `vite.config.js` to use Vue's runtime compiler:

```javascript
resolve: {
    alias: {
        'vue': 'vue/dist/vue.esm-bundler.js',
    },
},
```

## The Trade-offs

Let's be real about what this is:

**Good for:**

- Quick interactive elements
- Prototyping
- When you want Vue's API without Vue's ceremony
- Laravel apps that are mostly server-rendered with islands of reactivity

**Not great for:**

- Complex component hierarchies
- When you need proper SFC features (scoped styles, etc.)
- Large-scale SPAs (just use Inertia at that point)

## Multiple Components? No Problem

Each `@vector` block is independent:

```blade
@vector
    <script setup>
        const name = ref('World');
    </script>
@endvector

<div>
    <input v-model="name" />
    <p>Hello, @{{ name }}!</p>
</div>

@vector
    <script setup>
        const items = ref(['Apple', 'Banana']);
        const count = computed(() => items.value.length);
    </script>
@endvector

<ul>
    <li v-for="item in items">@{{ item }}</li>
    <p>@{{ count }} items</p>
</ul>
```

## Is This Cursed?

A little bit, yes. We're essentially doing runtime compilation of Vue templates, which goes against the "compile everything ahead of time" philosophy.

But sometimes the right tool is the one that gets out of your way. And for those moments when you just want to add a reactive counter to your Blade view without spinning up a whole component ecosystem, Vector is there.

## Try It

The package is available on GitHub. Star it, fork it, tell me it's an abomination—whatever feels right.

```bash
composer require brunoabpinto/vector
```
