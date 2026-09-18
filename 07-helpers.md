# Helpers

The package provides several namespaced helper functions in `src/helpers.php`. Import them with `use function`:

```php
use function Juaniquillo\BackendComponents\processThemes;
use function Juaniquillo\BackendComponents\processLocalThemes;
use function Juaniquillo\BackendComponents\backendComponentNamespace;
use function Juaniquillo\BackendComponents\makeBackendComponent;
use function Juaniquillo\BackendComponents\cache;
use function Juaniquillo\BackendComponents\isComponent;
use function Juaniquillo\BackendComponents\isBackedEnum;
use function Juaniquillo\BackendComponents\isCellBag;
```

## backendComponentNamespace()

Returns the view namespace used by the package:

```php
$namespace = backendComponentNamespace();
// 'backend-component::'
```

## makeBackendComponent()

Quick factory for creating a `MainBackendComponent`:

```php
$component = makeBackendComponent('div');
$component = makeBackendComponent(ComponentEnum::BUTTON, new LocalThemeManager);
```

Second parameter accepts an optional `ThemeManager` instance.

## processThemes()

Process an array of themes using the default theme manager and return the compiled CSS class string:

```php
$classes = processThemes(['action' => 'success']);
// "whitespace-nowrap disabled:opacity-30 transition duration-150 ease-in-out bg-green-700 hover:bg-green-800 focus:ring-green-300"
```

Second parameter accepts an optional `ThemeManager` instance.

## processLocalThemes()

Same as `processThemes()` but uses `LocalThemeManager` to resolve themes from the app's local `resources/views/_themes/tailwind/`:

```php
$classes = processLocalThemes(['display' => 'none']);
```

## cache()

Returns a named `DefaultCache` instance (in-memory key-value store, persists for the request lifetime):

```php
$myCache = cache('theme-files');
$myCache->set('action', $data);
$value = $myCache->get('action');
```

Used internally by the theme manager to cache loaded theme file data.

## isComponent()

Type assertion,checks if a value implements `BackendComponent`:

```php
if (isComponent($value)) {
    // $value is a BackendComponent
}
```

PHPStan understands this as a type-narrowing assertion (`@phpstan-assert-if-true`).

## isBackedEnum()

Type assertion,checks if a value is a `BackedEnum`:

```php
if (isBackedEnum($value)) {
    // $value is a BackedEnum
}
```

## isCellBag()

Type assertion,checks if a value is a `CellBag` instance:

```php
if (isCellBag($value)) {
    // $value is a CellBag
}
```

## Type assertion reference

| Helper | Narrowing to | PHPStan support |
|---|---|---|
| `isComponent()` | `BackendComponent` | `@phpstan-assert-if-true` |
| `isBackedEnum()` | `BackedEnum` | `@phpstan-assert-if-true` |
| `isCellBag()` | `CellBag` | `@phpstan-assert-if-true` |
