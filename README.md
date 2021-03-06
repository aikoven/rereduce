# Rereduce


Simple reducer library for Redux. It's like Reselect but for reducers. 

By using aggressive memoization, reducers can depend on each others in an efficient way, without having to query Redux store.

It works fine with time-travel debugging and server-side rendering, because reducers remains totally stateless pure functions.

It permits to replace the imperative `waitFor` of original Flux implementation by a purely functional approach.


# API

The API is a first draft and may change in the future. Don't hesitate to discuss what the API should look like here: https://github.com/slorber/rereduce/issues/1

`createReducer( [reducerDependencies], reducerFunction)`

`[reducerDependencies]` is an optional object that looks like `{key1: reducer1, key2: reducer2}`

If dependencies are provided, `reducerFunction` will be called with a 3rd argument with the state of the dependency reducers.


### Simple Reducers (without dependencies)

Rereduce only add basic memoization on the original reducer.
You don't need to use `rereduce` for those reducers at all. But if you plan to make other reducers depend on your simple reducer, wrapping it with `rereduce` will be more CPU and memory efficient.


### Reducers with dependencies

- `reducerFunction` will be called with a 3rd argument with the state of the dependency reducers.
- The final reducer always an object. If your reducer returns a primitive value, it will be wrapped in an object having a `value` attribute
- The returned object will have a `__dependencies` attribute that you could simply ignore (don't worry, it does not consume as much additional memory as you may naively think).
- `__dependencies` is required to keep the reducer stateless.
- You should take care of the `__dependencies` attribute if you need efficient Redux store serialization/deserialization.


# Example

```javascript

import {  createReducer  } from 'rereduce'

const firstReducer =  createReducer((state = 0, action) => {
  switch (action.type) {
    case 'increment': return state + 1
    case 'decrement': return state - 1
    default: return state
  }
})

const secondReducer = createReducer({ firstReducer },
  (state = 0,action,{ firstReducer }) => firstReducer + 1
)

const rootReducer = combineReducers({ firstReducer, secondReducer })

const initialState = undefined
const stateInitialized = rootReducer(initialState,{ type: 'init' })
const stateIncremented = rootReducer(stateInitialized,{ type: 'increment' })

assert.equal(stateInitialized.firstReducer, 0)
assert.equal(stateInitialized.secondReducer.value, 1)

assert.equal(stateIncremented.firstReducer, 1)
assert.equal(stateIncremented.secondReducer.value, 2)
```


