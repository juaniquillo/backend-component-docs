# Notes

## Creator's Notes

You might think I don't like HTML or CSS. On the contrary,I love them both. I've been working with these technologies for a long time, and I particularly enjoy semantic HTML and well-structured, reusable CSS.

But there are times when I wish I could manage complex HTML on the server side. Complex tables and forms with multiple conditionals and access policies, for example. Server-side control also makes it convenient to resolve and cache static content. That's why this package doesn't include every HTML element by default,for layout, it's easier to work directly in HTML or Blade.

The package's main goals are **simplicity**, **predictability**, and **composability**. Think of the components as primitives: use them as-is, or build higher-level abstractions on top of them. I referenced [MDN's HTML Basics](https://developer.mozilla.org/en-US/docs/Learn/HTML/Introduction_to_HTML/HTML_basics) and [Laravel's component documentation](https://laravel.com/docs/blade#components) to ensure consistency.

## Architecture

The package follows a **trait-per-concern** pattern. `MainBackendComponent` composes 5 traits, each responsible for a single aspect:

| Trait | Purpose |
|---|---|
| `HasContent` | Content array management (`setContent`, `setContents($overwrite)`, `prependContent`, `unsetContent`) |
| `HasPath` | View namespace and component path resolution (`getName`, `useLocal`, `getComponentPath`) |
| `HasSettings` | Key-value settings bag (`setSetting`, `getSetting`, `setSettings`, `unsetSetting`) |
| `IsBackendComponent` | HTML attributes (`setAttribute`, `setAttributes`, `getAttributes`) |
| `IsThemeable` | Theme management (`setTheme`, `setThemes`, `getTheme`, `compileTheme`) |
| `IsLivewireComponent` | Livewire integration (`setLivewire`, `setLivewireKey`, `setLivewireParams`) |

Plus an `isFactory` trait on `ComponentFactory` for deserialization from arrays.

## Code style conventions

- **PHP 8.3+** with `declare(strict_types=1)` on every file
- **PSR-4** autoloading: `Juaniquillo\BackendComponents` → `src/`
- **Fluent return types**,all setters return `: static`
- **Generics in docblocks**,`@param` and `@return` documented for all array types
- **Laravel Pint** for code style (PSR-12 based)
- **PHPStan level 8** for static analysis
- **No public properties**,all component state is managed through methods

## Adding a new component

1. **Add the enum case** in `src/Enums/ComponentEnum.php`:
   ```php
   case NEW_COMPONENT = 'path.to.component';
   ```

2. **Create the Blade template** at `resources/views/components/path/to/component.blade.php`:
   ```blade
   @props(['attrs' => null])
   @php
       $serverAttrs = [];
       $content = null;
       $slot = $slot ?? null;
       if ($attrs) {
           $serverAttrs = $attrs->getAttributes();
           $content = $attrs->content;
       }
   @endphp
   <your-element {{ $attributes->merge($serverAttrs) }}>{{ $content }}{{ $slot }}</your-element>
   ```

3. **Write a feature test** at `tests/Feature/Components/{Category}/{Name}Test.php` following the standard test methods (see Testing section below).

## Testing

### Running tests

```bash
composer test        # Run PHPUnit with warnings
composer qa          # Pint + PHPStan + PHPUnit (CI gate)
vendor/bin/pint      # Code style fix
vendor/bin/phpstan   # Static analysis (level 8)
```

### Test conventions

- Feature tests extend `Tests\TestCase` (Orchestra Testbench-based)
- Use `#[Test]` attribute (not `@test` docblock)
- Render components with `$this->blade('{{ $component }}', ['component' => $component])`
- Assert output with `assertSee()` / `assertDontSee()`

### Standard test methods

Every component feature test includes these standard methods:

```php
#[Test]
public function empty_component()
{
    $component = ComponentBuilder::make(ComponentEnum::YOUR_ENUM);

    $this->blade('{{ $component }}', ['component' => $component])
        ->assertSee('<your-element', false)
        ->assertSee('</your-element>', false);
}

#[Test]
public function component_accepts_content()
{
    $component = ComponentBuilder::make(ComponentEnum::YOUR_ENUM)
        ->setContent('Some text');

    $this->blade('{{ $component }}', ['component' => $component])
        ->assertSee('Some text');
}

#[Test]
public function component_accepts_contents_array()
{
    $component = ComponentBuilder::make(ComponentEnum::YOUR_ENUM)
        ->setContents([
            ComponentBuilder::make(ComponentEnum::SPAN)->setContent('Item 1'),
            ComponentBuilder::make(ComponentEnum::SPAN)->setContent('Item 2'),
        ]);

    $this->blade('{{ $component }}', ['component' => $component])
        ->assertSee('Item 1')
        ->assertSee('Item 2');
}

#[Test]
public function component_accepts_attributes()
{
    $component = ComponentBuilder::make(ComponentEnum::YOUR_ENUM)
        ->setAttribute('id', 'test-id');

    $this->blade('{{ $component }}', ['component' => $component])
        ->assertSee('id="test-id"', false);
}

#[Test]
public function component_accepts_theme()
{
    $theme = ['display' => 'inline-block'];
    $component = ComponentBuilder::make(ComponentEnum::YOUR_ENUM)
        ->setThemes($theme);

    $this->blade('{{ $component }}', ['component' => $component])
        ->assertSee('class="'.processThemes($theme), false);
}
```

## CI

The package runs three checks on pull requests (via GitHub Actions):

1. **Pint**,PHP code style fixer
2. **PHPStan**,static analysis at level 8
3. **PHPUnit**,feature and unit tests

Run `composer qa` locally to verify all three before submitting a PR.

## Package discovery

The service provider (`BackendComponentsServiceProvider`) uses `spatie/laravel-package-tools` and is auto-discovered by Laravel. It registers:

- The `backend-component::` view namespace
- Namespaced helper functions via `include_once 'helpers.php'`

## Config

The config file `config/backend-component.php` can be published with:

```bash
php artisan vendor:publish --tag=backend-component-config
```

It currently returns an empty array and is reserved for future use.
