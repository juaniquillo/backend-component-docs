# How to Use

## Creating a component

Every component starts with the builder. The recommended way is using `ComponentBuilder::make()` with a `ComponentEnum` case:

```php
use Juaniquillo\BackendComponents\Builders\ComponentBuilder;
use Juaniquillo\BackendComponents\Enums\ComponentEnum;

$button = ComponentBuilder::make(ComponentEnum::BUTTON);
$div = ComponentBuilder::make(ComponentEnum::DIV);
$paragraph = ComponentBuilder::make(ComponentEnum::PARAGRAPH);
```

You can also use the `MainBackendComponent` class directly:

```php
use Juaniquillo\BackendComponents\MainBackendComponent;

$button = new MainBackendComponent('inline.button');
$div = new MainBackendComponent('div');
```

The second parameter accepts a `ThemeManager` instance to control where theme files are loaded from. The default uses the package's own `resources/views/_themes/tailwind/` directory; pass a `LocalThemeManager` to resolve themes from the application's `resources/views/` instead:

```php
use Juaniquillo\BackendComponents\Themes\LocalThemeManager;

$div = new MainBackendComponent('div', new LocalThemeManager);
```

## Setting content

Use `setContent()` for a single item and `setContents()` for multiple items at once:

```php
$div = ComponentBuilder::make(ComponentEnum::DIV)
    ->setContent('Hello')                               // appended (no key)
    ->setContent('World', 'key_1')                      // appended with key
    ->setContents([                                      // batch (ignores keys)
        ComponentBuilder::make(ComponentEnum::SPAN)->setContent('A'),
        ComponentBuilder::make(ComponentEnum::SPAN)->setContent('B'),
    ])
    ->setContents([                                      // batch with keys (overwrites existing)
        'item_1' => ComponentBuilder::make(ComponentEnum::SPAN)->setContent('A'),
        'item_2' => ComponentBuilder::make(ComponentEnum::SPAN)->setContent('B'),
    ], overwrite: true)
    ->prependContent('First')                           // prepended (no key)
    ->prependContent('Really First', 'key_0')           // prepended with key
    ->unsetContent()                                     // clear all content
    ->unsetContent('key_1');                             // remove by key
```

### Prepend and unset

- `prependContent($content, $key = null)`, adds content at the beginning of the content array
- `unsetContent($key = null)`, if a key is given, removes that specific item; otherwise clears all content

### Accessing content

```php
$content = $component->getContent('key_1');   // single item by key
$all = $component->getContents();              // full content array
```

## Setting attributes

Use `setAttribute()` for a single attribute and `setAttributes()` for multiple at once:

```php
$div = ComponentBuilder::make(ComponentEnum::DIV)
    ->setAttribute('id', 'my-id')
    ->setAttribute('class', 'custom-class')
    ->setAttributes(['data-foo' => 'bar', 'style' => 'display:none']);
```

## Composition (nesting)

Components can be nested by passing them as content:

```php
$card = ComponentBuilder::make(ComponentEnum::DIV)
    ->setAttribute('class', 'card')
    ->setContent(
        ComponentBuilder::make(ComponentEnum::H2)->setContent('Card Title')
    )
    ->setContent(
        ComponentBuilder::make(ComponentEnum::PARAGRAPH)->setContent('Card body text')
    )
    ->setContent(
        ComponentBuilder::make(ComponentEnum::BUTTON)
            ->setContent('Read more')
            ->setTheme('action', 'link')
    );
```

## Individual components

`DivComponent` is both a utility **and** a blueprint for creating new targeted component classes that bypass the enum/builder entirely. To create a new individual component, duplicate the `DivComponent` pattern:

1. Put the class in `src/Components/Individual/`
2. Implement `BackendComponent`, `IndividualComponent`, `ThemeComponent`, `Htmlable` (omit `ContentsComponent`+`HasContent` for self-closing elements)
3. Use traits `IsBackendComponent`, `IsThemeable` (add `HasContent` only if the component can hold children)
4. Define `getName()` to return the `ComponentEnum` value (or any dotted view path)
5. Define `getComponentPath()` and `getPathOnly()` following the existing convention
6. Wire up `getAttributeBag()`, `toHtml()`, `toArray()`, and a static `make()` factory
7. Mark the class `final` (optional but recommended)

```php
use Juaniquillo\BackendComponents\Components\Individual\DivComponent;

$div = new DivComponent;
$div->setAttribute('class', 'my-class');
$div->setContent('Hello');

// Or via the static factory
$div = DivComponent::make();
```

