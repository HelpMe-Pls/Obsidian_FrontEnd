- Answer these questions before integrating Redux into your project: 
	- What *specific* problems are you trying to solve?
    - What problems do these tools solve?
    - Where’s the overlap between those?

- Redux is best used for:
	- Predictable state updates (Redux DevTools)
	- Keeping the state update logic *outside* the React component tree

- Redux is also capable of:
	- Caching state from a server (if this is **ALL** you use Redux for then consider using `react-query` or `Apollo Client` instead)
    - Other complex data management on the client (if using React `useContext`, `useState`, `useReducer` is *obviously* insufficient at ***many*** places)
	- Keep in mind that React `Context` in and of itself is a form of Dependency Injection (*__not__* a state management system) which is scoped to some portion of your subtree, where you say “Here is a value”, and any portion of that component subtree can ask to read the value. That’s literally all it does.

- Redux is made up of: 
	-   **_Actions_**: are plain objects with a type field, and describe *what happened* in the app.
	-   **_Reducers_**: are **functions** that return a new state value based on previous state and its action.
	-   A Redux ***store***: runs the "root" reducer whenever an action is dispatched.

- Redux data flow:
	- State describes the *condition* of the app at a point in time, and UI renders based on that state.
    - **When** something happens in the app:
      1. The UI *dispatches* an action.
      2. The store runs the reducers (with their actions), and the state is updated based on what occurred.
      3. The store notifies the UI that the state has changed.
      4. The UI re-renders to reflect the new state.
