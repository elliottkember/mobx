# Autorun

`mobx.autorun` can be used to create a function that runs whenever any of its dependencies change. It's useful for bridging from reactive to imperative code - for example, logging, persistence or code that updates a UI.

An `autorun` function always runs and cannot be observed - by contrast, `computed` functions only run when they have observers.

As a rule of thumb: use `autorun` if you have a function that should run automatically, but which doesn't calculate or return anything. Use `computed` for everything else; autoruns are about initiating _effects_, not about producing new values.
If a string is passed as first argument to `autorun`, it will be used as debug name.

The function passed to autorun will receive one argument when invoked, the current reaction (autorun), which can be used to dispose of the autorun during execution. `autorun` also returns this function, so you can dispose of the autorun.

Just like the [`@observer` decorator/function](./observer-component.md), `autorun` will only observe data that is used during the execution of the provided function.

```javascript
var numbers = observable([1,2,3]);
var sum = computed(() => numbers.reduce((a, b) => a + b, 0));

var dispose = autorun(() => console.log(sum.get()));
// prints '6'
numbers.push(4);
// prints '10'

dispose();
numbers.push(5);
// won't print anything, nor is `sum` re-evaluated
```

## Error handling

Exceptions thrown in autorun and all other types of reactions are caught and logged to the console, but are not propagated back to the original code. This is to make sure that an exception in one reaction does not prevent the scheduled execution of other reactions.
This also allows reactions to recover from exceptions; throwing an exception does not break the tracking done by MobX,
so the reaction may complete normally the next time it is run if whatever caused the exception is fixed by then.

However, you can override the default logging behavior of Reactions by calling the `onError` handler on the disposer of the reaction.
Example:

```javascript
const age = observable(10)
const dispose = autorun(() => {
    if (age.get() < 0)
        throw new Error("Age should not be negative")
    console.log("Age", age.get())
})

age.set(18)  // Logs: Age 18
age.set(-10) // Logs "Error in reaction .... Age should not be negative
age.set(5)   // Recovered, logs Age 5

dispose.onError(e => {
    window.alert("Please enter a valid age")
})

age.set(-5)  // Shows alert box
```

A global onError handler can be set as well through `extras.onReactionError(handler)`. This can be useful in tests or for monitoring.
