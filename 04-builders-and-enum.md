# Builders and Enum

## ComponentEnum

`ComponentEnum` is a backed string enum that catalogs every available component. Each case maps to a Blade view path under `resources/views/components/`.

### Full catalog

| Category | Case | View path |
|---|---|---|
| **Template** | `TEMPLATE` | `template` |
| **Collection** | `COLLECTION` | `collection` |
| **Block** | `DIV` | `div` |
| | `PARAGRAPH` | `paragraph` |
| **Inline** | `BUTTON` | `inline.button` |
| | `LINK` | `inline.link` |
| | `IMG` | `inline.img` |
| | `SPAN` | `inline.span` |
| | `BOLD` | `inline.bold` |
| | `EM` | `inline.em` |
| | `ITALIC` | `inline.italic` |
| | `STRONG` | `inline.strong` |
| | `SMALL` | `inline.small` |
| **Headers** | `H1`–`H6` | `headers.h1`–`headers.h6` |
| **Form** | `FORM` | `form.form` |
| | `LABEL` | `form.label` |
| | `LEGEND` | `form.legend` |
| | `FIELDSET` | `form.fieldset` |
| | `TEXT_INPUT` | `form.text` |
| | `FILE_INPUT` | `form.file` |
| | `EMAIL_INPUT` | `form.email` |
| | `SEARCH_INPUT` | `form.search` |
| | `PASSWORD_INPUT` | `form.password` |
| | `CHECKBOX_INPUT` | `form.checkbox` |
| | `HIDDEN_INPUT` | `form.hidden` |
| | `RADIO_INPUT` | `form.radio` |
| | `NUMBER_INPUT` | `form.number` |
| | `URL_INPUT` | `form.url` |
| | `TEL_INPUT` | `form.tel` |
| | `DATE_INPUT` | `form.date` |
| | `TIME_INPUT` | `form.time` |
| | `COLOR_INPUT` | `form.color` |
| | `RANGE_INPUT` | `form.range` |
| | `DATALIST` | `form.datalist` |
| | `TEXTAREA` | `form.textarea` |
| | `SELECT` | `form.select` |
| | `OPTGROUP` | `form.optgroup` |
| | `OPTION` | `form.option` |
| **Table** | `TABLE` | `table.table` |
| | `THEAD` | `table.thead` |
| | `TBODY` | `table.tbody` |
| | `TFOOT` | `table.tfoot` |
| | `TR` | `table.tr` |
| | `TH` | `table.th` |
| | `TD` | `table.td` |
| | `CAPTION` | `table.caption` |
| | `COLGROUP` | `table.colgroup` |
| | `COL` | `table.col` |
| **Lists** | `OL` | `lists.ol` |
| | `UL` | `lists.ul` |
| | `LI` | `lists.li` |
| **Details** | `DETAILS` | `details.details` |
| | `SUMMARY` | `details.summary` |
| **Layers** | `DIALOG` | `layers.dialog` |

## Builders

There are three builders that control which `resources/views/` directory resolves components and themes.

### ComponentBuilder

Resolves both components and themes from the package's own `resources/views/`:

```php
use Juaniquillo\BackendComponents\Builders\ComponentBuilder;
use Juaniquillo\BackendComponents\Enums\ComponentEnum;

$button = ComponentBuilder::make(ComponentEnum::BUTTON);
$div = ComponentBuilder::make('div');
```

### LocalComponentBuilder

Resolves both components and themes from the consuming application's `resources/views/`:

```php
use Juaniquillo\BackendComponents\Builders\LocalComponentBuilder;

$button = LocalComponentBuilder::make(ComponentEnum::BUTTON);
```

This lets an app override any component and any theme file locally.

### LocalThemeComponentBuilder

Resolves components from the package, but themes from the app's local views:

```php
use Juaniquillo\BackendComponents\Builders\LocalThemeComponentBuilder;

$button = LocalThemeComponentBuilder::make(ComponentEnum::BUTTON);
```

Useful when you only want to customize themes without duplicating component templates.

### useLocal()

`ComponentBuilder::make()->useLocal()` is shorthand equivalent to `LocalComponentBuilder`:

```php
$button = ComponentBuilder::make(ComponentEnum::BUTTON)
    ->useLocal();
```

### Comparison

| Builder | Component path | Theme path |
|---|---|---|
| `ComponentBuilder` | package `resources/views/components/` | package `resources/views/_themes/tailwind/` |
| `LocalComponentBuilder` | app `resources/views/components/` | app `resources/views/_themes/tailwind/` |
| `LocalThemeComponentBuilder` | package `resources/views/components/` | app `resources/views/_themes/tailwind/` |

## StaticBuilder interface

All three builders implement `StaticBuilder`:

```php
interface StaticBuilder
{
    public static function make(string|ComponentEnum $name): Htmlable|CompoundComponent;
}
```

## Custom paths and namespaces

You can override paths and namespaces on any component:

```php
$component = ComponentBuilder::make('img')
    ->setPath('inline')                           // sets the view path prefix
    ->setNamespace('custom-namespace::');          // overrides the view namespace
```
