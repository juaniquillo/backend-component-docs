# Start Here

Welcome to **Laravel Backend Component**. This package lets you build dynamic, dynamic HTML components in PHP, no raw Blade HTML required.

## What is it?

Instead of scattering Blade HTML snippets across your views, you compose component trees entirely in PHP using objects. These components implement Laravel's `Htmlable` interface, so they render safely in Blade with simple `{{ $component }}` syntax.

## Why use it?

- **Composable**, nest components inside other components
- **Themeable**, apply Tailwind CSS classes via a structured theme system
- **Serializable**, export component trees to arrays and restore them later
- **Livewire-ready**, wrap any component as a Livewire component
- **Localizable**, override components and themes from your app's own views

## Quick tour

```php
use Juaniquillo\BackendComponents\Builders\ComponentBuilder;
use Juaniquillo\BackendComponents\Enums\ComponentEnum;

// Create a button
$button = ComponentBuilder::make(ComponentEnum::BUTTON)
    ->setContent('Click me')
    ->setTheme('action', 'success');

// Compose it inside a div
$card = ComponentBuilder::make(ComponentEnum::DIV)
    ->setAttribute('class', 'card')
    ->setContent(
        ComponentBuilder::make(ComponentEnum::H2)->setContent('Title')
    )
    ->setContent($button);

// Render in Blade
// {{ $card }}
```

## Prerequisites

- PHP 8.3+
- Laravel 11, 12, or 13

## What's next?

- [Intro and install](02-intro-and-install.md), installation, Tailwind config, setup
- [How to use](03-how-to-use.md), creating components, content, attributes, nesting
- [Builders and Enum](04-builders-and-enum.md), component builders and the component catalog
- [Theming](05-theming.md), the Tailwind theme system
- [Blade use](06-blade-use.md), template conventions and rendering
- [Helpers](07-helpers.md), namespaced helper functions
- [Notes](08-notes.md), architecture, conventions, testing, adding components
