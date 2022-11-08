- The mental model is that a saga is like a separate thread in your application that's solely responsible for side effects.
- `redux-saga` is a redux middleware, which means this thread can be started, paused and cancelled from the main application with normal redux actions, it has access to the full redux application state and it can dispatch redux actions as well.
- In order to run our Saga, we need to:
	-   Create a Saga middleware with a list of Sagas to run
	-   Connect the Saga middleware to the Redux store
```ts
import { createStore, applyMiddleware } from 'redux'
import createSagaMiddleware from 'redux-saga'

import { sampleSaga } from './sagas'

// Create a Saga middleware
const sagaMiddleware = createSagaMiddleware()

const store = createStore(
  reducer,
  // connect the Saga middleware to the Redux store
  applyMiddleware(sagaMiddleware)
)

// Run the sagas
sagaMiddleware.run(sampleSaga)
```