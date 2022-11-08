- [[umi-state-flow.png |State flow]]
- Since we need **clean** UI components - with only rendering logic - i.e. they don't need to care about the miscellaneous (such as network requests). A way to extract and manage the business logic that is not related to rendering and form a separate layer is to have the ==***model***== (managed in a folder).
- Consider using the `useModel` hook if the scope of your state is not that big.
---

### Model
- A `model` is the equivalent of `slice` in `redux-toolkit`. 
- A `model` consists of:
	- `namespace`: The name of the current `model`. The `state` of the entire application, composed by the `state` of several small `model`s with `namespace` as the key
	- `state`: The current state of the `model`. The data is saved here, which directly determines the output of the view layer
	- `reducers`: An Action processor that processes *synchronous* actions and is used to calculate the *next* `state` based on the previous `state`
	- `effects`: An Action processor that handles asynchronous actions (side effects). There are 3 most common handlers for `effects`:
		- `select`: to get the `state` from any `model`
		- `call`: Executes an asynchronous function
		- `put`: similar to `dispatch`
- 