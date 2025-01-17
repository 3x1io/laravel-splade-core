# [Experimental] Laravel Splade Core

A package to use Vue 3's Composition API in Laravel Blade components.

[![Latest Version on Packagist](https://img.shields.io/packagist/v/protonemedia/laravel-splade-core.svg)](https://packagist.org/packages/protonemedia/laravel-splade-core)
[![GitHub Tests Action Status](https://github.com/protonemedia/laravel-splade-core/actions/workflows/run-tests.yml/badge.svg?branch=main&event=label)](https://github.com/protonemedia/laravel-splade-core/actions/workflows/run-tests.yml)
[![Splade Discord Server](https://dcbadge.vercel.app/api/server/qGJ4MkMQWm?style=flat&theme=default-inverted)](https://discord.gg/qGJ4MkMQWm)
[![GitHub Sponsors](https://img.shields.io/github/sponsors/pascalbaljet)](https://github.com/sponsors/pascalbaljet)

## Sponsor Splade

We sincerely value our community support in our endeavor to create and offer free Laravel packages. If you find our package beneficial and rely on it for your professional work, we kindly request your support in [sponsoring](https://github.com/sponsors/pascalbaljet) its maintenance and development. Managing issues and pull requests is a time-intensive task, and your assistance would be greatly appreciated. Thank you for considering our request! ❤️

## Why is this experimental?

Although this package comes with a extensive test suite, it's important to note that it's in an experimental stage. It's a new package, and I haven't used it in production yet. Currently, I'm using the [original Splade v1](https://splade.dev) package for my production needs. However, in 2023 Q4, I plan to integrate this new package into two new projects. It's also worth mentioning that I haven't yet confirmed full compatibility between this package and all the features of the original Splade package.

## What does this have to do with the existing Laravel Splade package?

Laravel Splade currently offers a ton of features:

- Use Blade to build Single-Paged-Applications
- 20+ interactive components
- Extensive Form and Table components
- Modals, Slideovers, Toasts, SEO, SSR, and more

While this is great and tremendously helps to build SPAs with Laravel, it also makes it harder to maintain and extend the package. This is why I decided to split Splade into multiple packages:

- Splade Core: This package. It only contains the core functionality to use Vue 3's Composition API in Blade components. No pre-built components. No markup. No CSS. Just the core functionality.
- Splade Navigation: The SPA and Navigation components from the original Splade package (Modals, Slideovers, Toasts, SEO, SSR, etc.)
- Splade UI: The UI components from the original Splade package, excluding the Form and Table components.
- Splade Form: The Form components from the original Splade package.
- Splade Table: The Table components from the original Splade package.

## Installation

You can install the package via composer:

```bash
composer require protonemedia/laravel-splade-core
```

### Automatic Installation

For new projects, you may use the `splade:core:install` Artisan command to automatically install the package:

```bash
php artisan splade:core:install
```

This will install the JavaScript packages, create a root layout and a demo component, and add the required configuration to your `app.js` and `vite.config.js` files. After running this command, you may run `npm install` to install the JavaScript dependencies, and then run `npm run dev` to start Vite.

```bash
npm install
npm run dev
```

### Manual Installation

First, you should install the companion JavaScript packages:

```bash
npm install @protonemedia/laravel-splade-core @protonemedia/laravel-splade-vite
```

Splade Core automatically generates Vue components for all your Blade components. By default, they are stored in `resources/js/splade`. You don't have to commit these files to your repository, as they are automatically generated when you run Vite. To initialize this directory, run the following command:

```bash
php artisan splade:core:initialize-directory
```

In your main Javascript file (`app.js`), you must instantiate a new Vue app and use the Splade Core plugin. Following Laravel's convention, the `app.js` file is stored in `resources/js`, so you may pass the relative `./splade` path to the plugin options:

```js
import { createApp } from 'vue/dist/vue.esm-bundler.js'
import { SpladeCorePlugin } from '@protonemedia/laravel-splade-core'

createApp()
    .use(SpladeCorePlugin, {
        components: import.meta.glob('./splade/*.vue', { eager: true }),
    })
    .mount('#app')
```

In your `vite.config.js` file, you must add the `laravel-splade-vite` plugin. Make sure it is added before the `laravel` and `vue` plugins:

```js
import { defineConfig } from 'vite'
import laravel from 'laravel-vite-plugin'
import vue from '@vitejs/plugin-vue'
import spladeCore from '@protonemedia/laravel-splade-vite'

export default defineConfig({
    plugins: [
        spladeCore(),
        laravel({
            ...
        }),
        vue({
            ...
        }),
    ],
})
```

Lastly, in your root layout file, you must create a root element for your Vue app, and you must include a script tag for the Splade templates stack. This must be done above the `#app` element:

```blade
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <title>My app</title>
        @vite("resources/js/app.js")
    </head>
    <body>
        <script> @stack('splade-templates') </script>

        <div id="app">
            {{ $slot }}
        </div>
    </body>
</html>
```

## Usage

You may use the default `make:component` Artisan command to generate a Blade Component:

```bash
php artisan make:component MyComponent
```

In the Blade template, you may use a `<script setup>` tag:

```vue
<script setup>
const message = ref('Hello Vue!')
const uppercase = computed(() => message.value.toUpperCase())
</script>

<input v-model="message" />
<p v-text="uppercase" />
```

Besides running Vite, you don't have to do anything else. The component will be automatically compiled and registered as a Vue component.

### Composition API imports

In the example above, we used the `ref` and `computed` functions. Splade Core automatically imports these functions for you. Here's a list of all the functions that are automatically imported:

- `computed`
- `nextTick`
- `onMounted`
- `reactive`
- `readonly`
- `ref`
- `watch`
- `watchEffect`
- `watchPostEffect`
- `watchSyncEffect`

### Attribute Inheritance

Just like regular Blade components, you may use the `$attributes` variable to inherit attributes passed to the component. This also works for `v-model`:

Here's the template of the `<x-form>` component:

```vue
<script setup>
const form = ref({
    framework: 'laravel',
})
</script>

<form>
    <x-select v-model="form.framework" />
</form>
```

And here's the template of the `<x-select>` component:

```blade
<select {{ $attributes }}>
    <option value="laravel">Laravel</option>
    <option value="tailwind">Tailwind</option>
    <option value="vue">Vue</option>
</select>
```

### Element Refs

You may use the `$refs` variable to access element refs. This doesn't naturally work in Vue 3's Composition API, but it was great in Vue 2, so I decided to add it back in Splade Core:

```vue
<script setup>
onMounted(() => {
    const creditcardEl = $refs.creditcard;
});
</script>

<input ref="creditcard" />
```

### Tapping into the Vue ecosystem

Using libraries and packages from the Js/Vue ecosystem is easy. Here's an example of using [Flatpickr](https://flatpickr.js.org):

```vue
<script setup>
import flatpickr from "flatpickr";

const emit = defineEmits(["update:modelValue"]);

onMounted(() => {
    let instance = flatpickr($refs.date, {
       onChange: (selectedDates, newValue) => {
            emit("update:modelValue", newValue);
        },
    });

    instance.setDate(props.modelValue);
});
</script>

<input ref="date" />
```

Note that you can use `props.modelValue` without defining it. Splade Core automatically detects the usage of `modelValue` and adds it to the `props` object.

### Calling methods on the Blade Component

If your Blade Component has a `public` method, you may call it from the Vue component. Splade Core detects the HTTP Middleware of the current page and applies it to requests made from the Vue component.

```php
<?php

namespace App\View\Components;

use Closure;
use Illuminate\Contracts\View\View;
use Illuminate\View\Component;

class UserProfile extends Component
{
    public function notify(string $message)
    {
        auth()->user()->notify($message);
    }

    public function render()
    {
        return view('components.user-profile');
    }
}
```

Template:

```vue
<script setup>
const message = ref('Hey there!')
</script>

<input v-model="message" placeholder="Enter a message" />
<button @click="notify(message)">Notify User</button>
<p v-if="notify.loading">Notifying user...</p>
```

Note that you can use `notify.loading` to check if the method is currently running.

Instead of calling the method from the template, you may also call it from the script. This way, you can use `try/catch` and `finally`:

```vue
<script setup>
function notifyWithFixedMessage() {
    notify('Hey there!')
        .then(() => alert('User notified!'))
        .catch(() => alert('Something went wrong!'))
        .finally(() => {
            //
        })
}
</script>

<button @click="notifyWithFixedMessage">Notify User</button>
```

Alternatively, you may add global callbacks to the `notify` method:

```js
<script setup>
notify.before((data) => {

})

notify.then((response, data) => {

})

notify.catch((e) => {

})
</script>

<input v-model="message" placeholder="Enter a message" />
<button @click="notify('Hey there!')">Notify User</button>
```

## Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information on what has changed recently.

## Contributing

Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

## Security

If you discover any security-related issues, please email <pascal@protone.media> instead of using the issue tracker.

## Credits

- [Pascal Baljet](https://github.com/protonemedia)
- [All Contributors](../../contributors)

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
