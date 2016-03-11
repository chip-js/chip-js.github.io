---
layout: default
---


# Fragments.js built-ins

## Built-in Binders

These binders are a collection of common binders that many frameworks would use. Below are some examples of how you can
use these binders.

## Example Uses

Fragments allows you to create binders in many ways enabling you to create your own syntax or style of templating. Here
are some examples of existing framework that could be emulated using Fragments.js and these built-in binders.

### Angular 1.0

```js
var fragments = require('fragments-js');

// Set the attribute delimiters to none
fragments.setExpressionDelimiters('attribute', '', '');

// Set up formatters
fragments.registerFormatter('filter', require('fragments-built-ins/formatters/filter'));
fragments.registerFormatter('orderBy', require('fragments-built-ins/formatters/sort'));

// Set up binders
fragments.registerAttribute('ng-repeat', require('fragments-built-ins/binders/repeat')());
fragments.registerAttribute('ng-model', require('fragments-built-ins/binders/value')());
fragments.registerAttribute('ng-click', require('fragments-built-ins/binders/events')('click'));
fragments.registerAttribute('ng-show', require('fragments-built-ins/binders/show')(true));
fragments.registerAttribute('ng-hide', require('fragments-built-ins/binders/show')(false));
fragments.registerAttribute('ng-src', require('fragments-built-ins/binders/attributes')('src'));
```

```html
Phone: {{phone.name}}
<ul class="phone-thumbs">
  <li ng-repeat="img in phone.images | filter(currentPhoneFilter)">
    <img ng-src="img" ng-click="setImage(img)">
  </li>
</ul>
```

We aren't able to get to exactly the same syntax in every case, but you will see it is close. There are two ways we are
not able to exactly duplicate Angular 1.0 syntax. First, `ng-src` remains consistent with the other `ng-*` attributes
and doesn't use curly-braces. But in Angular 1 it is one of the only (if not only) `ng-*` directives that does use
curly-braces. Second, formatters in both Angular 1 and 2 use colons to separate their arguments like
`img in phone.images | filter:currentPhoneFilter`, but fragments uses the parenthesis and commas like a function call.
If there is demand for it, perhaps we will allow this to be customizable.

### Angular 2.0

```js
var fragments = require('fragments-js');

// Set the attribute delimiters to none
fragments.setExpressionDelimiters('attribute', '', '');

// Set up binders
fragments.registerAttribute('ng-for', require('fragments-built-ins/binders/repeat')());
fragments.registerAttribute('ng-if', require('fragments-built-ins/binders/if')());
fragments.registerAttribute('(*)', require('fragments-built-ins/binders/key-events')());
fragments.registerAttribute('[*]', require('fragments-built-ins/binders/properties')());
fragments.registerAttribute('[style.*]', require('fragments-built-ins/binders/styles')());
fragments.registerAttribute('#*', require('fragments-built-ins/binders/ref')());
```

```html
Phone: {{phone.name}}
<ul class="phone-thumbs">
  <li ng-for="img in phone.images | filter(currentPhoneFilter)">
    <img [src]="img" (click)="setImage(img)">
  </li>
</ul>
<input #name-entry (keydown.enter)="saveName(nameEntry.value)">
<div class="progress-bar" [style.width.%]="loaded/total * 100"></div>
```

### Polymer 0.5

```js
var fragments = require('fragments-js');

// Leave the default attribute delimiters at {{}}

// Set up binders
fragments.registerAttribute('repeat', require('fragments-built-ins/binders/repeat')());
fragments.registerAttribute('if', require('fragments-built-ins/binders/if')());
fragments.registerAttribute('on-*', require('fragments-built-ins/binders/key-events')());
fragments.registerAttribute('*$', require('fragments-built-ins/binders/attributes')());
fragments.registerAttribute('value', require('fragments-built-ins/binders/value')());
```

```html
Phone: {{phone.name}}
<ul class="phone-thumbs">
  <template repeat="{{img in phone.images}}">
    <li>
      <img src$="{{img}}" on-click="{{setImage(img)}}">
    </li>
  </template>
</ul>
```
