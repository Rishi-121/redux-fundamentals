# Redux Fundamentals

Redux is a predictable state container for JavaScript apps.

## Architecture of Redux:

- A store that holds the state of our application.
- An action that describes the changes in the state of our application.
- A reducer which actually carries out the state transition depending on the action.

### Reducers:

- A reducer is a pure function that takes in the current state and the action and returns the next state.
- (previousState, action) => newState

### Actions:

- An action is a plain JavaScript object that represents an action in our application.
- It is a description of the changes we want to make to the state.
- e.g. { type: 'BUY_CAKE' }

### Store:

- A store holds the state of our application.
- It is the single source of truth for the current state of our application.
- Responsibilities
  - Holds application state
  - Allows access to the state via getState()
  - Allows the state to be updated via dispatch(action)
  - Registers listeners via subscribe(listener)
  - Handles unregistering of listeners via the function returned via subscribe(listener)

```js
const { createStore } = require("redux");

const BUY_CAKE = "BUY_CAKE";

// action
function buyCake() {
  return {
    type: BUY_CAKE,
  };
}

// initial state
const initialState = {
  noOfCakes: 10,
};

// reducer
const reducer = (state = initalState, action) => {
  switch (action.type) {
    case BUY_CAKE:
      return {
        ...state,
        noOfCakes: state.noOfCakes - 1,
      };
    default:
      return state;
  }

  // store
  const store = createStore(reducer);
  console.log("Initial state", store.getState());
  const unsubscribe = store.subscribe(() => console.log("Updated state", store.getState()));

  store.dispatch(buyCake()); // { noOfCakes: 0 }
  store.dispatch(buyCake()); // { noOfCakes: -1 }
  store.dispatch(buyCake()); // { noOfCakes: -2 }

  unsubscribe();
};
```

### Combine Reducers:

- A function that takes in an object whose values are reducer functions and returns a new reducer function.
- It is used to combine multiple reducer functions into one.
- e.g. combineReducers({ cake: cakeReducer, iceCream: iceCreamReducer })

```js
const { createStore, combineReducers } = require("redux");

const BUY_CAKE = "BUY_CAKE";
const BUY_ICECREAM = "BUY_ICECREAM";

// action
function buyCake() {
  return {
    type: BUY_CAKE,
  };
}

function buyIceCream() {
  return {
    type: BUY_ICECREAM,
  };
}

// inital state
const initialCakeState = {
  noOfCakes: 10,
};

const initialIceCreamState = {
  noOfIceCreams: 20,
};


// multiple reducers
const cakeReducer = (state = initialCakeState, action) => {
  switch (action.type) {
    case BUY_CAKE:
      return {
        ...state,
        noOfCakes: state.noOfCakes - 1,
      };

    default:
      return state;
  }
};

const iceCreamReducer = (state = initialIceCreamState, action) => {
  switch (action.type) {
    case BUY_ICECREAM:
      return {
        ...state,
        noOfIceCreams: state.noOfIceCreams - 1,
      };

    default:
      return state;
  }
};

const rootReducer = combineReducers({
    cake: cakeReducer,
    iceCream: iceCreamReducer
}); // { cake: { noOfCakes: 10 }, iceCream: { noOfIceCreams: 20 } }
const store = createStore(rootReducer);
console.log("Initial state", store.getState());
const unsubscribe = store.subscribe(() => console.log("Updated state", store.getState()));
store.dispatch(buyCake()); // { cake: { noOfCakes: 9 }, iceCream: { noOfIceCreams: 20 } }
store.dispatch(buyCake()); // { cake: { noOfCakes: 8 }, iceCream: { noOfIceCreams: 20 } }
store.dispatch(buyCake()); // { cake: { noOfCakes: 7 }, iceCream: { noOfIceCreams: 20 } }
store.dispatch(buyIceCream()); // { cake: { noOfCakes: 7 }, iceCream: { noOfIceCreams: 19 } }
store.dispatch(buyIceCream()); // { cake: { noOfCakes: 7 }, iceCream: { noOfIceCreams: 18 } }
unsubscribe();
```
