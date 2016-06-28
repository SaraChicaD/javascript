# Eventbrite JavaScript Coding Style Guide

Guidelines and best practices used by Eventbrite to provide consistency and prevent errors in any environment where JavaScript code is written.

## Table of Contents

0. [Conditionals](#conditionals)


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

## Ternary statements

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

Applying this pattern provides more robust maintenance and Traceability (on debugging) over time and uses the basic structures of the language in a more formal way (ternaries are used under the assumption that we need a value).

This correlates as well with a defined state and later on if needed altering such state instead of having an default state that is dynamic.

a good case study could be the following code: 

for this example we have created a ficcional segment of code. 

**_First iteration_**: nothing fancy, just a method that needs to gather some information.

```js
var isAllowed = hasParent & isOwner,
    parentEvent = isAllowed ? getParentEvent() : undefined;
    
    /* some use of these 2 variables late on..*/
```
**_Second iteration_**: Another developer comes around and adds yet another condition following the current style given that a refactor is neither need nor part of the scope. 

```js
var isAllowed = hasParent & isOwner,
    parentEvent = isAllowed ? getParentEvent() : undefined,
    anotherOptions = isAllowed ? undefined : getOptions();
   
    /* some use of these 3 variables late on..*/
```
**_Third iteration_**: yet another team needs more variables added to this method to add features to it. 

``` js
var isAllowed = hasParent & isOwner,
    parentEvent = isAllowed ? getParentEvent() : undefined,
    anotherOptions = isAllowed ? undefined : getOptions();
    childEventsTitles = isAllowed ? getEventChildTitles() : undefined;
    ownerAccount = isAllowed ? undefined : getOwnerAccount(); 
   
    /* some use of these 4 variables late on..*/
```

At this point telling what is the base state for this method is quite hard giving that all the possible states are based on ternaries. 

furthermore optimization (we use the same isAllowed 4 times) now is deliced given that we should understand this code way better. 

if this code would have fallowed our recomendation, the steps would look like the most clear way we could write this code. 

**_first Step revisited_**:

```js
var isAllowed = hasParent & isOwner,
    parentEvent = undefined;
    
    If (isAllowed) {
    	parentEvent = getParentEvent();
    }
    
    /* some use of these 2 variables late on..*/
```
 **_Third iteration revisited_**

```js
var isCurry = bitmask & CURRY_FLAG,
    newHolders,
    newPartials,
    newHoldersRight,
    newPartialsRight;
   
if (isCurry) {
   newHolders = holders;
   newPartials = partials;
} else {
 	newHodldersRight = holders;
 	newPartialsRight = partials;
}
   
``` 

## Avoid indirection when possible. 

re-assigning variables when no transformation is needed creates indirection and is prone to induce errors later on.

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
