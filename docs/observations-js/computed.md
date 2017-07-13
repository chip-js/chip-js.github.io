---
layout: default
---


---
layout: default
---


---
layout: default
---


---
layout: default
---


---
layout: default
---


---
layout: default
---


---
layout: default
---


---
layout: default
---


---
layout: default
---


---
layout: default
---


---
layout: default
---


---
layout: default
---


---
layout: default
---


---
layout: default
---


---
layout: default
---


---
layout: default
---


# Computed Properties

Observations.js makes computing object properties easy. Computed properties resolve data using expressions.js and add a
few additional features for loading, mapping, and managing data.

## Basic Usage

The `computed` library creates or extends an object using a hash of computed properties.

Examples:

```js
var observations = require('observations-js');
var computed = observations.computed;
var user = computed({
  fullName: 'firstName + " " + lastName'
});

obj.firstName = 'Bob';
obj.lastName = 'Smith';
observations.syncNow();
console.log(obj.fullName); // Bob Smith
```

```js
var user = { firstName: 'Bob', lastName: 'Smith' };

computed.extend(user, {
  fullName: 'firstName + " " + lastName'
});
console.log(obj.fullName); // Bob Smith
```

### `sync()` and `syncNow()`

Whenever data changes the computed properties must be synced with the data. Using explicit methods to sync with the data
allows you to use robust JavaScript expressions in your computed properties. It also allows you to use any data you want
without requiring special getters and setters or a custom model.

The `observations.sync()` method is what you mostly want to use. It will queue up a data-sync to be run just after the
current microtask (using `Promise.resolve()`). You can call this multiple times during the current microtask and
it will only run the sync once. It is very performant.

Example:

```js
var data = computed({
  loadMe: function() {
    ajax.get('/me').then(function(me) {
      this.me = me;
      observations.sync();
    }.bind(this));
  },

  myName: 'me.name'
});

data.loadMe();
```

In this example, as soon as `me` is loaded then `myName` will be set before the next render. By putting
`observations.sync()` into your data loading libraries (as was done with `computed.resource` which you can read about
below) you may not need to call `observations.sync()` very often.

You may need the changes to be applied right away. You have the option of calling `observations.syncNow()` which will
run the data-sync immediately.

## Computed API

The API has two methods to define the properties on an object. The remaining methods return a computed propert which
handles the computation. A string is a shortcut for `computed.expr()`.

### Defining Properties

#### `computed(definition[, options])`

Creates and returns a new object that has the computed properties bound to it. You may pass in `{ enabled: false }` in
the `options` to have it unbound until you bind it later (see `computedObservers.enable()` below).

#### `computed.extend(obj, definition[, options])`

Extends an existing object (any object will work) with the computed properties defined. You may pass in
`{ enabled: false }` in the `options` to have it unbound until you bind it later (see `computedObservers.enable()`
below).

### ComputedProperties

When creating or extending an object with `computed` you define the properties to be computed. These create computed
properties that handle how changes in the expressions are handled. The most common computed property is `computed.expr`
which gets created when you just use a string expression. The following describes all available computed properties, how
they work, and when you might use them.

#### `computed.expr(expression)` or `'expr'`

When you just use a string, or use `computed.expr(expression)`, it creates an expression computed property. This simply
sets the value of the expression to the property.

Example:

```js
computed.extend(obj, {
  fullName: computed.expr('firstName + " " + lastName')
});

// or

computed.extend(obj, {
  fullName: 'firstName + " " + lastName'
});
```

#### `computed.if(ifExpression, thenExpression)`

Will only set the result of `thenExpression` to the property if the `ifExpression` evalutes to a truthy value. Use
this when you want to only load/compute/set data if a state is true. Expressions are usually evaluated every sync cycle,
so this allows you to have the `thenExpression` only run once, and only if your conditional is true.

When the if-expression is true, the then-expression is observed and any changes will be set to the property accordingly.

The `thenExpression` can just be a string expression, but it can also be any of the other computed properties.

Example:

```js
computed.extend(obj, {
  settings: computed.async('loadSettings()'),
  isFeatureXEnabled: 'settings.featureXEnabled',
  xList: computed.async('isFeatureXEnabled', 'loadXList()')
});
```

#### `computed.map(sourceExpression, keyExpression[, valueExpression])`

Creates an object lookup of the source array. The `sourceExpression` should be the format "item in array" and will
create an entry in the map for each item with the key and value. If the `valueExpression` is not included it will
default to the item. `valueExpression` may be any computed property.

Example:

```js
computed.extend(obj, {
  maps: computed.async('loadMaps()'),
  mapsById: computed.map('map in maps', 'map.id', 'map'),
  queues: computed.async('loadQueues()'),
  queueMapNames: computed.map('queue in queues', 'queue.id', 'mapsById[queue.mapId].name')
});
```

#### `computed.async([whenExpression, ]thenExpression)`
#### `computed.when([whenExpression, ]thenExpression)` (alias)

Similar to the `if` computed property, `async` and `when` only evaluate their `thenExpression` when the the first
property is true. Unlike `if`, the `thenExpression` is not observed. The property will only be set to the results of
the `thenExpression` whenever the `whenExpression` changes. The `if` computed property will observe the `thenExpression`
and continually update the property with changes so long as the "if" remains true.

