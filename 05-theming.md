# Theming

## Overview

The package includes a theme system designed for Tailwind CSS class management. Instead of hardcoding CSS classes in Blade templates or in PHP, you define them in dedicated theme files and apply them by name.

## Theme files

Theme files live in `resources/views/_themes/tailwind/`. Each file returns a PHP array of CSS class strings keyed by variant name.

### Available themes

The package ships with 27 theme files:

`action`, `background`, `border`, `border-radius`, `box-shadow`, `color`, `cursor`, `display`, `flex`, `font`, `grid`, `inputs`, `line-height`, `lists`, `margin`, `max-width`, `modal`, `overflow`, `padding`, `position`, `ring`, `separator`, `size`, `table`, `text`, `transition`, `z-index`

### Theme file structure

```php
<?php
// resources/views/_themes/tailwind/action.blade.php

$allButtons = "whitespace-nowrap ";
$allActions = "disabled:opacity-30 transition duration-150 ease-in-out ";

return [
    'default'           => $allButtons.$allActions."bg-blue-700 hover:bg-blue-800 focus:ring-blue-300",
    'error'             => $allButtons.$allActions."bg-red-700 hover:bg-red-800 focus:ring-red-300",
    'success'           => $allButtons.$allActions."bg-green-700 hover:bg-green-800 focus:ring-green-300",
    'secondary'         => $allButtons.$allActions."bg-gray-700 hover:bg-gray-800 focus:ring-gray-300",
    'info'              => $allButtons.$allActions."bg-cyan-600 hover:bg-cyan-700 focus:ring-cyan-300",
    'warning'           => $allButtons.$allActions."bg-yellow-300 hover:bg-yellow-400 focus:ring-yellow-400",
    'secondary-light'   => $allButtons.$allActions."bg-gray-500 hover:bg-gray-600 focus:ring-gray-300",
    'success-lighter'   => $allActions."bg-green-200 hover:bg-green-300 focus:ring-green-300",
    'link'              => $allActions."text-blue-500 underline hover:no-underline",
    'link-error'        => $allActions."text-red-500 underline hover:no-underline",
    'link-success'      => $allActions."text-green-500 underline hover:no-underline",
];
```

Despite the `.blade.php` extension (required for view discovery), these files are pure PHP,they don't contain any Blade syntax.

## Applying themes

### Single variant

```php
$button = ComponentBuilder::make(ComponentEnum::BUTTON)
    ->setTheme('action', 'success');
```

The first argument is the theme file name (`action`), the second is the variant key (`success`).

### Multiple variants from one theme file

Some themes support an array of variant keys. The theme system resolves each key and concatenates the classes:

```php
$th = ComponentBuilder::make(ComponentEnum::TH)
    ->setTheme('table', ['th', 'th-dark']);
```

This resolves both `th` and `th-dark` from the `table` theme file and applies their combined classes.

### Batch apply

```php
$button = ComponentBuilder::make(ComponentEnum::BUTTON)
    ->setThemes([
        'action' => 'success',
        'size' => 'lg',
    ]);
```

### Theme accumulation

Themes accumulate by default,calling `setTheme` with the same name appends rather than replaces:

```php
$button = ComponentBuilder::make(ComponentEnum::BUTTON)
    ->setTheme('action', 'success')
    ->setTheme('action', 'error');

// theme['action'] = ['success', 'error']
```

Duplicate values are automatically filtered:

```php
$button = ComponentBuilder::make(ComponentEnum::BUTTON)
    ->setTheme('action', 'success')
    ->setTheme('action', 'success');

// theme['action'] = ['success'],no duplicate
```

To replace a theme instead of accumulating, pass `overwrite: true`:

```php
$button = ComponentBuilder::make(ComponentEnum::BUTTON)
    ->setTheme('action', 'success')
    ->setTheme('action', 'error', overwrite: true);

// theme['action'] = 'error',replaced, not accumulated
```

The same applies to `setThemes`:

```php
$button = ComponentBuilder::make(ComponentEnum::BUTTON)
    ->setTheme('action', 'success')
    ->setThemes(['action' => 'error'], overwrite: true);

// theme['action'] = 'error'
```

### Retrieving themes

```php
$variant = $component->getTheme('action');     // 'success' (or whatever was set)
$all = $component->getThemes();                 // ['action' => 'success', 'size' => 'lg']
```

## How themes merge with classes

The `DefaultAttributeBag` merges compiled theme classes with any explicit `class` attribute:

```php
$div = ComponentBuilder::make(ComponentEnum::DIV)
    ->setAttribute('class', 'my-custom-class')
    ->setTheme('display', 'inline-block');

// Result: class="my-custom-class inline-block"
```

Explicit class attributes come first, then theme classes.

## Theme managers

The theme system is handled by `ThemeManager` implementations:

### DefaultThemeManager

Points to the package's `resources/views/_themes/tailwind` directory. Used by `ComponentBuilder`.

### LocalThemeManager

Points to the consuming application's `resources/views/_themes/tailwind` directory. Used by `LocalComponentBuilder` and `LocalThemeComponentBuilder`.

Both use the `IsThemeManager` trait which:
1. Reads `.blade.php` theme files from disk
2. Caches loaded theme arrays in-memory via `DefaultCache`
3. Resolves themes by variant key to CSS class strings

### Override the theme manager

```php
use Juaniquillo\BackendComponents\Themes\LocalThemeManager;

$component = ComponentBuilder::make(ComponentEnum::BUTTON);
$component->setThemeManager(new LocalThemeManager);
```

## Processing themes programmatically

You can process themes outside of a component using the helper:

```php
use function Juaniquillo\BackendComponents\processThemes;

$classes = processThemes(['action' => 'success']);
// Returns: "whitespace-nowrap transition duration-150 ease-in-out bg-green-700 hover:bg-green-800 focus:ring-green-300"
```
