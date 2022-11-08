## [[Life cycle with hooks.png |Life cycle with hooks]]

## [Common mistakes](https://dmitripavlutin.com/react-hooks-mistakes-to-avoid/#1-do-not-change-hooks-invocation-order)
---

### `useState`
- `useState()` hook triggers a re-render, and a good practice is to set a default value for the [initial state](https://github.com/HelpMe-Pls/react-hooks/blob/extra/src/final/TS/01.tsx) so that your component is "*under control*".
- The initial value of a `useState()` hook is always _discarded_ on re-renders: it *only* has an effect when the component **_mounts_**.
- The state value might be different between 2 renders, but *remains a **constant*** inside any rendered instance and inside ***any closures*** (e.g. `useEffect()`, event handlers, *inside* any `Promise` or 
  `setTimeout()`). That's why we use the `useState()` return values as `const`:
```javascript
const [state, setState] = useState('initial')
```
 - Use [the functional updater](https://tkdodo.eu/blog/things-to-know-about-use-state#1-the-functional-updater) (in form of a *[pure function](https://beta.reactjs.org/apis/usestate#my-initializer-or-updater-function-runs-twice))* to compute new state based on previous state and prevent stale state values from closure-related issues. Also useful if you need [complex calculation](https://tkdodo.eu/blog/things-to-know-about-use-state#2-the-lazy-initializer) to initialize the state (i.e. lazy state initialization), or [avoiding repeated calculation](https://tkdodo.eu/blog/use-state-for-one-time-initializations#state-to-the-rescue) is necessary.
 - Use TypeScript's generic in case type inferrence is [not guaranteed](https://tkdodo.eu/blog/things-to-know-about-use-state#4-the-convenience-overload).
---

### Lifting state
- [[React/Performance#Where to put state |Lifting state]]: moving the **shared** state from components to their least common parent to avoid the problem of putting props into state (keep in mind the Separation of Concern in terms of performance).
- Just because you have an input doesn’t mean it needs to be controlled. You really only need a controlled input when you’re going to programmatically change the value of that input. Otherwise it’s just extra work [for no benefit](https://github.com/HelpMe-Pls/react-hooks/blob/extra/src/final/TS/03.tsx).
- Consider using the `key` prop [as an alternative](https://tkdodo.eu/blog/putting-props-to-use-state#3-fully-uncontrolled-with-a-key).
---

### How to `useState`
- **Derived** state: consider using this technique and see whether certain state variables can be [**computed on the fly**](https://github.com/HelpMe-Pls/react-hooks/blob/extra/src/final/TS/04.tsx) rather than having another state to store them, making it easier to keep your data in sync when changes occur (if computing those variables takes up a lot of resources, consider **[[React/Performance#`useMemo` - for expensive calculations |useMemo]]**. React's state should be *[a single source of truth](https://tkdodo.eu/blog/dont-over-use-state#an-example)* that you need to keep track of, while having its related values to be derived from it, which simplifies the state mutation.
- If your state updates *independently*: use separate `useStates()`s.
- For states that [updates together](https://tkdodo.eu/blog/use-state-vs-use-reducer#client-state), or only [one state's field](https://github.com/HelpMe-Pls/react-hooks/blob/master/src/final/TS/06.tsx) update at a time: use a single state object
- For state where user interactions [update different parts](https://tkdodo.eu/blog/use-state-vs-use-reducer#passing-props-to-reducers) of the state: `useReducer()`.
- __Dev tip__: when you're facing a challenging problem, try to set up the UI before adding interactivity and state management.
---

### `useEffect`
- `useEffect()` should be used to ***sync*** your state with something **_outside of React_**. Utilizing `useEffect()` to sync two (or more) _React's states_ is prone to bugs.
- [Getting initial state](https://github.com/HelpMe-Pls/react-hooks/blob/extra/src/final/TS/02.tsx) from `localStorage` and setting `localStorage` with `useEffect()`.
- A custom hook is a hook which uses default React hook(s) or other custom hook(s) inside of it.
- [Beware](https://tkdodo.eu/blog/hooks-dependencies-and-stale-closures#another-example) of stale closure. If you want to bypass the ESLINT's `react-hooks/exhaustive-deps` rule, consider the [[Design Patterns#Latest Ref |latest ref]] pattern.
- To prevent executing something inside of `useEffect()` on the first render:
```js
  const firstRender = useRef(false);
  ...
  ...
  useEffect(() => {
    if (firstRender.current) {
      // do Some;
    }

    firstRender.current = true;
  }, [doSomeDeps]);
```
---

### DOM Side-Effects
- It's a good practice to do direct DOM interactions/manipulations (with the `ref.current`) [inside](https://github.com/HelpMe-Pls/react-hooks/blob/extra/src/final/TS/05.tsx) the `useEffect()` callback.
---

### HTTP request with `useEffect`
-  `useEffect()` hook cannot return anything other than the cleanup function, so it's a good practice to extract all the async code into a utility function and then call it using the promise-based `.then()` method instead of using `async/await` syntax.
-  The difference between using `.catch()` method and adding an extra argument to `.then()` method to handle errors (see more in [Extra Credit 1](https://github.com/HelpMe-Pls/react-hooks/blob/master/src/exercise/06.md#1--handle-errors)). 
-  Combining states into a state object.
-  There **is** a logical difference between setting the state inside of a clean up function versus setting the state at the top of a `useEffect()` callback in terms of resetting the state ([see more at line 45](https://github.com/HelpMe-Pls/react-hooks/blob/master/src/final/TS/06.tsx)).
-  Resetting a child component's state by passing to it an extra prop (ideally, it should be the `key` prop) which contain the parent component's state ([see more at line 95](https://github.com/HelpMe-Pls/react-hooks/blob/master/src/final/TS/06.tsx)).
-  `ErrorBoundary` not only catches error from our defined `error` state but also catches other errors **anywhere** in their child component tree, log those errors, and display a fallback UI (defined by us).
- We can use the `ErrorBoundary` component and its props from the `react-error-boundary` library instead of writing our own `ErrorBoundary` ~~***cLaSs***~~ component.
-------------

### `useReducer`
- [How](https://codesandbox.io/s/how-usereducer-works-3bhcp) it works.
- [Type checking](https://github.com/HelpMe-Pls/advanced-react-hooks/blob/extra/src/final/TS/01.extra-3.tsx) using `typeof` keyword to support an `action` in function and object form.
- The classic [Redux `dispatch` approach](https://github.com/HelpMe-Pls/advanced-react-hooks/blob/master/src/final/TS/01.tsx) to `useReducer()`.
---

### `useCallback` - custom hooks
- [Consider](https://kentcdodds.com/blog/usememo-and-usecallback) **when** to `useCallback`.
- Wrap a function in a `useCallback()` if it's listed in `useEffect()`'s dep arr.
- [Discriminated union](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes-func.html#discriminated-unions) for typing a promise's responses (note: [this won't work (line 94)](https://github.com/HelpMe-Pls/advanced-react-hooks/blob/extra/src/final/TS/02.tsx) for destructured `state` properties).
- Avoiding *race condition* by [type checking](https://github.com/HelpMe-Pls/advanced-react-hooks/blob/extra/src/final/TS/02.tsx) the state's promise vs the action's promise.
- Abort unused requests [(at line 122)](https://github.com/HelpMe-Pls/advanced-react-hooks/blob/extra/src/final/TS/02.tsx) by using the `new AbortController()` interface.
- Preventing memory leak by [cleaning up](https://github.com/HelpMe-Pls/advanced-react-hooks/blob/master/src/exercise/02.js) the `ref` (example at the `useSafeDispatch` definition).
- Dependency array ([at 3:00](https://epicreact.dev/modules/advanced-react-hooks/usecallback-custom-hooks-solution)).
- Spreading state object (in the custom hook defintion) and then destructuring it when we use that custom hook ([at `useAsync()` hook](https://github.com/HelpMe-Pls/advanced-react-hooks/blob/master/src/exercise/02.js)).
---

### `useContext`
- Basic implementation of `context` API.
- [Sometimes](https://github.com/HelpMe-Pls/advanced-react-hooks/blob/extra/src/final/TS/03.tsx) you don’t need context. The biggest use case for context is for libraries that need to implicitly share state between components (i.e. *[[Design Patterns#Compound Component |compound component]]*).
---

### `useLayoutEffect`
- `useLayoutEffect()` is invoked AFTER the render (DOM mutations) but BEFORE the browser paints the change. Use it if you need to mutate the DOM (making observable changes to the DOM) and/or **do need** to perform measurements that are essential to the browser paint (the appearance or placement of the UI elements on the screen). [Example](https://github.com/HelpMe-Pls/advanced-react-hooks/blob/extra/src/final/TS/04.tsx).
---

### `useImperativeHandle`
- Use `React.forwardRef` to pass a `ref` prop to a function component
- Assigning custom methods to `ref.current` by using [`useImperativeHandle`](https://github.com/HelpMe-Pls/advanced-react-hooks/blob/extra/src/final/TS/05.tsx) hook.
---

### `useDebugValue`    
- `useDebugValue` can **only** be called inside of a custom hook and is used to label custom hooks so you can easily identify them in the React DevTools.
---

### `useId`
- The `useId()` hook takes no argument and helps generate a unique stable Id (as a `string`) [on both](https://youtu.be/_vwCKV7f_eA?t=188) the client-side and server-side.
- Most common usage:
```js
function Checkbox() {
  const id = useId();
  return (
    <>
      <label htmlFor={id}>Do you like React?</label>
      <input id={id} type="checkbox" name="react"/>
    </>
  );
}

// For multiple IDs in the same component, append a suffix using the same `id`:
function NameFields() {
  const id = useId();
  return (
    <div>
      <label htmlFor={id + '-firstName'}>First Name</label>
      <div>
        <input id={id + '-firstName'} type="text" />
      </div>
      <label htmlFor={id + '-lastName'}>Last Name</label>
      <div>
        <input id={id + '-lastName'} type="text" />
      </div>
    </div>
  );
}
```
- The `useId()` hook is ==**not**== used for generating the value of the `key` prop.
---

### Custom hooks
- Are user-defined hooks which combines the functionality of one or more default React hooks (i.e. take the return value of those default hooks, do some black magic on them and finally have them as the custom hook's return value).
- Useful when you need to create a reusable logic to share between multiple components.
- [Example](https://github.com/HelpMe-Pls/react-design-patterns/blob/Custom-hooks/src/useDataSource.js).