Currently only `DivComponent` exists in this category, add more as needed.

## PropsComponent (third-party packages)

`PropsComponent` in this package is an example of how third-party packages can create custom components with typed props. Create your own class implementing the same interfaces and using the same traits in your own namespace. (The example class aliases the contract as `PropsContract` to avoid colliding with its own class name — the interface itself is `Contracts\PropsComponent`.)

The interfaces to implement: `BackendComponent`, `ContentComponent`, `Htmlable`, `PathComponent`, `PropsComponent`, `SettingsComponent`, `ThemeComponent`.

The traits to use: `HasContent`, `HasPath`, `HasProps`, `HasSettings`, `IsBackendComponent`, `IsThemeable`.

### Setting props

Use **`setProp()`** for a single prop and **`setProps()`** for multiple at once:

```php
// In your own class:
$component->setProp('title', 'Hello')
    ->setProps(['count' => 5, 'items' => ['a', 'b']]);
```

Use **`getProp()`** and **`getProps()`** to retrieve:

```php
$title = $component->getProp('title');   // 'Hello'
$all = $component->getProps();           // ['title' => 'Hello', 'count' => 5, 'items' => ['a', 'b']]
```

### Blade template conventions

The Blade template resolves from the dotted path (e.g., `path.to.component` → `resources/views/components/path/to/component.blade.php`). Use `@props()` to declare expected props:

```blade
@props([
    'title' => null,
    'data' => [],
])

<div {{ $attributes->merge(['title' => $title]) }}>
    {{ $title }}
</div>
```

Non-scalar props (arrays, objects) are not rendered automatically — map them to `data-*` attributes (or similar) in your own template:

```blade
@props([
    'dataProps' => [],
    'customProp' => null,
])

@php
    $localAttrs = [];
    foreach ($dataProps as $key => $value) {
        $localAttrs["data-{$key}"] = $value;
    }
    if($customProp) {
        $localAttrs[$customProp->key] = $customProp->value;
    }
@endphp

<div {{ $attributes->merge($localAttrs) }}>{{ $content }}{{ $slot }}</div>
```

### Rendering

`toHtml()` uses `backend-component::_utilities.resolve-third-party-component`, which renders `<x-dynamic-component>` with the resolved path and attribute bag.

Use `{{ $component }}` in Blade to render.

### Feature testing

```php
$this->blade('{{ $component }}', ['component' => $component])
    ->assertSee('Hello');
```

## Settings

Components accept a key-value settings bag, useful for passing configuration to custom components like `FORM`:

```php
$component = ComponentBuilder::make(ComponentEnum::DIV)
    ->setSetting('transition', 'fade')
    ->setSettings(['setting_1' => 'value_1', 'setting_2' => 'value_2']);

$value = $component->getSetting('transition');

$component->unsetSetting('setting_1');
```

## Modal utility

`ModalUtil` builds a complete modal component tree using Alpine.js for interactivity:

```php
use Juaniquillo\BackendComponents\Utils\ModalUtil;

$modal = ModalUtil::make(
    content: 'Hello World',
    button: ComponentBuilder::make(ComponentEnum::BUTTON)
        ->setContent('Open')
        ->setAttribute('@click', 'showModal = true'),
    title: ComponentBuilder::make(ComponentEnum::H2)->setContent('Modal Title'),
    footer: ComponentBuilder::make(ComponentEnum::DIV)->setContent('Footer'),
)
    ->setAttribute('id', 'my-modal')
    ->setTheme('modal', 'lg')
    ->getComponent();
```

The modal is composed from `DIV` components with Alpine.js attributes, no separate blade template needed.

| Parameter | Type | Description |
|---|---|---|
| `content` | `string\|int\|CompoundComponent` | Main body content (required) |
| `button` | `?CompoundComponent` | Trigger button (optional) |
| `title` | `?CompoundComponent` | Title section (optional) |
| `footer` | `?CompoundComponent` | Footer section (optional) |
| `overlay` | `?CompoundComponent` | Overlay element (defaults to themed overlay) |

Methods `setAttribute()`, `setAttributes()`, `setTheme()`, and `setThemes()` configure the inner content container.

## Theme resolution

All entry points (`ComponentBuilder::make()`, `new MainBackendComponent()`, `ModalUtil::make()`, `TableUtil::make()`) accept an optional `ThemeManager` instance that controls where theme files are loaded from.

