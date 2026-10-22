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

Every component Blade template is a plain view that renders one element:

```blade
{{-- resources/views/components/div.blade.php --}}
<div {{ $attributes }}>{{ $slot }}</div>
```

### Template variables

| Variable | Type | Description |
|---|---|---|
| `$attributes` | `ComponentAttributeBag` | HTML attributes merged with compiled theme classes |
| `$slot` | `Htmlable` | Processed content (renders child components) |
| `$settings` | `array` | Key-value settings bag (`setSetting()` / `setSettings()`) |

Templates that do not use settings simply ignore the variable.

### Self-closing elements

For elements like `input`, `img`, and `col`, use `/>`:

```blade
{{-- resources/views/components/form/text.blade.php --}}
<input {{ $attributes->merge(['type' => 'text']) }} />
```

`merge()` only supplies defaults: attributes set through the PHP API always win over them.

### Delegating templates

Inputs that share the text input markup delegate to it and only change the type:

```blade
{{-- resources/views/components/form/email.blade.php --}}
<x-backend-component::form.text {{ $attributes->merge(['type' => 'email']) }} />
```

## Rendering pipeline

When `{{ $component }}` is called:

1. `MainBackendComponent::toHtml()` loads `_utilities.resolve-components`
2. If `isLivewire`, renders via `@livewire($name, $params, key($key))`
3. Otherwise, `_utilities.resolve-component` builds a `ComponentAttributeBag` from `getAttributeBag()->getAttributes()` (attributes + compiled theme classes)
4. `getViewPath()` resolves the matching Blade template in the `components/` directory (namespace + optional `setPath()` subdirectory + component name), which is included with `$attributes`, `$slot` (processed content) and `$settings`
5. The template renders the HTML element

### Where the view comes from

Every component resolves its template through `getViewPath()`:

| Component | `getViewPath()` |
|---|---|
| `ComponentBuilder::make(ComponentEnum::DIV)` | `backend-component::components.div` |
| `ComponentBuilder::make(ComponentEnum::DIV)->useLocal()` | `components.div` |
| `ComponentBuilder::make('img')->setPath('inline')` | `backend-component::components.inline.img` |

The path is built from the view namespace, the `components/` directory, the optional subdirectory set with `setPath()`, and the component name.

`getComponentPath()` is a different value: the `x-dynamic-component` form leaves the `components/` directory out because the tag resolver adds it. Never pass it to `view()` or `@include`; it resolves to a view such as `backend-component::.div`, which does not exist.

`DivComponent` does not use the `HasPath` trait, so it implements `getViewPath()` itself.

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

When the `isLivewire` flag is set, the rendering pipeline switches to `@livewire` instead of rendering the component's Blade template:

```php
$livewire = ComponentBuilder::make(ComponentEnum::DIV)
    ->setLivewire()
    ->setLivewireKey('dashboard-stats')
    ->setLivewireParams(['teamId' => 5]);
```

This renders as `@livewire('div', ['teamId' => 5], key('dashboard-stats'))`. Attributes, content and settings are not forwarded in this mode.

## Custom Blade templates

When adding a new component type, create a new Blade file in `resources/views/components/` following the conventions above. The file path must match the enum value (e.g., `form.datalist` → `resources/views/components/form/datalist.blade.php`).

Templates are included as plain views, so they must only rely on the `$attributes`, `$slot` and `$settings` variables. Do not declare props or blueprint the component server state.

Plain views are used instead of Blade component tags on purpose: component tags compile attribute names into PHP variables, so Alpine attributes such as `@click` produce invalid PHP. Building a `ComponentAttributeBag` and including the view keeps attribute names untouched and keeps `$settings` out of the HTML.

Third-party packages are not bound by this pipeline; they implement `Contracts\BackendComponent` and choose their own view and rendering strategy.

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
