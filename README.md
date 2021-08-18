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
  const unsubscribe = store.subscribe(() =>
    console.log("Updated state", store.getState())
  );

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
  iceCream: iceCreamReducer,
}); // { cake: { noOfCakes: 10 }, iceCream: { noOfIceCreams: 20 } }

const store = createStore(rootReducer);
console.log("Initial state", store.getState());
const unsubscribe = store.subscribe(() =>
  console.log("Updated state", store.getState())
);

store.dispatch(buyCake()); // { cake: { noOfCakes: 9 }, iceCream: { noOfIceCreams: 20 } }
store.dispatch(buyCake()); // { cake: { noOfCakes: 8 }, iceCream: { noOfIceCreams: 20 } }
store.dispatch(buyCake()); // { cake: { noOfCakes: 7 }, iceCream: { noOfIceCreams: 20 } }

store.dispatch(buyIceCream()); // { cake: { noOfCakes: 7 }, iceCream: { noOfIceCreams: 19 } }
store.dispatch(buyIceCream()); // { cake: { noOfCakes: 7 }, iceCream: { noOfIceCreams: 18 } }

unsubscribe();
```

### Async Actions:

- A function that returns a promise.
- It is used to perform asynchronous actions.
- e.g. function buyCake() { return fetch("https://example.com/api/cake") }

We will use the axios and redux-thunk package.

```js
const redux = require("redux");
const thunkMiddleware = require("redux-thunk").default;
const axios = require("axios");

const createStore = redux.createStore;
const applyMiddleware = redux.applyMiddleware;

// initial state
const initialState = {
  loading: false,
  users: [],
  error: "",
};

// actions
const FETCH_USERS_REQUEST = "FETCH_USERS_REQUEST";
const FETCH_USERS_SUCCESS = "FETCH_USERS_SUCCESS";
const FETCH_USERS_FAILURE = "FETCH_USERS_FAILURE";

const fetchUsersRequest = () => {
  return {
    type: FETCH_USERS_REQUEST,
  };
};

const fetchUsersSuccess = (users) => {
  return {
    type: FETCH_USERS_SUCCESS,
    payload: users,
  };
};

const fetchUsersFailure = (error) => {
  return {
    type: FETCH_USERS_FAILURE,
    payload: error,
  };
};

const reducer = (state = initialState, action) => {
  switch (action.type) {
    case FETCH_USERS_REQUEST:
      return {
        ...state,
        loading: true,
      };

    case FETCH_USERS_SUCCESS:
      return {
        ...state,
        loading: false,
        users: action.payload,
        error: "",
      };

    case FETCH_USERS_FAILURE:
      return {
        ...state,
        loading: false,
        users: [],
        error: action.payload,
      };

    default:
      return state;
  }
};

const fetchUsers = () => {
  return function (dispatch) {
    dispatch(fetchUsersRequest());
    axios
      .get("https://jsonplaceholder.typicode.com/users")
      .then((response) => {
        // response.data is the array of users
        const users = response.data.map((user) => user.id);
        dispatch(fetchUsersSuccess(users));
      })
      .catch((error) => {
        // error.message is the error description
        dispatch(fetchUsersFailure(error.message));
      });
  };
};

// store
const store = createStore(reducer, applyMiddleware(thunkMiddleware));
store.subscribe(() => {
  console.log(store.getState());
});

store.dispatch(fetchUsers());
```
