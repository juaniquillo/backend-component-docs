# Laravel Backend Component

Build dynamic, dynamic HTML components in PHP, no raw Blade HTML required.

## Documentation

| # | Title | Description |
|---|-------|-------------|
| 01 | [Start Here](01-start-here.md) | Overview, quick tour, prerequisites |
| 02 | [Intro and Install](02-intro-and-install.md) | Installation, Tailwind config, view namespace |
| 03 | [How to Use](03-how-to-use.md) | Creating components, content, attributes, nesting, Livewire, serialization |
| 04 | [Builders and Enum](04-builders-and-enum.md) | Three builders, all 50+ enum cases |
| 05 | [Theming](05-theming.md) | Theme file format, single/multi/batch apply |
| 06 | [Blade Use](06-blade-use.md) | Template anatomy, $attrs bag, rendering pipeline |
| 07 | [Helpers](07-helpers.md) | Namespaced helper functions |
| 08 | [Notes](08-notes.md) | Architecture, code style, testing, adding components |

## Install

```bash
composer require juaniquillo/laravel-backend-component
```

## Quick example

```php
use Juaniquillo\BackendComponents\Builders\ComponentBuilder;
use Juaniquillo\BackendComponents\Enums\ComponentEnum;

$button = ComponentBuilder::make(ComponentEnum::BUTTON)
    ->setContent('Click me')
    ->setTheme('action', 'success');
```

## License

[MIT](LICENSE)
