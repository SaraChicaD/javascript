# Eventbrite Backbone & Marionette Coding Style Guide

Guidelines used by Eventbrite to provide consistency and prevent errors in JavaScript code written for [Backbone.js](backbonejs.org) and [Marionette.js](marionettejs.com).

## Table of Contents

0. [What is Backbone.js?](#what-is-backbonejs)
	0. [What not to use from Backbone] (#what-not-to-use-from-backbone)
0. [What is Marionette.js?](#what-is-marionettejs)
	0. [What not to use from Marionette] (#what-not-to-use-from-marionette)
0. [What Plugins are used at Eventbrite?](#what-plugins-are-used-at-eventbrite)
0. [Common terminology] (#common-terminology)
0. [Good Practices](#good-practices)
	0. [Function] (#good-practices-functions)
	0. [Hydrating apps](#good-practices-hydrating-apps)
		0. [Static or on Bootstrap] (#good-practices-static-hydration)
		0. [dynamic] (#good-practices-dynamic-hydration-apps)
	0. [Marionette.Layout](#marionette-layout)
		0. [Regions] (#marionette-regions)
	0. [Marionette.Views] (#marionette-views)
	0. [Backbone.Model] (#backbonemodel)
		0. 	[Handling errors] (#handling-errors-on-models)
	0. [Backbone.Collection] (#marionette-collection)
		0. [Handling errors] (#handling-errors-on-collections)
0. [Marionette Artifacts Life Cycle] (#marionette-artifacts-life-cycle)
0. [Backbone Life Cycle] ()
0. [Architecting JS Apps at Eventbrite](#architecting-js-apps-at-eventbrite)
	0. [app.js] (#app.js)
	0. [Templates] (#templates)
	0. [File structure] (#file-structure)
	0. [File name conventions] (#file-name-conventions)
	0. [Eb Flux] (#eb-flux-architecture)
		0. [Stores] (#eb-flux-stores)
		0. [Views] (#eb-flux-views)
		0. [Actions] (#eb-flux-actions)
0. [debugging common issues] (#debugging-common-issues)


## What is Backbone.js

From the [Backbone.js](http://backbonejs.org/) docs:

> Backbone.js gives structure to web applications by providing **models** with key-value binding and custom events, **collections** with a rich API of enumerable functions, **views** with declarative event handling, and connects it all to your existing API over a RESTful JSON interface.

Eventbrite still uses v1.0.0 of Backbone.
For more, see [Getting started with Backbone.js](http://backbonejs.org/#Getting-started).

### What not to use from Backbone
*Backbone.View*: deprecated in our platform giving the presence of **Marionette** in every page. Please use [Marionette.Views] (#marionette-views) instead.

## What is Marionette.js

From the [Marionette.ItemView.js](http://marionettejs.com/) docs:

> Marionette simplifies your Backbone application code with robust views and architecture solutions.

Eventbrite still uses v1.8.8 of Marionette.ItemView. For more, see [Marionette v1.8.8 docs](http://marionettejs.com/docs/v1.8.8/).

We use a selection of Plugins that improve some aspects of Marionette/Backbone, for more, see [what Plugins are used at Eventbrite?] (#what-plugins-are-used-at-eventbrite)

### What not to use from Marionette.js

*Marionette.Modules*: deprecated in our platform and still in use in some parts of the product (such **listings** and **my_contacts**) but we advice not to use it. instead use [Marionette.Layout] (#marionette-layout).

*Mariontette.Controller*: deprecated by Marionette it self and confusing giving that we don't do MVC on FrontEnd. instead use [Marionette.Layout] (#marionette-layout) or a [Marionette.Object] (#marionette-Object)

## What Plugins are used at Eventbrite?

We have a couple of plugins/libraries to enhance and simplify our use of Backbone/Marionette.ItemView.

	* Backbone.Advice: Functional mixins for Backbone based
	* dorsal: HTML decorator library.
	* backbone.stickit: Backbone data binding plugin that binds Model attributes to View elements.
	* backbone.validation: A validation plugin for Backbone.js that validates both your model as well as form input
	* backbone.wreqr: A simple infrastructure based on messaging patterns and service bus implementations for decoupling Backbone and Backbone.Marionette applications.

## Common terminology
- Context
- Hydrating
- Bootstrap
- Module
- Component
- app
- parameters
- argument
- config
- artifact
- helpers
- mixins
- base_bundle
- bundle

# Good Practices

Backbone and Marionette come with a rich API and also functions provided by `underscore (_)` and `jquery($)`, this although good and fast to use, can be hard to navigate or even challenging when build applications with the same concepts behind it. this guide will attempt to ease some this of problems.

### Requiring Marionette

Backbone and Marionette are realated (we can pull Marionette from within Backbone):

 ``` javascript
 	//Bad
 	Backbone = require('backbone');

 	return Backbone.Marionette.ItemView.extend({ /* do something here */ });
 ```
We recommend to pull Marionette independiently
**Reason**: If we try to simplify our stack we would have to change a considrable amount of code to remove Backbone as a dependency or a namespace. instead:

 ``` javascript
 	//Good
 	Marionette = require('marionette');

 	return Marionette.ItemView.extend({ /* do something here */ });
 ```

### one return per file.
We try to keep one Artifact per file.

 ``` javascript
 	file_with_views.js

 	// Bad
 	var Marionette = require('marionette'),
 		a = Marionette.ItemView.extend({ /* do something here */ }),
 		b = Marionette.ItemView.extend({ /* do something here */ });

 	return {a: a, b: b};

 ```

 instead we would have 2 files.

 ``` javascript
 	// Good

 	//view_a.js

 	var Marionette = require('marionette');

 	return Marionette.ItemView.ItemView.extend({ /* do something here */ });

 	//view_b.js

 	var Marionette = require('marionette');

 	return Marionette.ItemView.ItemView.extend({ /* do something here */ });
 ```

### return without namespace
Find yet another name for a variable that only is used to be returned is hard. let's not waste energy there.

 ``` javascript
 	// Bad

 	var Marionette = require('marionette'),
 		a;

 	a = Marionette.ItemView.ItemView.extend({ /* do something here */ });

 	return a;

 	// Good

 	var Marionette = require('marionette');

 	return Marionette.ItemView.ItemView.extend({ /* do something here */ });
 ```

## Good Practices: Functions

### Stateless functions (helpers, static functions)

When we write Views or Models/Collections we tend to enclusure all our functions within the artifact. Sometimes we have `functions` that are used as `helpers`. this technic gets much more use when working in other technologies where the framework is less predominant. We tend to recomend such technic when possible.

If a function doesn't need context (not binded to `this`) we recommend to extract this function, making it private. this helps to simplify what we expose as well as the code review process, it also helps to understand possible issues when dynamic interpretation happens (when running the code).

``` javascript
	// Bad
	var Marionette = require('marionette');

 	return Marionette.ItemView.extend({
 		initialize: function(options) {
 			var attrs = this.exractAttributes(options);

 			this.model = new Backbone.Model(attrs);
 		},
 		extracAttributes: function(options) {
			var attrs = {};
			// some process that needs to be done
			return attrs;
		}
 	});

 	// Good
 	var Marionette = require('marionette');

	function extracAttributes(options) {
		var attrs = {};
		// some process that needs to be done
		return attrs;
	};

 	return Marionette.ItemView.extend({
 		initialize: function(options) {
 			var attrs = exractAttributes(options);

 			this.model = new Backbone.Model(attrs);
 		};
 	});
```
### Static content (config object, magical words/numbers).
Often we are in need of some `className`, `selector` or `text (gettext)` that needs to be used but doesn't change (is not provided by any dynamic or external input such as a partent View, Layout or event). For such occasions we store all this data in a static object called `config`.

``` javascript
	// Bad
	var $ = require('jquery'),
		Marionette = require('marionette');

	return Marionette.ItemView.extend({
  		initialize: function(options) {
 			$('someDynamicSelector').addClass('is-hidden');

 			window.setTimeout(this.someCallback, 10);
 		}
 	});

	// Ok
	var $ = require('jquery'),
		Marionette = require('marionette');

	return Marionette.ItemView.extend({
		config: {
			'selectorName': 'someDynamicSelector',
			'isHiddenClass': 'is-hidden',
			timeOut: 10
		},
 		initialize: function(options) {
			$(this.config.selectorName).addClass(this.config.isHiddenClass);

 			window.setTimeout(this.someCallback, this.config.timeOut);
 		}
	});

	// Good
	var $ = require('jquery'),
		Marionette = require('marionette'),
		config = {
			'selectorName': 'someDynamicSelector',
			'isHiddenClass': 'is-hidden',
			timeOut: 10
		};

	return Marionette.ItemView.extend({
        initialize: function(options) {
            $(config.selectorName).add(config.isHiddenClass);
 		    window.setTimeout(this.someCallback, config.timeOut);
 		}
	});
```
### styles and js separate

put styles classes in handlebars and the logic in the view.  this simplify the searchs when trying to find templates.

``` javascript

	// Bad

	return Marionette.ItemView({
		classname: 'g-cell g-cell-12-12'
	});

	// Good

	/*some_view.handlebars */

	<div class="g-cell g-cell-12-12"> </div>

	/* some_view.js */

	var Marionette = require('marionette'),
		template = require('hb!./some_view.handlebars');

	return Marionette.ItemView({
		template: template
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
