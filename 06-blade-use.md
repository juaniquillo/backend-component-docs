# Blade Use

## Rendering components in Blade

Components implement `Htmlable`, so they render safely with `{{ }}` syntax:

```blade
{{ $button }}
{{ $card }}
{{ $table }}
```

Never use `{!! !!}` or `echo`, the component handles escaping and HTML generation internally.

## Component view conventions

Every component Blade template follows a consistent pattern:

```blade
{{-- resources/views/components/div.blade.php --}}
@props([
    'attrs' => null,
])

@php
    $serverAttrs = [];
    $content = null;
    $slot = $slot ?? null;

    if($attrs) {
        $serverAttrs = $attrs->getAttributes();
        $content = $attrs->content;
    }
@endphp

<div {{ $attributes->merge($serverAttrs) }}>{{ $content }}{{ $slot }}</div>
```

### Template anatomy

1. **`@props(['attrs' => null])`**, the component receives a `DefaultAttributeBag` as the `$attrs` prop
2. **`@php` block**, extracts server attributes (merged with theme classes), content, and slot
3. **Render**, the HTML element uses `{{ $attributes->merge($serverAttrs) }}` to combine server-set attributes with any attributes passed from Blade

### Self-closing elements

For elements like `input`, `img`, and `col`, use `/>`:

```blade
{{-- resources/views/components/form/text.blade.php --}}
<input {{ $attributes->merge($serverAttrs) }}/>
```

## The $attrs property

`$attrs` is a `DefaultAttributeBag` containing:

| Property | Type | Description |
|---|---|---|
| `getAttributes()` | `array` | Merged server attributes + compiled theme classes |
| `content` | `?ContentsComponent` | Processed content (renders child components) |
| `path` | `?string` | Component view path |
| `settings` | `array` | Key-value settings bag |
| `isLivewire` | `bool` | Whether the component is Livewire |
| `livewireKey` | `?string` | Livewire component key |
| `livewireParams` | `array` | Livewire parameters |

## Rendering pipeline

When `{{ $component }}` is called:

1. `MainBackendComponent::toHtml()` loads `_utilities.resolve-components`
2. If `isLivewire`, renders via `@livewire($name, $params, key($key))`
3. Otherwise, renders via `<x-dynamic-component :component="$path" :attrs="$componentArray" />`
4. The dynamic component resolves to the matching Blade template in the `components/` directory
5. The Blade template receives the `$attrs` prop and renders the HTML

## Passing attributes from Blade

Since components are rendered in PHP and passed to Blade as variables, attributes are set via the PHP API (`setAttribute`, `setAttributes`), not via Blade's attribute syntax.

```php
// In PHP, set attributes via the API
$div = ComponentBuilder::make(ComponentEnum::DIV)
    ->setAttribute('class', 'bg-gray-100')
    ->setAttribute('data-controller', 'dropdown');

// In Blade, just render
// {{ $div }}
```

## Mixing with Livewire

When the `isLivewire` flag is set, the rendering pipeline switches to `@livewire` instead of `x-dynamic-component`:

```php
$livewire = ComponentBuilder::make(ComponentEnum::DIV)
    ->setLivewire()
    ->setLivewireKey('dashboard-stats')
    ->setLivewireParams(['teamId' => 5]);
```

This renders as `@livewire('div', ['teamId' => 5], key('dashboard-stats'))` with the `attrs` data passed as a parameter.

## Custom Blade templates

When adding a new component type, create a new Blade file in `resources/views/components/` following the conventions above. The file path must match the enum value (e.g., `form.datalist` → `resources/views/components/form/datalist.blade.php`).

## Blade component syntax

While not the primary use case, components can also be created directly in Blade using `x-` component syntax. You need to reference the package's view namespace followed by the component's dotted path:

```blade
<x-backend-component::inline.button 
    class="whitespace-nowrap disabled:opacity-30 transition duration-150 ease-in-out bg-blue-700 hover:bg-blue-800 focus:ring-blue-300 py-2 px-4"
    type="button">
        Nice button
</x-backend-component::inline.button>
```

Note: This approach can be verbose for simple components and doesn't give you access to the theme system or component composition features. The PHP builder API is the recommended approach for anything beyond basic usage.
