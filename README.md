<p align="center"><img src="https://raw.githubusercontent.com/HughxDev/standard-standards/master/standard-standards-logo.svg?sanitize=true" width="150" alt="logo" /></p>

<h1 align="center">Standard Standards</h1>

<p align="center">Boilerplate recommendations for team coding practices, to aid maintenance of large projects. Currently (and perhaps perpetually) in draft status—everything is subject to change as new best practices emerge and prior recommendations are rethought.</p>

----

## General Naming Conventions

### DAMP (Descriptive And Meaningful Phrases).

Prefer readability for other developers over less typing for yourself. Abbreviations that have become standalone words like `param` and `config` are OK, as are counters like `i` and `j`. But otherwise, aim to decrease the cognitive load for reading your code, e.g. `event` over `e`, `error` over `err`, `populationData` over `data`.

#### Examples

##### HTML/CSS:

```html
<h2 class="sh">Section Header</h2>
<!-- Bad -->
<h2 class="section-header">Section Header</h2>
<!-- Good -->
```

##### JavaScript:

```js
const newElCmpShrtNm = "Header"; // Bad
const newElementComponentShortName = "Header"; // Good
```

## Programming Conventions

Consistent and readable JavaScript formatting is enforced by [`eslint-config-hughx`](https://github.com/hguiney/eslint-config-hughx) + an ESLint auto-formatter of your choice, such as [ESLint for VS Code](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint).

### Functional Programming

Use Functional Programming principals as often as possible to aid maintainability and predictability. The basic idea is for every function to produce the same output for a given set of inputs regardless of when/where/how often they are called. This means a preference for functions taking their values from explicit parameters as opposed to reading variables from the surrounding scope. Additionally, a function should not produce side-effects by e.g. changing the value of a variable in the surrounding scope.

### Directory Layout

- `public/`: Static files such as images, favicon, etc. that are copied to the server root on deploy.
- `src/`: React source.
  - `components/`: React components.
  - `data/`: Flat-file JSON database.
  - `util/`: Utility functions.
  - `index.js`: React entrypoint.
- `.env`: [Dotenv](https://github.com/motdotla/dotenv#readme) configuration.
- `.eslintrc.js`: [ESLint](https://eslint.org/) configuration.
- `.gitignore`: List of files to exclude from Git commits.
- `package.json`: Project metadata/NPM dependencies.
- `README.md`: Project documentation (this file).
- `yarn.lock`: Yarn lockfile.

### Component Layout

Every React component consists of the following structure:

- `Component/`
  - `__tests__`: Integration tests (optional)
  - `Component.css`: CSS styling
  - `Component.test.js`: Unit test
  - `index.js`: React component
  - `methods.js`: Any methods that don’t need to go in the render function, for tidiness. (optional)

## CSS Conventions

### BEM Methodology

[Mostly] Vanilla BEM (Block-Element-Modifier):

- Blocks: Lowercase name (`block`)
- Elements: two underscores appended to block (`block__element`)
- Modifiers: two dashes appended to block or element (`block--modifier`, `block__element--modifier`).

When writing modifiers, ensure the base class is also present; modifiers should not mean anything on their own. This also gives modifiers higher specificity than regular classes, which helps ensure that they actually get applied.

```scss
.block--modifier {
} // Bad
.block.block--modifier {
} // Good
```

<b>⚠️ Warning</b>: This is a departure from vanilla BEM. Rationale follows; may want to revisit.

<details>
<summary>Rationale</summary>
In vanilla BEM, the recommendation is to always use modifier classes as independent selectors, like so:

```scss
.alert {
  background-color: white;
}
.alert--error {
  background-color: red;
}
```

However, this introduces the possibility of overriding one modifier with another depending on where in the cascade you define the modifier:

```scss
.alert {
  background-color: white;
  border-radius: 0;
}
.alert--error {
  background-color: red;
}
.alert--alt {
  background-color: green;
  border-radius: 3px;
}
```

This CSS would cause the following to display as `green` even though `alert--error` is defined last in the `classList`:

```html
<div class="alert alert--alt alert--error"></div>
```

To get around this, a rule can be implemented to always use a single modifier class, combining disparate modifiers into a combined modifier, such as `alert--alt-error`:

```scss
.alert {
  background-color: white;
  border-radius: 0;
}
.alert--error {
  background-color: red;
}
.alert--alt {
  background-color: green;
  border-radius: 3px;
}
.alert--alt-error {
  background-color: red;
  border-radius: 3px;
}
```

However, this allows for `alert--error`, `alert--alt`, and `alert--alt-error` to be placed on any element, including non-`alert`s, and still have their styles applied, as in `<div class="button alert--alt">`. A modifier should never be used in this way as it means nothing on its own; its definition is in relation to the thing being modified. Additionally, any styles common to `alert--alt` and `alert--alt-error` have to be repeated, unless rewritten like this:

```scss
.alert--alt,
.alert--alt-error
// or [class^="alert--alt"]
{
  border-radius: 3px;
}
.alert--alt {
  background-color: green;
}
.alert--alt-error {
  background-color: red;
}
```

This is workable, but requires shifting code around as new additions are made. The recommended alternative, then, is to ensure the base class is always present in modifier selectors, in a combined class selector:

```scss
.alert.alert--alt {}
```

While vanilla BEM sees this higher specifity as undesirable, since later classes will fail in instances like this:

```html
<div class="alert alert--alt alert--error"></div>
```
```scss
.alert {}
.alert.alert--alt {}
.alert--error {}
```

…this is a bit contrived as the combined selector practice requires the following, which preserves specificity parity:

```scss
.alert {}
.alert.alert--alt {}
.alert.alert--error {}
```

Then, if you needed overrides, rather than creating a whole new class, you can just do this:

```scss
.alert {}
.alert.alert--error {}

.alert.alert--alt {}
.alert.alert--alt.alert--error {}
```

It’s more verbose but it also allows you to be clear about what you want to happen when an element has multiple modifiers that would otherwise compete with each other.

Alternatively, you could keep component state out of CSS classes altogether, limiting them to `aria-*` or `data-*` attributes, which communicate a semantics of not being purely cosmetic, and may be more intuitive to navigate in JavaScript:

```scss
.alert {}
.alert[data-has-error] {}

.alert.alert--alt {}
.alert.alert--alt[data-has-error] {}
```
```js
$alert.className = $alert.className.replace( /\balert--error\b/, '' );
// or
$alert.classList.remove( 'alert--error' );

// vs.
$alert.dataset.hasError = false;
```
</details>

An exception to this would be for mixin classes that are intended to be used broadly. For example, responsive utilities to show/hide elements at different breakpoints:

```scss
.--hide-until-large {
  display: none;
}
@media screen and (min-width: $large) {
  .--hide-until-large {
    display: inline-block; // IE/Edge compat
    display: unset;
  }
}
```

Don’t reflect the expected DOM structure in class names, as this expectation is likely to break as projects evolve. Only indicate which block owns the element. This allows components to be transposable and avoids extremely long class names.

```scss
.block__element__subelement {
} // Bad
.block__subelement {
} // Good
```