| Builder / Constructor | Default | App-local alternative |
|---|---|---|
| `ComponentBuilder::make($enum)` | Package themes | `->useLocal()` or pass `LocalThemeManager` |
| `new MainBackendComponent($name)` | Package themes | `new MainBackendComponent($name, new LocalThemeManager)` |
| `ModalUtil::make(...)` | Package themes | `new LocalThemeManager` as `$themeManager` param |
| `TableUtil::make(...)` | Package themes | `new LocalThemeManager` as `$themeManager` param |

`LocalThemeManager` resolves theme files from the application's `resources/views/_themes/tailwind/` directory instead of the package's.

## Livewire

Any component can be wrapped as a Livewire component:

```php
$livewire = ComponentBuilder::make(ComponentEnum::DIV)
    ->setLivewire()
    ->setLivewireKey('my-key')
    ->setLivewireParams(['userId' => 1, 'team' => 'engineering']);
```

When rendered, Livewire components use `@livewire($name, $params, key($key))` instead of the standard Blade component path.

## Table utilities

`TableUtil` builds a complete `<table>` component tree from head and body arrays:

```php
use Juaniquillo\BackendComponents\Utils\TableUtil;
use Juaniquillo\BackendComponents\Utils\CellBag;

$table = TableUtil::make(
    head: ['Name', 'Email', 'Role'],
    body: [
        [                         // plain values
            'Alice',
            'alice@example.com',
            'Admin',
        ],
        [                         // CellBag for per-cell control
            new CellBag(content: 'Bob', theme: ['color' => 'success']),
            'bob@example.com',
            'Editor',
        ],
    ],
)
    ->setCaption('Team members')
    ->setTableThemes(['table' => 'table'])
    ->setThThemes(['table' => ['th', 'th-dark']])
    ->setTdThemes(['table' => ['td', 'td-dark']])
    ->getComponent();
```

`CellBag` is a readonly value object that bundles content, optional per-cell theme overrides, and optional per-cell HTML attributes.

## Serialization

Export a component tree to an array and restore it later:

```php
use Juaniquillo\BackendComponents\Factories\ComponentFactory;

// Serialize
$array = $component->toArray();

// Restore
$restored = ComponentFactory::fromArray($array);
```

This works recursively for nested content, themes, settings, custom paths, and Livewire state.

## Cached components

`CachedBackendComponent` caches its rendered HTML output to disk. It implements the same interfaces as `MainBackendComponent` and adds caching via PSR-16 (`Psr\SimpleCache\CacheInterface`).

```php
use Juaniquillo\BackendComponents\Components\CachedBackendComponent;

$button = new CachedBackendComponent(ComponentEnum::BUTTON);
$html = $button->getCachedHtml();   // renders + caches on first call
$html = $button->getCachedHtml();   // served from cache on subsequent calls
$button->clearCache();              // invalidates the cached entry
```

The cache key is generated from `md5(json_encode($toArray()))`, same component state always produces the same key. Livewire components bypass caching automatically.

### Cache configuration

```php
$component = new CachedBackendComponent(ComponentEnum::DIV);
$component->setCacheDirectory('/custom/path'); // override default
$component->disableCache();                    // bypass cache entirely
$component->enableCache();                     // re-enable after disabling
$component->isCacheEnabled();                  // check current state
$component->getCacheKey();                     // get the md5 hash key
```

Default cache directory: `cache/backend-components/` in the project root.

### Using the IsCachable trait directly

Any component class can use the `IsCachable` trait for caching:

```php
use Juaniquillo\BackendComponents\Concerns\IsCachable;

class MyCustomComponent implements BackendComponent {
    use IsBackendComponent, IsCachable;
    // ...
}
```

### Swapping cache backends

Since the trait type-hints `Psr\SimpleCache\CacheInterface`, you can inject any PSR-16 implementation (e.g., Laravel's cache store) without changing the component code.

### When to cache

Cache components that are expensive to render (complex nested trees, many theme compilations) and whose content doesn't change per-request, such as documentation pages, static navigation, footer blocks, or reusable layout sections. Avoid caching components with dynamic or user-specific content unless you handle invalidation.

## Rendering in Blade

Components implement `Htmlable`, so you use `{{ $component }}`, never `{!! !!}` or `echo`:

```blade
{{ $card }}
```

The `__toString()` method returns a JSON representation (`component->toArray()` encoded), useful for debugging or API responses.
