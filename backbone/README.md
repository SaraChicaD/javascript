# Eventbrite Backbone & Marionette Coding Style Guide

Guidelines used by Eventbrite to provide consistency and prevent errors in JavaScript code written for [Backbone.js](backbonejs.org) and [Marionette.js](marionettejs.com).

Backbone and Marionette come with a rich API and also functions provided by [underscore](http://underscorejs.org/) (`_`) and [jquery]((http://api.jquery.com/category/version/1.7/)) (`$`). Although good and fast to use, these utilities can be hard to navigate or even challenging when building large-scale applications. Many times midway through development, we find that were used the tools incorrectly and have to change course, resulting in Frankenstein code. This guide will attempt to ease some of these problems.

## Table of Contents

0. [Backbone.js](#backbonejs)
0. [Marionette.js](#marionettejs)
0. [Additional plugins](#additional-plugins)
0. [Common terminology](#common-terminology)
0. [File structure](#file-structure)
0. [Statics](#statics)
0. [Styling](#styling)
0. [Good Practices](#good-practices)
	0. [Function](#good-practices-functions)
	0. [Hydrating apps](#good-practices-hydrating-apps)
		0. [Static or on Bootstrap](#good-practices-static-hydration)
		0. [dynamic](#good-practices-dynamic-hydration-apps)
	0. [Marionette.Layout](#marionette-layout)
		0. [Regions](#marionette-regions)
	0. [Marionette.Views](#marionette-views)
	0. [Backbone.Model](#backbonemodel)
		0. 	[Handling errors](#handling-errors-on-models)
	0. [Backbone.Collection](#marionette-collection)
		0. [Handling errors](#handling-errors-on-collections)
0. [Marionette Artifacts Life Cycle](#marionette-artifacts-life-cycle)
0. [Backbone Life Cycle]()
0. [Architecting JS Apps at Eventbrite](#architecting-js-apps-at-eventbrite)
	0. [app.js](#app.js)
	0. [Templates](#templates)
	0. [File structure](#file-structure)
	0. [File name conventions](#file-name-conventions)
	0. [Eb Flux](#eb-flux-architecture)
		0. [Stores](#eb-flux-stores)
		0. [Views](#eb-flux-views)
		0. [Actions](#eb-flux-actions)
0. [debugging common issues](#debugging-common-issues)


## Backbone.js

From the [Backbone.js](http://backbonejs.org/) docs:

> Backbone.js gives structure to web applications by providing **models** with key-value binding and custom events, **collections** with a rich API of enumerable functions, **views** with declarative event handling, and connects it all to your existing API over a RESTful JSON interface.

Eventbrite still uses v1.0.0 of Backbone. For more, see [Getting started with Backbone.js](http://backbonejs.org/#Getting-started).

_NOTE:_ [`Backbone.View`](http://backbonejs.org/#View) is deprecated in favor of using [Marionette views](#marionette-views).

## Marionette.js

From the [Marionette.js](http://marionettejs.com/) docs:

> Marionette simplifies your Backbone application code with robust views and architecture solutions.

Eventbrite still uses v1.8.8 of Marionette.ItemView. For more, see [Marionette v1.8.8 docs](http://marionettejs.com/docs/v1.8.8/).

_NOTE:_ [`Marionette.Application.module`](http://marionettejs.com/docs/v1.8.8/marionette.application.module.html) is deprecated in favor of [`Marionette.Layout`](http://marionettejs.com/docs/v1.8.8/marionette.layout.html). You will still see it used in certain parts of the product, such as in **Listings** or **My Contacts**.

_NOTE:_ [`Marionette.Controller`](http://marionettejs.com/docs/v1.8.8/marionette.controller.html) is deprecated in favor of [`Marionette.Layout`](http://marionettejs.com/docs/v1.8.8/marionette.layout.html). [`Marionette.Object`](http://marionettejs.com/docs/v2.1.0/marionette.object.html) is also available. It was taken from a later version of Marionette and stitched in.

## Additional plugins

We have a couple of plugins/libraries to enhance and simplify our use of Backbone/Marionette:

- [`Backbone.Advice`](https://github.com/rhysbrettbowen/Backbone.Advice): Adds functional mixin abilities for Backbone objects
- [`dorsal`](https://github.com/eventbrite/dorsal): An HTML decorator library
- [`Backbone.Stickit`](https://github.com/NYTimes/backbone.stickit): Backbone data binding plugin that binds Model attributes to View elements
- [`Backbone.Validation`](https://github.com/thedersen/backbone.validation): A validation plugin for Backbone that validates both your model as well as form input
- [`Backbone.Wreqr`](https://github.com/marionettejs/backbone.wreqr): Messaging patterns for Backbone applications

## Common terminology

- _context_ -
- _hydrating_ -
- _bootstrap_ -
- _module_ -
- _component_ -
- _app_ -
- _parameters_ -
- _argument_ -
- _config_ -
- _artificat_ -
- _helpers_ -
- _mixins_ -
- _base bundle_ -
- _bundle_ -

## File structure

A reference to `Marionette` can actually be retrieved from a reference to `Backbone`. However, we recommend requiring `Marionette` separately so that if we try to simply our stack, we don't have to change a considerable amount of code to remove the `Backbone` dependency/namespace:

```js
// good
var Marionette = require('marionette');

return Marionette.ItemView.extend({ /* do something here */ });

// bad (access Marionette from Backbone)
var Backbone = require('backbone');

return Backbone.Marionette.ItemView.extend({ /* do something here */ });
```

Whenever possible, return only one [artifact](#common-terminology) per file:

```js
// good

//view_a.js

var Marionette = require('marionette');

return Marionette.ItemView.ItemView.extend({ /* do something here */ });

//view_b.js

var Marionette = require('marionette');

return Marionette.ItemView.ItemView.extend({ /* do something here */ });


// bad (returning multiple artifacts in one file)

var Marionette = require('marionette'),
	ViewA = Marionette.ItemView.extend({ /* do something here */ }),
	ViewB = Marionette.ItemView.extend({ /* do something here */ });

return {ViewA: ViewA, ViewB: ViewB};
```

Whenever possible, return the artifact immediately instead of assigning to a variable that just gets returned afterward:

```js
// good
var Marionette = require('marionette');

return Marionette.ItemView.ItemView.extend({ /* do something here */ });

// bad (assigns the ItemView to a variable unnecessarily)
var Marionette = require('marionette'),
	MyItemView;

MyItemView = Marionette.ItemView.extend({ /* do something here */ });

return MyItemView;
```

## Statics

When we write views or models/collections, we tend to enclose all of our functions as methods on the artifact. However, sometimes these methods are really just static helpers that don't need context (i.e. not bound to `this`). In this case, it's better to extract out the function as a private helper, which also simplifies the API exposed by the artifact:

```js
// good
var Marionette = require('marionette');

function extractAttributes(options) {
	var attrs = {};
	// do stuff
	return attrs;
};

return Marionette.ItemView.extend({
	initialize: function(options) {
		var attrs = extractAttributes(options);

		this.model = new Backbone.Model(attrs);
	};
});

// bad (extractAttributes is an additional method on the view unnecessarily)
var Marionette = require('marionette');

return Marionette.ItemView.extend({
	initialize: function(options) {
		var attrs = this.exractAttributes(options);

		this.model = new Backbone.Model(attrs);
	},
	extracAttributes: function(options) {
		var attrs = {};
		// do stuff
		return attrs;
	}
});
```

Oftentimes an artifact needs some static/constant data that never need to change. Instead of having magic numbers/strings in the code, or having a configuration object attached to each instance, we should store the configuration information in a const object variable:

```js
// good
var $ = require('jquery'),
	Marionette = require('marionette'),
	config = {
		selectorName: 'someDynamicSelector',
		isHiddenClass: 'is-hidden',
		timeout: 10
	};

return Marionette.ItemView.extend({
	initialize: function(options) {
		$(config.selectorName).add(config.isHiddenClass);
		window.setTimeout(this.someCallback, config.timeout);
	}
});

// ok (config objects exists as a property for each view instance)
var $ = require('jquery'),
	Marionette = require('marionette');

return Marionette.ItemView.extend({
	config: {
		selectorName: 'someDynamicSelector',
		isHiddenClass: 'is-hidden',
		timeout: 10
	},
	initialize: function(options) {
		$(this.config.selectorName).addClass(this.config.isHiddenClass);
		window.setTimeout(this.someCallback, this.config.timeout);
	}
});

// bad (uses magic numbers/strings)
var $ = require('jquery'),
	Marionette = require('marionette');

return Marionette.ItemView.extend({
	initialize: function(options) {
		$('someDynamicSelector').addClass('is-hidden');
		window.setTimeout(this.someCallback, 10);
	}
});
```

## Styling

To simplify searches when trying to find templates, put CSS classes in handlebars templates instead of coupling it with the view logic:

```js
// good

// some_view.handlebars

<div class="g-cell g-cell-12-12"></div>

// some_view.js

var Marionette = require('marionette'),
	template = require('hb!./some_view.handlebars');

return Marionette.ItemView({
	template: template
});


// bad (CSS classes aren't separated out)
var Marionette = require('marionette');

return Marionette.ItemView({
	classname: 'g-cell g-cell-12-12'
});
```

## Good Practices: Algorithms

there are some common issues that we encounter regarding logic preferences, the following recommendations are based on what we have seen over the past years that helps to our code review speed and helps.

### avoid negative statements in conditional expressions

When evaluating logic, positive conditional expresion form are easier to wrap our heads around them. instead we could gather the options in a variable so it can be understood later or try to flip the logic so it considers the positive case first.

``` javascript
	// Bad

	if (!something || !somethingElse ) {
		/* do stuff */
	} else {
		/* do other stuff */
	}

	// Ok

	var hasTheNeededState = !something || !somethingElse;

	if (hasTheNeededState) {
		/* do stuff*/
	} else {
		/* do other stuff */
	}

	// Good

	if (something || somethingElse) {
		/* do stuff*/
	} else {
		/* do other stuff */
	}
```

### avoid nested `if else` statements

Creating nested chains of conditions makes refactor really hard. instead split the decision into functions that can return simple actions. (separate decision from action);

``` javascript
 // Bad

 if (this.something()) {
 	/* do stuff */
 }  else if (this.otherSomething()) {
 	/* do other stuff */
 } else if (this.somethingEvenMoreCurious()) {
 	/* yet another thing */
 } else {
 	/* default something */
 }

 // Good
 /* decision */
 var whatToDoWithTheThing = this.getSomethingAction();
 /* exectution */
 this[whatToDoWithTheThing]();

```

### avoid using `this` to store persistent data

contextual data is often fragile and error prone when code review (static analisis).

instead we could choose to calculate when is required or have a model storing the needed computed information  or reference.
 > keep in mind `options` received on instantiation are reachable via `this.options` inside the view.

``` javascript

	// Bad
	return Marionette.ItemView.extend({
		initialize: function(options) {
			this.computedData = someTransformation(options);
		}

		getComputedData: function() {
			return this.computedData;
		}
	});

	// Ok
	return Marionette.ItemView.extend({

		getComputedData: function() {
			return someDataTransformation(this.options);
		}
	});

	// Good
	return Marionette.ItemView.extend({

		getComputedData: function() {
			return this.model.getComputedData();
		}
	});
```

### default case instead of ternary statement

sometimes we need to assign a value only when a condition is met. in these cases we will be tempted to use a simple ternanry even tho we don't have a value for when the condition is not met.

For these cases we prefer a default case and to alter such a value if the condition is met.

applying this pattern provides more robust mantainability over time and uses the basic structures of the language in a more formal way (ternaries are used under the assumption that we need a value)

``` javascript
	// Bad
	return Marionette.ItemView.extend({
		someMethod: function(options) {
			return options.isSomethingTrue ? 'hey there' : undefined;
		}
	});

	// Good
	return Marionette.ItemView.extend({
		someMethod: function(options) {
			var value;

			if (options.isSomethingTrue) {
				value = 'hey there';
			}

			return value;
		}
	});
```

### Native bind instead of `_.bindAll`

We have [es5-shim](https://github.com/es-shims/es5-shim) as part of our `base_bundle` thus, `bind` is present and we sholuld use it in favor of any other utility. (like `underscore`).

``` javascript
	// Bad
	return Marionette.ItemView.extend({
		initialize: function(options) {
			_.bindAll(this, 'someMethod');

			this.listenTo(channel.vent, 'someSignal', this.someMethod);
		},

		someMethod: function(options) {
			/* do something */
		}
	});

	// Good
	return Marionette.ItemView.extend({
		initialize: function(options) {
			this.listenTo(channel.vent, 'someSignal', this.someMethod.bind(this));
		},

		someMethod: function(options) {
			/* do something */
		}
	});
```
