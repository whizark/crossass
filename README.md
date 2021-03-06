# Crossass

A Sass mixin / function library, framework for modular CSS (SMACSS, OOCSS, BEM etc.).

**After I struggled with the issues Sass/CSS specification has, new design for Modular CSS comes to my mind,
which is more simple than Crossass but powerful & flexible.**

**That's just the [Xass](https://github.com/whizark/xass).**

**The naming is not so good... :p**
**However, the Xass way will be useful for designing CSS component/mixin/function with Sass, even if you won't use Xass.**

## Features

* Module definition
* Modifier support (also the nesting)
* Module exporting (with namespace / alias support)
* Module importing (with the modifiers / alias support)
* Modifier-level inheritance (from parent ruleset)
* Module-level inheritance (from parent module, **including the modifiers!**)
* Multiple inheritance
* Selectable inheritance
* Selector Helper
* BEM syntax support
* Variable inspection extension ([Crossass Inspector](https://github.com/whizark/crossass-inspector))
* [Live Templates](https://github.com/whizark/crossass-live-templates) for JetBrains IntelliJ IDEA family (PhpStorm, WebStorm etc.)
* No dependencies (except for Sass)

## Roadmap

1. *Development dropped*.
2. 0.9.2: Support Sass 3.3.0.rc.3+ (drop support prior to 3.3.0.rc.2)
3. 0.9.2: Add `x-class()` mixin to register class selector as parent context.
4. 0.9.x: Merge BEM extension to core.
5. Make `x-extend()` mixin to be more flexible, robust.
6. Almost done all I want to do.
7. Module hook to dynamically interrupt Module definition.
8. Automatic prefixing for Module name by the variable to configure.
   e.g. `@include x-module('module', true) { ... }` produces `.my-module { ... }` selector(s).
9. Module Mixin that imports other module and generates new selector(s) with mixin-ed Module's Modifier(s).
   e.g. `@include x-mixin('navbar')` in `header` module produces new `header__navbar`, `header__navbar__item` selectors.
10. Update document (code in Summary section is always the latest example).
11. Use more, test more.

## Summary

With Crossass, you can build modular CSS as below!

HTML:
```html
<div class="fancy-block">
    <div class="fancy-block-header">
        <img class="fancy-block-header-icon" src="/assets/images/icon.png" alt="Icon">
        Header
    </div>
    <div class="fancy-block-body">
        Body
    </div>
    <div class="fancy-block-footer">
        Footer
    </div>
</div>
```

SCSS:
```scss
// Module definition
@include x-module( 'block' ) {
    border: 1px solid #aaa;

    // Element definition using Modifier
    @include x-modifier( 'header', 'footer' ) {  // Multiple definitions at once
        padding: .5em;
        background: #ccc;
    }

    // Modifier produces `%<module name>-<modifier name>` selector(s)
    // `%block-body` in this context
    @include x-modifier( 'body' ) {
        padding: .5em;
    }
}

// 'icon-containable' module
@include x-module( 'icon-containable' ) {
    @include x-modifier( 'icon' ) {
        margin-right: 8px;
    }
}

// 'fancy-block' module with exporting using class selectors
@include x-module( 'fancy-block', true ) {
    // 'fancy-block' module extends 'block' module
    // Automatic Module-level inheritance, including the parent Module's Modifiers!
    @include x-extend( x-module( 'block' ) );

    // Partial overriding inherited properties from parent Module's root
    border-color: #f66;
    border-radius: 16px;
    overflow: hidden;

    // Partial Overriding inherited Modifiers from parent Module's Modifiers
    @include x-modifier( 'header', 'footer' ) {
        background: #faa;
    }

    @include x-modifier( 'header' ) {
        // Extending a module with its Modifiers
        @include x-extend( x-module( 'icon-containable' ) );
    }
}
```

CSS:
```css
.fancy-block {
  border: 1px solid #aaa;
}
.fancy-block-header {
  padding: .5em;
  background: #ccc;
}
.fancy-block-footer {
  padding: .5em;
  background: #ccc;
}
.fancy-block-body {
  padding: .5em;
}

.fancy-block-header-icon {
  margin-right: 8px;
}

.fancy-block {
  border-color: #f66;
  border-radius: 16px;
  overflow: hidden;
}
.fancy-block-header {
  background: #faa;
}
.fancy-block-footer {
  background: #faa;
}
```

The following is another example using almost all of the features.

SCSS:
```scss
// Module definition without exporting rulesets (Placeholder definition)
@include x-module( 'block' ) {
    border: 0 solid #aaa;

    // Element definition using Modifier
    @include x-modifier( 'header', 'footer' ) {  // Multiple definitions at once
        // Modifier-level inheritance (from the parent ruleset)
        // Inheritting `%block` in this context
        @include x-parent();

        padding: .5em;
        border-width: 1px;
    }

    @include x-modifier( 'colored' ) {
        background: #ccc;

        @include x-modifier( 'dark' ) {  // Nested Modifier for parent Modifier(s)
            color: #fff;
            background: #888;
        }
    }

    // Modifier produces `%<module name>-<modifier name>` selector(s)
    // `%block-body` in this context
    @include x-modifier( 'body' ) {
        // Modifier-level inheritance using x-parent() function
        // This is the same effect as `@include x-parent()`
        @include x-extend( x-parent() );

        padding: .5em;
        border-right-width: 1px;
        border-left-width: 1px;
    }
}
@include x-export( 'block' );    // Exporting a module using class selectors

// 'fancy-block' module
@include x-module( 'fancy-block', true ) { // Exporting with the module definition
    border: 4px solid #aaa;
    border-radius: 16px;
    overflow: hidden;

    @include x-modifier( 'header' ) {
        // Selectable and Multiple extending
        @include x-extend( (
            x-module( 'block' ): (                // From 'block' module,
                x-modifier( 'header', 'colored' ) // Inheriting 'header' and 'colored'
            )
        ) );

        // Partial overriding inherited property above
        border: none;
    }

    @include x-modifier( 'footer' ) {
        @include x-extend( (
            x-module( 'block' ): (
                x-modifier( 'footer', 'colored' )
            )
        ) );

        border: none;
    }

    @include x-modifier( 'colored' ) {
        @include x-extend( (
            x-module( 'block' ): ( // From 'block' module, inheriting same Modifier
                x-modifier()       // 'colored' in this context
            )
        ) );

        // You can also use interpolation as below
        // @include x-extend( #{x-module('block')}#{x-modifier()} );
    }

    @include x-modifier( 'body' ) {
        @include x-extend( '%block-body' );  // Extending `%block-body` like `@extend`

        border: none;
    }
}

// 'alert' module
@include x-module( 'alert', true ) {
    // 'alert' module extends 'fancy-block' module
    // Automatic Module-level inheritance, including the specified Module's Modifiers!
    @include x-extend( x-module( 'fancy-block' ) );

    // Partial overriding inherited properties from parent Module's root
    border-color: #f66;

    // Overriding inherited Modifiers from parent Module's Modifiers
    @include x-modifier( 'header', 'footer' ) {
        background: #faa;
    }
}

// Another way to extend other module with its modifiers
// 'block-margined' module extends 'block' module
@include x-module-extend( 'block-margined', 'block' ) {
    // Automatic Module-level inheritance, including the parent Module's Modifiers!

    margin: 1em;

    @include x-modifier( 'body' ) {
        &::before {
            // Referencing Module-root selector by `x-root()` function
            // Referencing Module name by `x-module-name()` function
            // Referencing Module parent's name by `x-module-parent()` function
            //
            // Note: You can use x-module-parent() only inside x-module-extend()
            //
            // `content: "block-margined (%block-margined) extends block."`;
            content: "#{x-module-name()} (#{x-root()}) extends #{x-module-parent()}.";
            display: block;
        }
    }
}

// Exporting a module as the alias with a namespace-scope
// 'block-margined' module is exported as '.block-editable' with '.wysiwyg' scope.
@include x-export( 'block-margined', 'wysiwyg', 'block-editable' );

// Importing a module as the alias into other rulesets
.preview {
    // This is the alias of
    // `@include x-export( 'block-margined', null, 'block-draggable' )`
    @include x-import( 'block-margined', 'block-draggable' );
}
```

Compiling it with Sass, you can get the clean output.

CSS:
```css
.block-header, .fancy-block-header, .alert-header, .wysiwyg .block-editable-header, .preview .block-draggable-header, .block-footer, .fancy-block-footer, .alert-footer, .wysiwyg .block-editable-footer, .preview .block-draggable-footer, .block-body, .fancy-block-body, .alert-body, .wysiwyg .block-editable-body, .preview .block-draggable-body, .block, .wysiwyg .block-editable, .preview .block-draggable {
  border: 0 solid #aaa;
}
.block-header, .fancy-block-header, .alert-header, .wysiwyg .block-editable-header, .preview .block-draggable-header {
  padding: .5em;
  border-width: 1px;
}
.block-footer, .fancy-block-footer, .alert-footer, .wysiwyg .block-editable-footer, .preview .block-draggable-footer {
  padding: .5em;
  border-width: 1px;
}
.block-colored, .fancy-block-header, .alert-header, .fancy-block-footer, .alert-footer, .fancy-block-colored, .alert-colored, .wysiwyg .block-editable-colored, .preview .block-draggable-colored {
  background: #ccc;
}
.block-colored-dark, .wysiwyg .block-editable-colored-dark, .preview .block-draggable-colored-dark {
  color: #fff;
  background: #888;
}
.block-body, .fancy-block-body, .alert-body, .wysiwyg .block-editable-body, .preview .block-draggable-body {
  padding: .5em;
  border-right-width: 1px;
  border-left-width: 1px;
}

.fancy-block, .alert {
  border: 4px solid #aaa;
  border-radius: 16px;
  overflow: hidden;
}
.fancy-block-header, .alert-header {
  border: none;
}
.fancy-block-footer, .alert-footer {
  border: none;
}
.fancy-block-body, .alert-body {
  border: none;
}

.alert {
  border-color: #f66;
}
.alert-header {
  background: #faa;
}
.alert-footer {
  background: #faa;
}

.wysiwyg .block-editable, .preview .block-draggable {
  margin: 1em;
}
.wysiwyg .block-editable-body::before, .preview .block-draggable-body::before {
  content: "block-margined (%block-margined) extends block.";
  display: block;
}
```

## Background

To build moudlar CSS, you know there are some popular methodologies like [OOCSS](http://oocss.org/) / [SMACSS](http://smacss.com/) / [BEM](http://bem.info/) etc.
In [Sass](http://sass-lang.com/) 3.0.0+, we can build CSS using BEM syntax like the following.

SCSS:
```scss
.nav {    // Module
    @at-root {
        & {
            // ...
        }
        #{&}__menu {    // Module element
            // ...
        }
        #{&}--global {    // Module variant
            // ...
        }
    }
}
```

CSS:
```css
.nav {
    // ...
}
.nav__menu {
    // ...
}
.nav--global {
    // ...
}
```

It looks like nice but you will find out there are still a few issues.
For example, how can I add a child element into the module variant `.nav--global` ? Like this?

SCSS:
```scss
.nav {    // Module
    @at-root {
        & {
            // ...
        }
        #{&}__menu {    // Module element
            // ...
        }
        #{&}--global {    // Module variant
            // ...
            #{&}__menu {    // Module variant's element (?)
                // ...
            }
        }
    }
}
```

Actually, this works.

CSS:
```css
.nav {
    // ...
}
.nav__menu {
    // ...
}
.nav--global {
    // ...
}
.nav--global .nav--global__menu {
    // ...
}
```

However, it's bad for the eyes and it might also cause CSS specificity-related issue near the future for the `.nav--global .nav--global__menu`.

By using `@at-root` inside the child selector, this issue can also be solved.

SCSS:
```scss
.nav {    // Module
    @at-root {
        & {
            // ...
        }
        #{&}__menu {    // Module element
            // ...
        }
        #{&}--global {    // Module variant
            // ...
            @at-root {
                #{&}__menu {    // Module variant's element
                    // ...
                }
            }
        }
    }
}
```

CSS:
```css
.nav {
    // ...
}
.nav__menu {
    // ...
}
.nav--global {
    // ...
}
.nav--global__menu {
    // ...
}
```

Although you can do this, you have to repeatedly define the declarations for the elements
and the source code will get to be unreadable because of the nested module variant(s).

As another solution, you might come up with using `@extend`.

SCSS:
```scss
.nav {    // Module
    @at-root {
        & {
            // ...
        }
        #{&}__menu {    // Module element
            // ...
        }
    }
}
.nav--global {    // Module variant
    @extend .nav;
}
```

Unfortunately, this SCSS produces a sad result.

CSS:
```css
.nav, .nav--global {
    // ...
}
.nav__menu {
    // ...
}
```

Where is the module variant's element `.nav--global__menu` ...?

You should put the declarations for the element into `.nav__menu` together...?
Although it might not be a big issue while the module variant's element is the same as the parent module's element,
it will break up the principle of the modular CSS.
To extend parent module's elements, you hava to use `@extend` for the each element.

Crossass aims to be a Sass mixin / function library for easily building modular CSS.

##Usage

### Requirement

Crossass is a pure Sass mixin / function library, so really portable.

* [Sass](http://sass-lang.com/) 3.3.0.rc.1 - 3.3.0.rc.2

### Import

```scss
@import "crossass/scss/core";
```

### Module (Placeholder definition)

`x-module` mixin produces ruleset(s) that use [Placeholder Selector](http://sass-lang.com/documentation/file.SASS_REFERENCE.html#placeholder_selectors_).

SCSS:
```scss
@include x-module('module') {
  border: 1px solid #000;
  strong {
    color: #f00;
  }
}
.module {
  @extend %module;
}
```

CSS:
```css
.module {
    border: 1px solid #f00;
}
.module strong {
    color: #f00;
}
```

### Module with class exporting

By passing `true` to 2nd parameter, you can also export ruleset(s) that use class selector so you don't have to use `@extend` to export them.

SCSS:
```scss
@include x-module('module', true) {
  border: 1px solid #000;
  strong {
    color: #f00;
  }
}
```

CSS:
```css
.module {
    border: 1px solid #f00;
}
.module strong {
    color: #f00;
}
```

### Exporting

You may use `x-export` mixin to export a module.

SCSS:
```scss
@include x-module('module') {
  border: 1px solid #000;
  strong {
    color: #f00;
  }
}
@include x-export('module');
```

CSS:
```css
.module {
    border: 1px solid #f00;
}
.module strong {
    color: #f00;
}
```

### Modifier

`x-modifier` mixin generates a suffixed selector.
The default modifier pattern is `$x-modifier-separator` + the modifier name and the default `$x-modifier-separator` is `-` (see also [Variables](#variables)).

SCSS:
```scss
@include x-module('module', true) {
  border: 1px solid #000;
  @include x-modifier('important') {
    border-color: #f00;
  }
}
```

CSS:
```css
.module {
    border: 1px solid #f00;
}
.module-important {
    border-color: #f00;
}
```

You can also define multiple selectors at once.

SCSS:
```scss
@include x-module('module', true) {
  border: 1px solid #000;
  @include x-modifier('information', 'important') {
    margin: 8px;
  }
}
```

CSS:
```css
.module {
    border: 1px solid #f00;
}
.module-information {
    margin: 8px;
}
.module-important {
    margin: 8px;
}
```

### Modifier (nesting)

`x-modifier` mixin also allows you to nest the declarations.

SCSS:
```scss
@include x-module('module', true) {
  border: 1px solid #000;
  @include x-modifier('alert') {
    border-color: #fa0;
    @include x-modifier('important') {
      border-color: #f00;
    }
  }
}
```

CSS:
```css
.module {
    border: 1px solid #f00;
}
.module-alert {
    border-color: #fa0;
}
.module-alert-important {
    border-color: #f00;
}
```

### Inheritance from parent ruleset

You can easily extend parent ruleset by using `x-parent` mixin.

SCSS:
```scss
@include x-module('alert', true) {
  border-width: 1px;
  border-style: solid;

  @include x-modifier('important') {
    @include x-parent();
    border-color: #fa0;
    font-weight: bold;

    @include x-modifier('danger') {
      @include x-parent();
      border-color: #f00;
    }
  }
}
```

CSS:
```css
.alert, .alert-important, .alert-important-danger {
  border-width: 1px;
  border-style: solid;
}
.alert-important, .alert-important-danger {
  border-color: #fa0;
  font-weight: bold;
}
.alert-important-danger {
  border-color: #f00;
}
```

### Sub Module (Module-level extending)

By using `x-module-extend` mixin, you can define sub module based on another module.
The sub module inherits parent module's rulesets including the modifiers!

SCSS:
```scss
@include x-module('parent', true) {
  border: 1px solid #000;

  @include x-modifier('modifier') {
    background: #fff;
  }
}
@include x-module-extend('child', 'parent', true) {
}
```

CSS:
```css
.child {
    border: 1px solid #000;
}
.child-modifier {
    background: #fff;
}
.parent {
    border: 1px solid #000;
}
.parent-modifier {
    background: #fff;
}
```

### Extending

`x-extend` mixin is similar with `@extend`.

SCSS:
```scss
@include x-module('module') {
  border: 1px solid #f00;
}
.module {
  @include x-extend('%module');
}
```

CSS:
```css
.module {
    border: 1px solid #f00;
}
```

### Extending (overriding)

You might override some declarations inside `x-extend` mixin.

SCSS:
```scss
@include x-module('module') {
  border: 1px solid #f00;
}
.module {
  @include x-extend('%module') {
    border-width: 2px;
  }
}
```

CSS:
```css
.module {
    border: 1px solid #f00;
}
.module {
    border-width: 2px;
}
```

### Extending (multiple inheritance)

`x-extend` mixin allows you to extend a ruleset from multiple rulesets.

SCSS:
```scss
@include x-module('module-1') {
  border: 1px solid #f00;
}
@include x-module('module-2') {
  background: #000;
}
.module {
  @include x-extend('%module-1', '%module-2');
}
```

CSS:
```css
.module {
    border: 1px solid #f00;
}
.module {
    background: #000;
}
```

You cannot specify the order of multiple inheritance and **Sass outputs rulesets in order of the definition**.
This is a restriction of Sass so **the order of ruleset definition is very important** with Crossass.

SCSS:
```scss
@include x-module('module-1') {
  border: 1px solid #f00;
  background: #f00;
}
@include x-module('module-2') {
  background: #000;
}
.module {
  @include x-extend('%module-1', '%module-2');
}
```

CSS:
```css
.module {
    border: 1px solid #f00;
    background: #f00;
}
.module {
    background: #000;
}
```

### Extending with specific module element(s)

You can extend a ruleset with specific module element at once.

SCSS:
```scss
@include x-module('block') {
  border: 1px solid #000;
  @include x-modifier('header') {
    font-size: 1.5em;
  }
  @include x-modifier('body') {
    margin: 1em;
  }
  @include x-modifier('footer') {
    background: #ccc;
  }
}
@include x-module('alert', true) {
  @include x-extend((
    '%block': ('-header', '-body')
  ));
  background: #f00;
}
```

CSS:
```css
.alert {
    border: 1px solid #000;
}
.alert-header {
    font-size: 1.5em;
}
.alert-body {
    margin: 1em;
}
.alert {
    background: #f00;
}
```

You can still use multiple inheritance.

SCSS:
```scss
@include x-module('block-1') {
  border: 1px solid #000;
  @include x-modifier('header') {
    font-size: 1.5em;
  }
  @include x-modifier('body') {
    margin: 1em;
  }
  @include x-modifier('footer') {
    background: #ccc;
  }
}
@include x-module('block-2') {
    @include x-modifier('footer') {
        font-size: .75em;
    }
}
@include x-module('alert', true) {
  @include x-extend((
    block-1: ('-header', '-body'),
    block-2: ('-footer'),
  ));
  background: #f00;
}
```

CSS:
```css
.alert {
    border: 1px solid #000;
}
.alert-header {
    font-size: 1.5em;
}
.alert-body {
    margin: 1em;
}
.alert-footer {
    font-size: .75em;
}
.alert {
    background: #f00;
}
```

### Selector Helper

`x-modifier` function is useful if you want to generate the selector for a module element.
This function returns the selector that is `$x-modifier-separator` + a modifier name (see also [Variables](#variables)).

SCSS:
```scss
@include x-module('block') {
  border: 1px solid #000;
  @include x-modifier('body') {
    margin: 1em;
  }
}
@include x-module('alert', true) {
  @include x-extend((
    x-module('block'): (x-modifier('body'))
  ));
  background: #f00;
}
```

CSS:
```css
.alert {
    border: 1px solid #000;
}
.alert-body {
    margin: 1em;
}
.alert {
    background: #f00;
}
```

`x-modifier` function can generate multiple selectors at once.
In addition, you can also use multiple `x-modifier` function in `x-extend` mixin.

SCSS:
```scss
@include x-module('block') {
  border: 1px solid #000;
  @include x-modifier('header') {
    font-size: 1.5em;
  }
  @include x-modifier('body') {
    margin: 1em;
  }
  @include x-modifier('footer') {
    background: #ccc;
  }
}
@include x-module('alert', true) {
  @include x-extend((
    x-module('block'): (
      x-modifier('header', 'body'),
      x-modifier('footer')
    )
  ));
  background: #f00;
}
```

CSS:
```css
.alert {
    border: 1px solid #000;
}
.alert-header {
    font-size: 1.5em;
}
.alert-body {
    margin: 1em;
}
.alert-footer {
    background: #ccc;
}
.alert {
    background: #f00;
}
```

### Variables

Variable | Default | Description
--- | --- | ---
$x-module-exporting | false | Whether `x-module` mixin should export ruleset(s) that use class selector
$x-modifier-separator | '-' | The separator for modifiers
$x-strict-extend | false | `x-extend` mixin never uses `!optional` if true

## BEM syntax support

Crossass also supports [BEM](http://bem.info/) syntax with BEM extension.

### Import

```scss
@import "crossass/scss/core";
@import "crossass/scss/extension/bem";
```

### Mixins

* `x-bem-modifier`
* `x-bem-element`

The usage is the same as `x-modifier` mixin.

### Selector Helper

* `x-bem-modifier`
* `x-bem-element`

The usage is the same as `x-modifier` function.

### Variables

Variable | Default | Description
--- | --- | ---
$x-bem-modifier-separator | '--' | The separator for modifiers
$x-bem-element-separator | '__' | `The separator for elements
