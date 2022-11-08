- An [intro](https://kentcdodds.com/blog/aha-programming) to design patterns in general (which suggests "prefer duplication over the wrong abstraction" and "optimize for change first").*
---

### Latest Ref
- This pattern allows you to access the *latest* value of a prop, state, or a callback without needing to list it in the dependency array. It's an **intentional replicate** of an old default feature of class-based React (which ***may cause*** typical asynchronous bugs in some cases, e.g: Imagine, you start the async thing, then props change while that is in flight, when your code continues you'll have the **latest** values of `this.props` rather than what you want: the values that **existed** at the time the function started running [adheres to its closure](https://epicreact.dev/how-react-uses-closures-to-avoid-bugs/)):

```js
function useExampleOne(callback) {
  React.useEffect(() => {
    callback()
  }, [callback]) // <-- have to include the callback in the dep array
}

function useExampleTwo(callback) {
  const {current: latestCallbackRef} = React.useRef(callback)

  React.useLayoutEffect(() => {
  // https://kentcdodds.com/blog/useeffect-vs-uselayouteffect#one-special-case
    latestCallbackRef = callback
  })

  React.useEffect(() => {
    latestCallbackRef()
  }, []) // <-- don't have to include the callback in the dep array
}
```
---

### Context Module Functions
- SoC and enhancing reusability, more details [at 02:20](https://epicreact.dev/modules/advanced-react-patterns/context-module-functions-solution).
- Typical use cases [at 03:25](https://epicreact.dev/modules/advanced-react-patterns/context-module-functions-solution).
---

### Layout Component 
- Design a component's props and their expressions to [mimic the UI's layout](https://github.com/HelpMe-Pls/advanced-react-patterns/blob/extra/src/final/TS/02.tsx) to be more declarative. 
- Aim to create "template" components (not containing styles) that are highly reusable by having [other components](https://github.com/HelpMe-Pls/react-design-patterns/blob/Layout-components-02/src/App.js) as their `children`, props.
- [Read more](https://epicreact.dev/one-react-mistake-thats-slowing-you-down/).
---

### Container/Compound Component
- [Abstracting away](https://github.com/HelpMe-Pls/react-design-patterns/blob/Container-components-01/src/App.js) the data/state management of child components to their parent using `React.Children` and `React.cloneElement`.
- Use `cloneElement` to maintain immutability between renders of child components.
- Notice ([at 1:05](https://epicreact.dev/modules/advanced-react-patterns/compound-components-extra-credit-solution-1)) when passing props to a non-functional component (DOM element).
- Use `context` to manage state of a compound component with multiple levels of children.
- [Read more](https://ryanflorence.dev/p/advanced-element-composition-in-react).
---

### Higher-Order Components
- They are a function which returns [another component](https://github.com/HelpMe-Pls/react-design-patterns/blob/Higher-order-Components/src/UserInfoForm.js) (i.e. takes other component as their param).
- [Example](https://github.com/HelpMe-Pls/react-design-patterns/blob/HOC%2B%2B/src/partiallyApply.js).
- A trick to capitalize prop's name [at line 34](https://github.com/HelpMe-Pls/react-design-patterns/blob/Higher-order-Components/src/withEditableResource.js).
---

### Recursive Components
- Useful when you want to display [nested data](https://github.com/HelpMe-Pls/react-design-patterns/blob/Recursive-components/src/App.js).
---

### Prop Collections and Getters
- Using `useRef` (at ` useToggle` [definition](https://github.com/HelpMe-Pls/advanced-react-patterns/blob/extra/src/final/TS/04.tsx)) to secure `initialState`.
- Returning objects of props (in the form of returned value from a function) in your custom hook/component so that later you can simply spread across the rendered UI, i.e. **composing props**. 
- See more [at 01:50](https://epicreact.dev/modules/advanced-react-patterns/prop-collections-and-getters-extra-credit-solution-1) and at the returned `onClick` function in `getTogglerProps` from [Extra Credit](https://github.com/HelpMe-Pls/advanced-react-patterns/blob/master/src/final/04.extra-1.js).
- A notice when combining props [at 0:50](https://epicreact.dev/modules/advanced-react-patterns/prop-collections-and-getters-extra-credit-solution-1).
---

### State Reducer
- Using **alias** to add extra functionality to or replacing an existing function ([example](https://github.com/HelpMe-Pls/advanced-react-patterns/blob/extra/src/final/TS/05.tsx) at `useToggle` hook call).
---

### Control props
- Controlling props by evaluating their use case through an `if` statement. [See more](https://github.com/HelpMe-Pls/advanced-react-patterns/blob/master/src/exercise/06.js) at `onIsControlled` in the `useToggle` hook. 
- `useRef` to get the [previous state](https://github.com/HelpMe-Pls/advanced-react-patterns/blob/master/src/exercise/06.js) of the component (at `useControlledSwitchWarning` hook definition).
- Set up Dev's warnings ([in custom hook](https://github.com/HelpMe-Pls/advanced-react-patterns/blob/extra/src/utils.tsx)) for the misuse of our "controlled" props.
- Using the `&` operator (i.e. `intersection`) to define a generic for a function that takes `...props` (checkout `getTogglerProps` in [06.tsx](https://github.com/HelpMe-Pls/advanced-react-patterns/blob/extra/src/final/TS/06.tsx)).