# Intro and Install

## What is Laravel Backend Component?

A Laravel package for building dynamic, dynamic HTML components entirely in PHP. Component trees are composed via PHP objects that implement `Htmlable`, rendering safely in Blade with `{{ $component }}`.

Internally, the package resolves each component to a Blade view from its namespaced path and renders it with the component's attributes and content, giving you the full power of Laravel Blade without writing HTML markup by hand.

### Key features

- **50+ HTML components**, divs, buttons, forms, tables, lists, headers, and more
- **Component composition**, nest any component inside another
- **Theme system**, Tailwind CSS class management via PHP arrays
- **Livewire integration**, wrap components as Livewire components
- **Serialization**, export/import component trees to/from arrays
- **Three builder flavors**, choose between package views, local app views, or hybrid

## Installation

Require the package via Composer:

```bash
composer require juaniquillo/laravel-backend-component
```

The package auto-discovers its service provider (`Juaniquillo\BackendComponents\BackendComponentsServiceProvider`) through Laravel's package discovery mechanism.

### Laravel Boost Skill

If you use [Laravel Boost](https://github.com/aleksipopovicdev/laravel-boost), install the AI skill for AI-assisted component development:

```bash
php artisan boost:add-skill https://github.com/juaniquillo/laravel-backend-component
```

This installs the skill file and auto-loaded guidelines so AI tools understand the package's API, component enums, theme system, and coding conventions.

## Tailwind CSS configuration

If you use the package's theme system (Tailwind-based), you need to tell Tailwind to scan the package's Blade files.

### Tailwind CSS v3

Add the package view path to your `tailwind.config.js`:

```js
// tailwind.config.js
export default {
    content: [
        './vendor/juaniquillo/laravel-backend-component/resources/views/**/*.blade.php',
        // your other paths
    ],
};
```

### Tailwind CSS v4

Use the `@source` at-rule in your main CSS file:

```css
@import 'tailwindcss';
@source '../../vendor/juaniquillo/laravel-backend-component/resources/views/**/*.blade.php';
```

## Publishing config

The package ships with a config file at `config/backend-component.php`. You can publish it:

```bash
php artisan vendor:publish --tag=backend-component-config
```

The config is currently extensible (returns an empty array by default).

## View namespace

All Blade views are registered under the `backend-component::` namespace:

- Components: `backend-component::components.div`, `backend-component::components.inline.button`
- Themes: `backend-component::_themes.tailwind.action`
- Internal utilities: `backend-component::_utilities.resolve-components`, `backend-component::_utilities.resolve-component`

You don't need to reference these directly, the builders resolve the correct paths for you.
