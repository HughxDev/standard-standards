# Standard Standards

Boilerplate recommendations for team coding practices, to aid maintenance of large projects.

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

Vanilla BEM (Block-Element-Modifier):

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