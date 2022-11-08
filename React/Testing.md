### Intro to testing
- Use VanillaJS to prepare the DOM for your test.
- Assert the order of your elements by using Array destructuring, from that, you can get the corresponding methods of that DOM object (thanks to [TypeScript](https://github.com/HelpMe-Pls/testing-react-apps/blob/master/src/__tests__/final/TS/01.extra-1.tsx), at line `17`).
- Remember to type check `HTMLElement` in your test.
- Use `dispatchEvent()` method to fire an event that doesn't have a dedicated method (like mouse-over).
- Clean up after each test to ensure their isolation:
```ts
beforeEach(() => {
	document.body.innerHTML = ''
})
``` 
- Use `@testing-library/react` [instead of](https://github.com/HelpMe-Pls/testing-react-apps/blob/master/src/__tests__/final/TS/02.tsx) `Jest` (for abstractions like auto cleanups, auto DOM preparation,...).
---

### Avoid implementation details
- You can tell whether tests rely on implementation details if they're written in a way that would fail if the implementation changes. For example, what if we wrapped our counter component in another `div` or swapped our `children` from a `div` to a `span` or `p`.  
- [Avoid](https://epicreact.dev/modules/testing-react-apps/avoid-implementation-details-solution) implementation details by querying for and interacting with the elements [in a way](https://testing-playground.com/) that is implementation detail free (i.e. presented in isolation) and refactor friendly.
- Abstract away the implementation details of an event (e.g. `click`) by using [`userEvent`](https://epicreact.dev/modules/testing-react-apps/avoid-implementation-details-extra-credit-solution-1) from the `@testing-library/user-event`.
---

### Form testing
- Use `screen.debug()` with the `testing-playground` Chrome extension [to build your test](https://epicreact.dev/modules/testing-react-apps/form-testing-solution-1).
- Use [`jest.fn()`](https://epicreact.dev/modules/testing-react-apps/form-testing-extra-credit-solution-1) as a mock function (i.e. when you don't care what that function does or use its returned values) and assert it was called correctly rather than defining your own.
- [Generating](https://epicreact.dev/modules/testing-react-apps/form-testing-extra-credit-solution-4) test data by using [`@jackfranklin/test-data-bot`](https://www.npmjs.com/package/@jackfranklin/test-data-bot).
---

### Mocking HTTP requests
- Use `msw` ([at 01:30](https://epicreact.dev/modules/testing-react-apps/mocking-http-requests-solution-1)) as a request interceptor for testing (so that we don't have to worry about finding an available port for the server to listen to and making sure we're making requests to the right port).
- Use `msw` as a offline [module](https://epicreact.dev/modules/testing-react-apps/mocking-http-requests-extra-credit-solution-1) to write UI for APIs that aren't finished yet.
- Use [`toMatchInlineSnapshot()`](https://epicreact.dev/modules/testing-react-apps/mocking-http-requests-extra-credit-solution-3) rather than an explicit assertion on an error element to keep your tests up-to-date if the error message were to change in the future.
- Colocating run-time server behavior tests ([at 0:30](https://epicreact.dev/modules/testing-react-apps/mocking-http-requests-extra-credit-solution-4)) by using `server.use`. For such cases, remember to add `server.resetHandlers()` to preserve test isolation and restore the original handlers for other tests.
---

### Mocking Browser APIs and modules
- [Non-null assertion](https://github.com/HelpMe-Pls/testing-react-apps/blob/master/src/__tests__/final/TS/06.tsx) for a promise's return.
- Expand the [`jsdom`](https://github.com/jsdom/jsdom)'s browser environment simulation in Node by [mocking the browser's API](https://epicreact.dev/modules/testing-react-apps/mocking-browser-apis-and-modules-solution-1), which allows us to continue to test with Jest (in Node) while not actually running in a browser. The main reason for that is because the tools we currently have for testing are WAY faster and WAY more capable when run in Node.
- However, if you are testing something that **really** relies on browser APIs or layout (like drag-and-drop) then you may be better served by writing those tests in a real browser, using a tool like Cypress.
- Sometimes, the module is interacting with browser APIs that are just too hard to mock (like `canvas`) or you're comfortable relying on the module's own test suite to give you confidence that as long as you use the module properly, everything should work. In that case, it makes sense to mock the module [directly](https://epicreact.dev/modules/testing-react-apps/mocking-browser-apis-and-modules-extra-credit-solution-1) by using [`jest.mock()`](https://jestjs.io/docs/es6-class-mocks#calling-jestmock-with-the-module-factory-parameter) instead of mocking the browser's API.
---

### Testing with context and a custom render method
- Use the [`wrapper` option](https://epicreact.dev/modules/testing-react-apps/context-and-custom-render-method-solution) of the `render()` from `testing-library/react` to test a component that uses context.
- Difference between a void function and a function that returns something ([at 2:20](https://epicreact.dev/modules/testing-react-apps/context-and-custom-render-method-extra-credit-solution-2)).
- It's highly recommended to [avoid implementation details](https://epicreact.dev/modules/testing-react-apps/context-and-custom-render-method-extra-credit-solution-3) by SoC into another module.
---

### Testing custom hooks
- The easiest and most straightforward way to test a custom hook is to
create a component that uses it and then test that component instead.
- To do that, we'll **make a test component** that uses the hook in the typical way that our hook will be used and then test that component, therefore _indirectly_ testing our hook.
- When it's difficult to create a test component that resembles the **typical** way that our hook will be used, we can _isolate_ the hook by creating a `null` component that **only** uses what the hook returns. Remember to wrap an `act()` around your state update ([at 1:30](https://epicreact.dev/modules/testing-react-apps/testing-custom-hooks-extra-credit-solution-1)). 
- Use the returned `{result}` of `renderHook` (from `@testing-library/react-hooks`) to test the returned values from your custom hooks (e.g. create a test for custom hooks that allows the hook's user to [customize its `initialProps`](https://epicreact.dev/modules/testing-react-apps/testing-custom-hooks-extra-credit-solution-3) instead of using its default value).