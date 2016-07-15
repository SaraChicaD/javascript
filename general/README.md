# Eventbrite JavaScript Coding Style Guide

Guidelines and best practices used by Eventbrite to provide consistency and prevent errors in any environment where JavaScript code is written.

## Table of Contents

0. [Conditionals](#conditionals)
0. [Assignments](#assignments)
0. [Naming Conventions](#naming-conventions)
0. [Comments](#comments)

## Conditionals

### Complex conditional expressions

Complex conditional expressions within conditional statements can make understanding code challenging because you need to fully understand the expression in order to be able to process the overall code flow. In addition, in code reviews (or revisiting old code), it's difficult to know if the logic is correct without knowing what state the conditional expression is trying to represent.

To make things easier, store complex conditional expressions in state variables:

```js
// good
var shouldContinue = !options || (options.continue && options.hasWon);

if (shouldContinue) {

}

// bad
if (!options || (options.continue && options.hasWon)) {

}
```

**[⬆ back to top](#table-of-contents)**

### Negative conditional expressions

Positive conditional expressions are generally easier to follow than negative ones. Try to flip the logic so that the positive case is considered first:

```js
// good
if (something && somethingElse) {
    /* do stuff*/
} else {
    /* do other stuff */
}

// bad (negative case comes first)

if (!something || !somethingElse ) {
    /* do stuff */
} else {
    /* do other stuff */
}
```

**[⬆ back to top](#table-of-contents)**

### Multiple `if-else`

Creating a chain of conditional statements can make refactoring hard because the decisions are mixed with the actions. Instead, separate out the decisions into functions that can return simple actions:

```js
// good

// decision
var whatToDoWithTheThing = this.getSomethingAction();

// execution
this[whatToDoWithTheThing]();


// bad (mixed decisions and actions)
if (this.something()) {
   /* do stuff */
} else if (this.otherSomething()) {
   /* do other stuff */
} else if (this.somethingEvenMoreCurious()) {
   /* yet another thing */
} else {
   /* default something */
}
```

For more details, check out [_Replacing the `switch` statement for object literals_](https://toddmotto.com/deprecating-the-switch-statement-for-object-literals/).

#### Case study

**_First iteration_**: Multiple return statements and if-else statements that mix decision with execution.
```js
var getType = function(model) {
    if (model % 2 !== 0) {
        return 'odd';
    } else if (model % 2 !== 0) {
        return 'even';
    } else if (model === 'Eventbrite') {
        return 'Eventbrite';
    }
    return 'default';
};
```

**_Second iteration_**: A lookup map helps avoid using more than one return statement inside a given function and abstracts away the conditional logic. This makes it easier for maintainability and readability, as well as helping with performance by using `.find`.
```js
// good (use a lookup map)
var helpers = {
    isOdd: function(value) {
        return value % 2 !== 0;
    },
    isEven: function(value) {
        return value % 2 === 0;
    },
    isEventbrite: function(value) {
    	return value === 'Eventbrite';
    }
}

var types = {
    isOdd: 'odd',
    isEven: 'even',
    isEventbrite: 'eventbrite'
};

_getType = function(types, model) {
    var type = _.find(types, function(value, key) {
        // will return the first value of the key that evaluates to true;
        return helpers[key](model);
    }
    return type || 'default';
}

/* NOTE: We are using Underscore's .find method here, but with ES6, we can use find natively.
For more info, visit https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/find */
```
**[⬆ back to top](#table-of-contents)**

### Ternary statements

Simple ternary expressions can be handy when we need to conditionally assign a value to a variable when the condition is both true and false. However, when we only want to assign the variable when the condition is true, we may be tempted to still use a ternary and return `undefined, null, ''` for the false case.

In this case, the preferred approach is to declare the value without assigning it (the default value is `undefined`) and then using an `if` statement to assign to it when the condition is met:

```js
// good
var value;

if (options.isSomethingTrue) {
    value = 'hey there';
}

return value;

// bad (uses a ternary that returns undefined or null)
var options.isSomethingTrue ? 'hey there' : undefined;

```

Applying this pattern provides more robust maintenance and traceability (in debugging) over time and uses the basic structures of the language in a more formal way (ternaries are used under the assumption that we need a value).

This correlates as well with a defined state and later on if needed altering such state instead of having an default state that is dynamic.

#### Case study

Let's look at a case study of how code can evolve over time. For this example we have created a fictional piece of code.

**_First iteration_**: nothing fancy, just a method that needs to gather some information:

```js
var isAllowed = hasParent && isOwner,
    parentEvent = isAllowed ? getParentEvent() : undefined;

/* some use of these 2 variables later on... */
```

**_Second iteration_**: another developer comes around and adds yet another condition following the current style given that a refactor is not really needed, and probably out of scope:

```js
var isAllowed = hasParent && isOwner,
    parentEvent = isAllowed ? getParentEvent() : undefined,
    options = isAllowed ? undefined : getOptions();

/* some use of these 3 variables later on... */
```

**_Third iteration_**: another team comes in needing more functionality and adds even more variables:

``` js
var isAllowed = hasParent && isOwner,
    parentEvent = isAllowed ? getParentEvent() : undefined,
    options = isAllowed ? undefined : getOptions(),
    childEventsTitles = isAllowed ? getEventChildTitles() : undefined,
    ownerAccount = isAllowed ? undefined : getOwnerAccount();

/* some use of these 5 variables later on... */
```

At this point, telling what is the base state for this method is quite hard, given that all the possible states are based on ternaries. Furthermore, because we use the same `isAllowed` four times, we have to know how all the state variables work in order to try optimize the code. The code is too fragile.

However, if this code would have followed the initial recommendation, the code wouldn't degrade as more functionality is added.

**_First iteration (revisited)_**:

```js
var isAllowed = hasParent && isOwner,
    parentEvent;

if (isAllowed) {
	parentEvent = getParentEvent();
}

/* some use of these 2 variables later on... */
```

**_Second iteration (revisited)_**:

```js
var isAllowed = hasParent && isOwner,
    parentEvent,
    options;

if (isAllowed) {
	parentEvent = getParentEvent();
} else {
	options = getOptions();
}

/* some use of these 3 variables later on... */
```

 **_Third iteration (revisited)_**:

```js
var isAllowed = hasParent && isOwner,
    parentEvent,
    options,
    childEventsTitles,
    ownerAccount;

if (isAllowed) {
    parentEvent = getParentEvent();
    childEventsTitles = getEventChildTitles();
} else {
    options = getOptions();
    ownerAccount = getOwnerAccount();
}

/* some use of these 5 variables later on... */
```

**[⬆ back to top](#table-of-contents)**

## Assignments

### Variable indirection

Avoid variable indirection when possible. Reassigning variables when no transformation is needed creates unnecessary indirection, which is prone to introduce future errors.

```js
// good
_handleEvent = function(e) {
    _doSomethingWithEvent(e.target.checked);
}

// bad
_handleEvent = function(e) {
    var checked = e.target.checked;

    _doSomethingWithEvent(checked);
}
```

**[⬆ back to top](#table-of-contents)**

## Naming Conventions
> There are only two hard things in Computer Science: cache invalidation and naming things. *Phil Karlton*

### Variables: Classes or newable functions
Classes or newable functions (meant to be factories) should be singular, camelCase, and begin with a capital letter:

```js
var groupTicket = new Ticket(id: 555);
```
### Variables: Helpers and common variables

Any variable that's not a class or newable function, should be camelCase. This includes local variables, helper functions, modules, etc.

```js
//good
var pagination = require('pagination'),
    pages = pagination.getPages();

//bad
var Pagination = require('pagination'),
    pages = Pagination.getPages();
```

### Varialbles: Boolean

Variables that will contain a boolean value should start with either `is` or `has`:

```js
//good
var isAvailable = true,
    hasTickets = false;

//bad
var available = true,
    IHaveTickets = false;
```
### Variables: jQuery elements

Variables that contain jQuery elements should start with a dollar sign ($) so that we can quickly identify these types of variables by scanning the code (i.e. static analysis). This makes code review easier.

```js
var $elements = $('someSelector'), // in case a global search
    $element = this.$('some selector'); // in case of backbone
```
### Variables: Deferred

If a variable contains a jQuery.Deferred() or Promise, the variable should end in Dfd or Promise, respectively:

```js
//good
var fetchPromise = fetch('http://some/url'),
     requestDfd = new $.Deferred() // in case of jquery promises.

//bad
var request = fetch('http://some/url'),
    onResolve = new $.Deferred();
```

### Constants

Constats should be all written in capital letters and in snake_case. (except config object.)

```js
//good
const DEFAULT_VALUE = 'none';

var TIMEOUT = 3;

var config = {};

//bad
const defaultValue = 'none';

var timeout = 3;

var someOtherStaticVariable = 'blah';
```

### Variables: objects, strings, integers

Avoid names that are generic and instead, try to make your best effort to find a proper name in order to explain the content of it. If data needs to be changed, as part of a process, try to create methods to produce this changes on demand.

```js

//good
var attendeeNames = {},
    translatedGreeting 'hola!',
    getDefaultPrice = parseInt(5, 10);

//bad
var propsObject = {},
    trimmedString = 'hello   '.trim(),
    intPrice = parseInt(5, 10);
    
```

### Method: Private

Prefix private method names with an underscore (_):

```js

// good
var _getInternalValue = function() {
    // code here.
};

// bad
var getInternalValuePleaseDonot = function() {
    // code here.
};
```

Using the underscore prefix naming convention for private methods allows us to easily identify which methods are not part of the component's public API. As a result, future developers can easily know which methods can be refactored or even removed without affecting any users of the component. This makes upgrading to new technologies/tools/idioms simpler.

The naming convention also helps code reviewers to quickly understand which methods have been added by us versus which are a part of the framework's lifecycle methods (see: [react](https://facebook.github.io/react/docs/component-specs.html#lifecycle-methods) or [Marionette](https://marionette.gitbooks.io/marionette-guides/content/en/views/triggers.html)).

### Method: public

should always begin with a [lowerCase](http://c2.com/cgi/wiki?LowerCamelCase)

```js

//good
getNode: function() {
        //code here.
    },
setAttr: function() {
        // code here.
    };

//bad

_superComplextName: function() {
    // code here.
}

UpdateAll: function() {
    / /code here
}
```
## comments

### commented code

*We recommend not to leave commented code lingering around. If the code is meant to be used in the future, either save the code on a branch or in our internal paste bin via phabricator. (arc paste).*

### when the comment has multiple lines we use the block form (i.e: JSDocs)

```js
 
/**
* returns the calculated offset for lower bound range. (from)
* @example
* // returns 2
* scenario:
* Viewport Large
* skiping first and applying corrections
* @returns {integer} lower bound range
*/

const getMinBound = (pageSize = constants.DEFAULT_PAGE_SIZE) => (
    pageSize === constants.DEFAULT_PAGE_SIZE ? constants.MIN_BOUND_SMALL : constants.MIN_BOUND_DEFAULT
);
```

### when the comment has a single line we instead one liner comments: 

```js
const getToOffset = (pageNumber, middleRangeOffset, maxPagesShown, maxBound) => {
    // default -> page is in the middle, and we have enough pages in both sides.
    let to = pageNumber + middleRangeOffset;

    // are we too close to the bottom? if so, we need to adjust the split
    // relative checks where the split doesn't work (sorry van damme)
    if (to < maxPagesShown + 1) {
        to = pageNumber + maxPagesShown;
    }

    // hard bound to the top
    if (to > maxBound) {
        // on small + 1
        to = maxBound;
    }

    return to;
};
```

**[⬆ back to top](#table-of-contents)**