Use this when you have an asyncronous call to be made or a computationally intensive calculation to run. You wouldn't
want to run an ajax call or a calculation every sync cycle to see if the value has changed.

The `thenExpression` can only be an expression, not another computed property. However the result of the
`thenExpression` may optionally return a Promise whose value will be set to the property.

If you leave out the `whenExpression` then the expression will be evaluted once and only once. Use this to load your
intial data.

Example:

```js
obj = {
  searchString: '',
  fuzzyFilter: function(user) {
    // computationally intense calculation
  }
};

computed.extend(obj, {
  gameInfo: computed.async('selectedGameId', 'loadGameInfo(selectedGameId)'),
  users: computed.async('loadusers()'),
  filteredusers: computed.when('searchString', 'users | filter(fuzzyFilter)')
});
```

In the above example, the game info is only updated when the `selectedGameId` changes (and remains undefined when it is
not set). The users are loaded immediately and only once. And the users are filtered only once when a `searchString`
is entered or changed, but not every sync cycle.

#### `computed.resource(url, idProperty = "id")`

This is very specific to Riot's client libraries. Whenever the `url` changes this computed property will load the data
from the server and continue watching the websocket for changes to that data. When the data is loaded, and after any
updates come through, a sync is triggered automatically, cascading the changes through the rest of your data.

You may use curly brace expression (`{{ expr }}`) within your URLs, although formatters do not work in this manner
(existing bug). You can use `encodeURIComponent` if needed. Using expressions to fill in a section between two slashes
will let `resource` know when to load or change the data. E.g. if you use
`computed.resource('/lol-plugin/v1/items/{{ itemId }}') and `itemId` is not yet set, the computed property will see
`"/lol-plugin/v1/items/"` and will assume the trailing slash means the URL is incomplete. It will not load it. Once
`itemId` is set and the URL becomes `"/lol-plugin/v1/items/31"` the resource will be loaded and watched. If `itemId`
changes from `31` to `45` the old data is dropped and the new data is loaded and watched. If `itemId` then becomes
`null` the data is dropped and cleaned up. This works the same for URLs that look like
`'/lol-plugin/v1/items/{{ itemId }}/subresource'`.

Example:

```js
computed.extend(obj, {
  users: computed.resource('/lol-users/v1/users'),
  userOptions: computed.map('user in users', 'user.id',
      computed.resource('/lol-users/v1/users/{{ user.id }}/options'))
});
```

In this example you can see how using computed properties inside other computed properties can be useful. Here we are
loading the options for each user. Be careful to not fload the server with requests this way and only load the data
you need.

### Components

Components in Components.js have some built-in support for `computed`. By defining a property called `computed` on a
component the component will be automatically extended with `computed.extend`. The computed properties will be enabled
once the component is attached and disabled after it is detached. The keeps inactive components from continuing to
process data unnecessarily. If you need data loaded before a component is attached to the screen consider doing this
in an extended object outside of the component lifecycle and assign it to the component (possibly in the default mixin
to be available to all components if it is global data).

Example:

```js
// data.js
var computed = require('components-js').computed;

// set this data on the exports object
computed.extend(exports, {
  users: computed.resource('/lol-users/v1/users'),
  usersById: computed.map('user in users', 'user.id')
});
```

```js
// my-component.js
var data = require('./data');
var components = require('components-js');

components.defineElement('my-component', {
  data: data,

  computed: {
    user: 'data.usersById[userId]'
  }
});
```

In this example, the users are loaded ahead of time and when `my-component` has its `userId` set, it will set the
user immediately from the existing collection.

### computedObservers

When you create or extend an object with `computed`, it adds a property to the object called `computedObservers` to
hold the observers for each computed property. The observers are enabled by default and will immediately compute the
values, however you can disable them by default by passing in `{ enabled: false }` in the `options`. You can then enable
or disable them any time. Components use this to enable the computed properties when attached and to disable them when
detached.

#### `computedObservers.enabled`

Whether the observers are enabled or not.

#### `computedObservers.enable()`

Enables the observers for the computed properties if not already enabled. This will immediately set the properties and
then keep them up-to-date aftewards.

Example:

```js
var obj = {
  foo: 'bar'
};

computed.extend({
  foobar: 'foo'
}, { enabled: false });

obj.foobar; // undefined
obj.computedObservers.enable();
obj.foobar; // bar
obj.computedObservers.disable();
obj.foobar; // undefined
```

#### `computedObservers.disable()`

Disables the observers for the computed properties. This will set the properties to `undefined` and readies the object
for garbage collection. If you do not disable computed properties an object cannot be garbage collected.

Example:

```js
var obj = {
  foo: 'bar'
};

computed.extend({
  foobar: 'foo'
}, { enabled: false });

obj.foobar; // undefined
obj.computedObservers.enable();
obj.foobar; // bar
obj.computedObservers.disable();
obj.foobar; // undefined
```

Next up, Advanced Usage, Data-binding

If you'd like to become an expert with Components.js, learn more about [data-binding, binders, and how you can create
your own](data-binding.md).
