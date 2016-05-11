# Alex Price's SCSS Style Guide

This is my personal guide book to writing clean, modular SCSS. It is heavily influenced by the [Trello Style Guide](https://github.com/trello/trellisheets) and this document is a fork of it.

## Index

1. [Tools](#1-tools)
2. [Base Styles](#2-base-styles)
3. [Components](#3-components)
    - [Modifiers](#modifiers)
    - [State](#state)
    - [Media Queries](#media-queries)
    - [Keeping It Encapsulated](#keeping-it-encapsulated)
    - [Structure](#structure)
4. [Variables](#4-variables)
5. [JavaScript](#5-javascript)
6. [Mixins](#6-mixins)
7. [Utilities](#7-utilities)
8. [File Structure](#8-file-structure)
9. [Style](#9-style)
10. [Miscellany](#10-miscellany)


## 1. Tools

> Use only imports, variables, and mixins.

To keep our CSS readable, we try and keep our CSS very vanilla. We use SCSS, but only use imports, data-uri, variables, and some mixins. We use imports so that variables and mixins are available everywhere and it all outputs to a single file. We occasionally use nesting. We don’t use more complex functions like guards and loops.

Vendor prefixing is handled by our SCSS pre-processor. I reccomend `Gulp.js`.

## 2. Base Styles

> A rule applied to an element selector is a base rule. This include child selectors and pseudo-classes. It doesn't include ID selectors.

Base styles should be used very sparingly. A few good examples of base styles are:
- Heading sizes
- Default link styles
- Default font styles
- Body backgrounds

**Example**

``` SCSS
html, body {
  @include clearfix();
  box-sizing: border-box;
}

*, *:before, *:after {
  box-sizing: inherit;
  min-height: 0;
}
```

## 3. Components

> Use the `.component-descendant-descendant` pattern for components.

Components help encapsulate your CSS and prevent run-away cascading styles and keep things readable and maintainable. Central to componentising CSS is namespacing. Instead of using descendant selectors, like `.header img { … }`, you’ll create a new hyphen-separated class for the descendant element, like `.header-image { … }`.

``` SCSS
.global-header {
  background: $main-bg-colour;
  color: $white;
  height: 40px;
  padding: 10px;
}

  .global-header-logo {
    float: left;
  }

    .global-header-logo-image {
      background: url("logo.png");
      height: 40px;
      width: 200px;
    }

  .global-header-nav {
      float: right;
  }

    .global-header-nav-item {
      background: $green;
      border-radius: 3px;
      display: block;
      float: left;
      transition: background 100ms;
     }

    .global-header-nav-item:hover {
      background: darken($green, 10%);
    }
```

Namespacing keeps specificity low, which leads to fewer inline styles, !important declarations, and makes things more maintainable over time.

Make sure **every selector is a class**. There should be no reason to use id or element selectors. No underscores or camelCase. Everything should be lowercase.

Components make it easy to see relationships between classes. You just need to look at the name. You should still **indent descendant classes** so their relationship is even more obvious and it’s easier to scan the file. Stateful things like `:hover` should be on the same level.


### Modifiers

> Use the `.component-descendant.mod-modifier` pattern for modifier classes.

Let’s say you want to use a component, but style it in a special way. We run into a problem with namespacing because the class needs to be a sibling, not a child. Naming the selector `.component-descendant-modifier` means the modifier could be easily confused for a descendant. To denote that a class is a modifier, use a `.mod-modifier` class.

For example, we want to specially style our sign up button among the header buttons. We’ll add `.global-header-nav-item.mod-sign-up`, which looks like this:

``` HTML
<!-- HTML -->

<a class="global-header-nav-item mod-sign-up">
  Sign Up
</a>
```

``` SCSS
// _global-header.scss

.global-header-nav-item {
  background: $green;
  border-radius: 3px;
  display: block;
  float: left;
  transition: background 100ms;
}

.global-header-nav-item.mod-sign-up {
  background: $blue;
  color: $white;
}
```

We inherit all the `.global-header-nav-item` styles and modify it with `.mod-sign-up`. This breaks our namespace convention and increases the specificity, but that’s exactly what we want. This means we don’t have to worry about the order in the file. For the sake of clarity, put it after the part of the component it modifies. Put modifiers on the same indention level as the selector it’s modifying.

**You should never write a bare `.mod-` class**. It should always be tied to a part of a component. `.header-button.mod-sign-up { background: green; }` is good, but `.mod-sign-up { background: green; }` is bad. We could be using `.mod-sign-up` in another component and we wouldn’t want to override it.

You’ll often want to overwrite a descendant of the modified selector. Do that like so:

``` SCSS
.global-header-nav-item.mod-sign-up {
  background: $blue;
  color: #fff;

  .global-header-nav-item-text {
    font-weight: bold;
  }
}
```

Generally, we try and avoid nesting because it results in runaway rules that are impossible to read. This is an exception.

Put modifiers at the bottom of the component file, after the original components.


### State

> Use the `.component-descendant.is-state` pattern for state. Manipulate `.is-` classes in JavaScript (but not presentation classes).

State classes show that something is enabled, expanded, hidden, or what have you. For these classes, we’ll use a new `.component-descendant.is-state` pattern.

Example: Let’s say that when you click the logo, it goes back to your home page. But because it’s a single page app, it needs to load things. You want your logo to do a loading animation.

You’ll use a `.global-header-logo-image.is-loading` rule. That looks like this:

``` SCSS
.global-header-logo-image {
  background: url("logo.png");
  height: 40px;
  width: 200px;
}

.global-header-logo-image.is-loading {
  background: url("logo-loading.gif");
}
```

JavaScript defines the state of the application, so we’ll use JavaScript to toggle the state classes. The `.component.is-state` pattern decouples state and presentation concerns so we can add state classes without needing to know about the presentation class. A developer can just say to the designer, “This element has an .is-loading class. You can style it however you want.”. If the state class were something like `global-header-logo-image--is-loading`, the developer would have to know a lot about the presentation and it would be harder to update in the future.

Like modifiers, it’s possible that the same state class will be used on different components. You don’t want to override or inherit styles, so it’s important that **every component define its own styles for the state**. They should never be defined on their own. Meaning you should see `.global-header.is-hidden { display: none; }`, but never `.is-hidden { display: none; }` (as tempting as that may be). `.is-hidden` could conceivably mean different things in different components.

We also don’t indent state classes. Again, that’s only for descendants. State classes should appear at the bottom of the file, after the original components and modifiers.


### Media Queries

> Use media query variables in your component.

It might be tempting to add something like a `_mobile.scss` file that contains all your mobile-specific rules. We want to avoid global media queries and instead include them inside our components. This way when we update or delete a component, we’ll be less likely to forget about the media rules.

I recommend using a package like [Include Media](https://github.com/eduardoboucas/include-media) to handle your media queries. This will allow you to create clear, simple and maintainable styles for all browser varients.

**Example**

``` SCSS
// _sidebar.scss

.sidebar {
  width: 30%;
}

// Media Queries

@include media("<=phone") {
  .sidebar {
    display: none;
  }
}

@include media(">phone", "<=desktop") {
  .sidebar {
    width: 20%;
  }
}
```

Note that print is a media attribute, too. Keep your print rules inside components. We don’t want to forget about them either.

Put media rules at the bottom of the component file.


## Keeping It Encapsulated

Components can be large parts of the layout or just a button. In your templates, you'll likely end up with components inside each other, like a button inside a list.

Components shouldn’t know anything about each other and should be reusable in other places. Everything you need to know about the component should be in the file. Inversely, you shouldn’t overwrite or include component styles in other components. This has a lot of advantages:

1. You can see everything about the component in the file.
2. You don’t have to worry about other components overriding a style.
3. You can reuse the component elsewhere.
4. Components are small and readable.

-----

As an example, you should keep list and item components separate. For a list of boards, you’ll want to separate `_board-list.scss`, which defines the grid and layout, from `_board-tile.scss`, which defines the board styles within. With all the modifiers, states, and media queries, this keeps file size down, which in turn makes it more readable. This also allows us to reuse the board tile component elsewhere.

-----

Now a more complex example. You may reuse the `button` component inside the `member-list` component. We need to change the button’s size and positioning to fit the list. The smaller button can be reused in multiple places, so we’ll add a modifier in the button component (like, `.button.mod-small`), which we’ll use in member-list (and elsewhere). Now we do the positioning within the member list component, since that’s specific to the member list.

Here’s an example:

``` HTML
<!-- HTML -->

<div class="member-list">
  <div class="member-list-item">
    <p class="member-list-item-name">Gumby</p>
    <div class="member-list-item-action">
      <button class="button mod-small">Add</button>
    </div>
  </div>
</div>
```

``` SCSS
// _button.scss

.button {
  background: $white;
  border: 1px solid $grey;
  padding: 8px 12px;
}

.button.mod-small {
  padding: 6px 10px;
}


// _member-list.scss

.member-list {
  padding: 20px;
}

  .member-list-item {
    margin: 10px 0;
  }

    .member-list-item-name {
      font-weight: bold;
      margin: 0;
    }

    .member-list-item-action {
      float: right;
    }
```

A _bad, no good_ thing to do would be this:

``` HTML
<!-- HTML -->

<div class="member-list">
  <div class="member-list-item">
    <p class="member-list-item-name">Pat</p>
    <button class="button member-list-item-button">Add</button>
  </div>
</div>
```

``` SCSS
// _member-list.scss

.member-list-item-button {
  float: right;
  padding: 6px 10px;
}
```

In the _bad, no good_ example, `.member-list-item-button` overrides styles specific to the button component. It assumes things about button that it shouldn’t have to know anything about. It also prevents us from reusing the small button style and makes it hard to clean or change up later if needed.

You should end up with a lot of components. That’s encouraged. Always be asking yourself if everything inside a component is absolutely related and can’t be broken down into more components. If you start to have a lot of modifiers and descendants, it might be time to break it up. As a general rule, you should break up components that are **longer than 300 lines**.

## Structure

Structure your component like so…

1. The name, location, usage and notes in the comments
2. Base components with descendants
3. Modifiers (if any)
4. States (if any). States came after modifiers since you may need to overwrite styles.
5. Media Queries (if any)

An example:

``` SCSS
// Component (The name)

// Used in the main section of the app. (Where it’s used.)

// Usage:
// <div class="component">
//   <p class="component-decendant">
//     <span class="component-decendant-descendant"></span>
//   </p>
// </div>
//
// (How it's used.)

// Put any other notes here, too.

.component {
  /* … */
}
  
  .component-descendant {
    /* … */
  }

    .component-descendant-descendant {
      /* … */
    }


// Modifiers

.component.mod-small {
  /* … */
}


// State

.component.is-highlighted {
  /* … */
}

.component.mod-small.is-highlighted {
  /* … */
}


// Media Queries

@include media("<=phone") {
  .component {
    /* … */
  }
}


```

## 4. Variables

> Use variables sparingly.

We should try to avoid a heavy relience on variables. I recommend using them for breakpoint values, colours and maybe stuff like font-families if necessary. Substantial use of variables can make css hard to follow and cause you to have to repeatedly open your variables files, which gets old pretty quick.

### Colours

I like to have a `variables/_colours.scss` that contains:
- Named `rgba` functions for easy `alpha` value changes
- Named colour variables that represent those colours at 100% opacity
- Named brand variables that reference specific colour variables
 

**Example:**

``` SCSS
// _colours.scss

// Functions

@function dark-blue($opacity:1) {
    @return rgba(44, 62, 80, $opacity);
}

@function white($opacity:1) {
    @return rgba(236, 240, 241, $opacity);
}

// Colour variables

$dark-blue: dark-blue();
$white:     white();

// Brand colours

$brand-main-colour: $dark-blue;
$brand-highlight-colour: $white;
```

This allows us to reference brand colours in our SCSS while continuing to have easy access to specific colours if we need them. Meaning if we need to change the main brand colour to a pink, we can simply update the `$brand-main-colour` variable without changing all of our `$dark-blue` colours.

## 5. JavaScript

> Separate style and behavior concerns by using `.js-` prefixed classes for behavior.

For example:

``` HTML
<!-- HTML -->

<div class="content-nav">
  <a href="#" class="content-nav-button js-open-content-menu">
    Menu
  </a>
</div>
```

``` JavaScript
// JavaScript (with jQuery)

$(".js-open-content-menu").on('click', function(e) {
  openMenu();
});
```

Why do we want to do this? The `.js-` class makes it clear to the next person changing this template that it is being used for some JavaScript event and should be approached with caution.

Be sure to **use a descriptive class name**. The intent of `.js-open-content-menu` is more clear than `.js-menu`. A more descriptive class is less likely to conflict with other classes and it’s lots easier to search for. The class should almost always include a verb since it’s tied to an action.

**`.js-` classes should never appear in your stylesheets**. They are for JavaScript only. Inversely, there is never a reason to see presentation classes like `.header-nav-button` in JavaScript. You will see state classes like `.is-state` in your JavaScript and your stylesheets as `.component.is-state`.


## 6. Mixins

> Prefix mixins with `%m-` and only use them sparingly for shared styles.

Mixins are shared styles that are used in more than one component. To prevent standalone classes and use in markup, prefer the user of [SCSS placeholders](http://sass-lang.com/documentation/file.SASS_REFERENCE.html#placeholder_selectors_). They should be single level and contain no nesting. Mixins make things complicated fast, so **use sparingly**.

**Example usage:**

``` SCSS
// _mixins.scss

%m-list-divider {
  border-bottom: 1px solid $grey;
}

// _component.scss
.component-descendent {
  @extend %m-list-divider;
}

.component-descenden-2 {
  @extend %m-list-divider;
}
```


## 7. Utilities

> Prefix utility classes with `%u-`.

Sometimes we need a universal class that can be used in any component. Things like clear fixes, vertical alignment, and text truncation. Denote these classes by prefixing them with `%u-`. Again only use [SCSS placeholders](http://sass-lang.com/documentation/file.SASS_REFERENCE.html#placeholder_selectors_).

**For example:**

``` SCSS
%u-truncate-text {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

// _component.scss
.component-descendent {
  @extend %u-truncate-text;
}
```

A few utility rules:

1. No utility class should be so complex that it includes nesting styles.
2. Utilities should never be overwritten or included in components or mixins.
3. Use of utility classes should be limited. Include the styles in the components when possible. We don’t need something like `%u-float-left { float: left; }` where including `float: left;` in the component is more visible.
4. You should be able to fit all the utility classes in a single file.


## 8. File Structure

The file will look something like this:

``` SCSS
// ==============================================
// Style imports
// ==============================================

// Vendor
@import 'vendor/normalize';

// Variables
@import 'variables/media-queries';
@import 'variables/colours';
@import 'variables/other-variables-like-fonts';

// Mixins
@import 'mixins/clearfix';
@import 'mixins/other-mixins';

// Utils
@import 'utils/utils';

// Base
@import 'base/global';
@import 'base/typography';


// Components

// Board Components
@import 'components/board/board-component-1';
@import 'components/board/board-component-2';

// Header Components
@import 'components/header/header-component-1';
@import 'components/header/header-component-2'; // and so forth


```

Include [normalize.css](http://necolas.github.io/normalize.css/) at the top of the file. It standardizes CSS defaults across browsers. You should use it in all projects. Then include variables, mixins, and utils (respectively).

Then include the components. Each component should have its own file and include all the necessary modifiers, states, and media queries. If components are well encapsulated, the order should not matter. Break up components into logical folders by section.

This should output a single `main.css` file (or something similarly named).


## 9. Style

Even following the above guidelines, it’s still possible to write CSS in a ton of different ways. Writing our CSS in a consistent way makes it more readable for everyone. Take this bit of CSS:

``` SCSS
.global-header-nav-item {
  background: $grey;
  border-radius: 3px;
  display: block;
  float: left;
  padding: 8px 12px;
  transition: background 100ms;
}
```

It sticks to these style rules:

-	Use a new line for every selector and every declaration.
- Use two new lines between rules.
-	Add a single space between the property and value, for example `prop: value;` and not `prop:value;`.
-	Alphabetize declarations.
-	Use 2 spaces to indent, not 4 spaces and not tabs.
-	No underscores or camelCase for selectors.
- Use shorthand when appropriate, like `padding: 15px 0;` and not `padding: 15px 0px 15px 0px;`.
- No trailing whitespace.
- Keep line length under 80 characters.

Many of these are preferences, but standardizing makes reading code easier.


## 10. Miscellany

You might get the impression from this guide that our CSS is in great shape. That is not the case. While we’ve always stuck to .js classes and often use namespaced-component-looking classes, there is a mishmash of styles and patterns throughout. That’s okay. Going forward, you should rewrite sections according to these rules. Leave the place nicer than you found it.

Some additional things to keep in mind:

- Comments rarely hurt. If you find an answer on Stack Overflow or in a blog post, add the link to a comment so future people know what’s up. It’s good to explain the purpose of the file in a comment at the top.
- In your markup, order classes like so `<div class="component mod state js"></div>`.
- You can embed common images and files under 10kb using datauris. In the Trello web client, you can use `embed(/path/to/file)` to do this. This saves a request, but adds to the CSS size, so only use it on extremely common things like the logo.
- Avoid body classes. There is rarely a need for them. Stick to modifiers within your component.
- Explicitly write out class names in selectors. Don’t concatenate strings or use preprocessor trickery to build a class name. We want to be able to search for class names and that makes it impossible. This goes for `.js-` classes in JavaScript, too.
- If you are worried about long selector names making our CSS huge, don’t be. Compression makes this a moot point.
