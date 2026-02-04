---
title: "Vector: Vue in Blade, the easy way"
slug: vector
publishDate: 3 Feb 2026
description: Laravel package that lets you write Vue directly in Blade templates using a simple <script setup> tag.
---

You know that feeling when you're building a Laravel app and you just need a tiny bit of reactivity? A counter. A toggle. Something that feels overkill for a full Vue component but too annoying for vanilla JavaScript?

I kept reaching for Alpine.js, which is great, but I wanted Vue's Composition API. The `ref()`, the `computed()`, the familiar syntax I already know. So I built Vector.

## What Even Is This?

Vector is a Laravel package that lets you write Vue directly in your Blade templates with zero ceremony:

```blade
<script setup>
    const i = ref(0);
</script>

<div>
    <button @click="i++">Click Me</button>
    <div>
        Count: @{{ i }}
    </div>
    <div v-if="i > 5">Success!</div>
</div>
```

That's the whole thing. No build step for your components. No separate `.vue` files. No special directives wrapping your code. Just a `<script setup>` tag and you're done.

## How It Works

The `<script setup>` tag gets transformed at compile time. Vector treats the **element immediately after** the script tag as your Vue template. Everything inside that element becomes reactive, and anything outside it remains regular Blade.

1. Blade's precompiler finds your `<script setup>` blocks
2. Extracts your variable declarations
3. Mounts Vue on the next sibling element

The key part is the variable extraction. It parses `const`, `let`, and `var` declarations and auto-returns them to the template. You write normal code, it figures out the rest.

### Escaping Blade Syntax

Since Blade also uses `{{ }}` for output, you need to prefix Vue's mustache syntax with `@` to prevent Blade from processing it:

```blade
{{-- This is Blade --}}
{{ $phpVariable }}

{{-- This is Vue (note the @) --}}
@{{ vueVariable }}
```

Alternatively, use Vue directives like `v-text` which don't conflict with Blade:

```blade
<span v-text="count"></span>
```

## Installation

```bash
composer require brunoabpinto/vector
```

Add Vector to your Vite entry points in `vite.config.js`:

```javascript
plugins: [
    laravel({
        input: [
            "resources/css/app.css",
            "resources/js/app.js",
            "resources/js/vendor/vector.js",
        ],
        // ...
    }),
],
resolve: {
    alias: {
        'vue': 'vue/dist/vue.esm-bundler.js',
    },
},
```

Add `@vectorJs` before your closing `</body>` tag in your layout:

```blade
<body>
    {{ $slot }}

    @vectorJs
</body>
```

That's it. Vector auto-publishes its runtime, and `@vectorJs` loads it where you need it.

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

## Try It

The package is available on [GitHub](https://github.com/brunoabpinto/vector). Star it, fork it, tell me it's an abomination. Whatever feels right.

```bash
composer require brunoabpinto/vector
```
