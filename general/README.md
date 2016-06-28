# Eventbrite JavaScript Coding Style Guide

Guidelines and best practices used by Eventbrite to provide consistency and prevent errors in any environment where JavaScript code is written.

## Table of Contents

0. [Conditionals](#conditionals)
0. [Assignments](#assignments)

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
}  else if (this.otherSomething()) {
   /* do other stuff */
} else if (this.somethingEvenMoreCurious()) {
   /* yet another thing */
} else {
   /* default something */
}
```

For more details, check out [_Replacing the `switch` statement for object literals_](https://toddmotto.com/deprecating-the-switch-statement-for-object-literals/).

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

**First iteration (revisited)_**:

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
