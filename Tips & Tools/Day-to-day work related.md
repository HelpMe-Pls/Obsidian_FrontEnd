## Example of my Fresher CV

#### What did you specifically contribute to the project and what impact did it make?
- We had a big project with many teams work together to build it. My team had 3 members, the senior frontend dev, a fullstack dev and me. We gather data and APIs from other teams to create a UI that basically is a cross platform desktop app which allow the users to manage and configure all of the sensors in an area.

- For the first month, I had a task where I need to display the list of all sensors after the user chose an area (describe the dropdown button), which requires me to have an understand on where the data comes from and the data flow in React. So, the senior frontend dev creates the table component, and my job was to pass the data to the component so it can display the data.

- Over the first month, I felt like I had a pretty good understanding of the data flow, so for the following month, I suggest the manager to give me a try on re-creating a similar table display for the configuration page (He gave me a task where I had to re-styling the dropdown menu with Material-UI, which is a plus my book to learn something new, but at that time, I see that there's more to learn about React for my suggested task).
I tried to implement everything I've learnt and for the first try, I thought I made it (like, editing a cell and it actually shows the updated value), but after a playing with it for a while, turns out that it only shows the updated value for the moment you edit it, and not actually update the value in the database. The reason for that is I didn't really understand the global state management tool called Redux that my team were using then, so what I did was just updating the state locally and didn't send the updated value to the backend.

- Show the dropdown layout:

```tsx

let networks = useSelector((store: RootState) => store.sensors.networks);

  

<FormControl className={fromSCSS}>

  <Select

    value={selectedNetwork}

    onChange={(event: React.ChangeEvent<{ value: unknown }>)

      => setSelectedNetwork(event.target.value))}

  >

    {networks.map((area, index) => (

      <MenuItem key={index}>

        {area}

      </MenuItem>

    ))}

  </Select>

</FormControl>

```

- Show the table layout:

```tsx

// <TableComponent/>, `routeToConfigPage` and `onChangeValue` are defined by senior

const sensorList = useSelector((store: RootState) => store.sensors.sensorList)

// header: get from backend's API: Network, IP, Serial Number, Status

<TableComponent

  header={state.headerTable}

  data={sensorList}

  onRowDblClick={(row) => {

    dispatch(routeToConfigPage(row.clientRowId));

  }}

  onDataCellChanged={(props) => dispatch(onChangeValue(props))}

/>
```
------------------------------------------------------------

#### Issues on the projects and how you solved them (STAR)?
- I was wanting to update the quantity of the items after the user clicked on the `+` button. I decided approach the task with `useState` like:

```tsx

const [itemQty, setItemQty] = useState(1)

  

const handleChange = async () => {

    const qty = await getStock()  // get the stock of that item from the server

    setItemQty(qty + 1)   // Update UI's state

    dispatchCart(addToCart(items[selectedItem], 1, name)) // Update server's state

}

return (

<Typography>

    {itemQty}

</Typography>

<Button onClick={() => handleChange()}>

    <Typography>

        +

    </Typography>

</Button>

)

```

- Seems to work just fine at first, but overtime, I noticed that if you try to click the `+` button rapidly (like at a double-click rate), the `itemQty` is not displayed properly. Turns out that `getStock()` takes a while to finish, so while the `getStock()` was running, if the user try to increase the `qty`, the final result will be updated from the point of the FIRST click (e.g. clicks `+` 3 times but eventually the `qty` is 2)

- After diving in the rabit hole, I found that the issue is called "stale value" which related to the closure that corresponding component. Long story short, when the component mounted (i.e. for a specific render instance of it), the functions within it has access to the variables on their same scope (i.e. `handleAddOne()` has access to the value of `itemQty` at the point of its creation). So, even if the user clicks `+` multiple times rapidly, the final value of `itemQty` will be updated from the point of `handleAddOne()` creation (i.e. `1`)

- To fix it, use the functional update to create an independant closure for the state so that React will compute new state based on `liền trước` state:

```tsx

setItemQty((qty) => qty + 1)

```

- So now the user clicks 3 times, the final value of `itemQty` will be 4 (maybe there's a bit delay between 1 and 4 but nonetheless it's still the expected result).
-------------------------------------------------------------

#### Tell me about the projects:
##### Bookshelf:
- `react-query` ? How ? What significant features ?
  - GUIDE THEM THROUGH WHAT DIFFERENCES DID IT MAKE FRIST, THEN:

  - CRUD: For example a "create" operation:

```ts

export function useCreateListItem() {

    const client = useClient()  // current user

    const queryClient = useQueryClient()    // current query session

  

    return useMutation(

      // `client` is an util which abstracts implementation details of fetching & authorization

        ({ bookId }: Pick<BookProps, "bookId">) => client(`list-items`, {

            data: { bookId }

        }),

        {  

            onError: () => queryClient.setQueryData('list-items'),    // in case optimistic update fails

            onSettled: () => queryClient.invalidateQueries('list-items'),   // optimistically refetch (prefetch) the updated list of items

        }

    )

}

```

  - Prefetch:

```tsx

// util hook:

export function useRefetchBookSearchQuery() {

    const client = useClient()

    const queryClient = useQueryClient()

    // memoize the return value to persist its closure coz it's in the `useEffect` dep list below

    return React.useCallback(

        async function refetchBookSearchQuery() {

            queryClient.removeQueries('bookSearch')

            await queryClient.prefetchQuery(

                getBookSearchConfig(queryClient, client, ''),

            )

        },

        [client, queryClient],

    )

}

  const refetchBookSearchQuery = useRefetchBookSearchQuery()

  

  /* This "clean up" effect serves its purpose to reload a new list of books

   * for the "Discovery" page (in the background)

   * right AFTER the server has updated the "Reading List"

   * (i.e. the user added/removed some books to/from the "Reading List")

   * Without this effect, the next time the user visit the "Discovery" page,

   * they will see a jump in the book list due to the effect of updating

   * the list only when they visit the "Discover" page.

   */

  React.useEffect(() => {

    // The ReturnType of useEffect() clean up function is `void`

    // and the call `refetchBookSearchQuery()` returns a promise

    // that's why we wrap it with an IIFE to please TS :))

    return () =>

      (function cleanUp() {

        refetchBookSearchQuery()

      })()

  }, [refetchBookSearchQuery])

  

```

- Render as you fetch ? Why ? How ?
  - Instead of waiting for the render to finish then fetch (fetching inside the `useEffect()`) or wait for the fetch to finish then do the render (having a "Container" (or a "parent") component that handles data fetching and conditionally renders the child presentational component once we’ve received the data), we could render and fetch at the same time: Start fetching data as early as possible and start trying to render components that may still need data

```tsx

// previously we had to render the layout then fetch the user's data inside an `useEffect()` :

async function getUser() {

  let user = null

  

  const token = await auth.getToken()

  if (token) {

    const data = await client('bootstrap', {token})

    queryClient.setQueryData('list-items', data.listItems)

    user = data.user

  }

  return user

}

  

// Inside the App:

React.useEffect(() => {

    const appDataPromise = getUser()

    // `run` is an util func which sets the state based on the promise's returned values

    run(appDataPromise)

}, [run])

  

// Now we can start fetching the user's data while the App is rendering:

// Outside the App:

const appDataPromise = getUser()

  

// Within the App:

React.useEffect(() => {

    run(appDataPromise)

}, [run])

```

- Cypress ? Why ? How ?

  - Static: catch typos and type errors as you write the code.

  - Unit: verify that individual, isolated parts works as expected (we're often testing a single function).

  - Integration: verify that several units works together in harmony (we're normally testing a single screen).

  - End to End (AKA "e2e test" or "functional testing"): a helper that behaves like an user to click around the app and verify that things function the way you want (we're putting it all together and testing the application as a whole without access to any internals, without mocking the backend,...), which is Cypress (another worthy alternative is Playwright) in this case:

```tsx

// the "role" could be specified by using the WAI-ARIA `role` attribute in HTML tags

cy.findByRole('somthing').within(()=> {/* someOtherThing */}).should('have some attributes').click() or .type(), etc...

```


##### Redux-TS:
- Redux fundamentals ? Why use it ?

  - Predictable state updates (Redux DevTools)
  - Keeping the state update logic outside the React component tree
  - Think of Redux like generic state management tool that can be used for:
    - Caching state from a server (if this is ALL you use Redux for then consider using `react-query` or `Apollo Client` instead)
    - Other complex data management on the client (if using React `useContext`, `useState`, `useReducer` is obviously insufficient at many places)
    - Keep in mind that React `Context` in and of itself is a form of Dependency Injection (*__not__* a state management system) which is scoped to some portion of your subtree, where you say “Here is a value”, and any portion of that component subtree can ask to read the value. That’s literally all it does.

  - So it’s really a question of:
    - What *specific* problems are you trying to solve?
    - What problems do these tools solve?
    - Where’s the overlap between those?

- redux-toolkit ? What's the deal ? How did it help with development ?
  - Redux data flow eli5:
    - State describes the condition of the app at a point in time, and UI renders based on that state.
    - When something happens in the app:
      1. The UI dispatches an action.
      2. The store runs the reducers (with their actions), and the state is updated based on what occurred.
      3. The store notifies the UI that the state has changed.
      4. The UI re-renders based on the new state.

  - Data flow using RTK:
1. Read data/state from the `store` using `useSelector`. Selectors will re-run whenever the Redux `store` is updated, and if the data they return has changed, the component will re-render.:

```tsx

// `useSelector` takes the store as its param and returns a specific piece of the store that the corresponding component needs:

  const post = useSelector<RootState, IPostState | undefined>((state) =>

    selectPostById(state, postId)

  )

```

2. UI `dispatch` action:

```tsx

const onUpdatePostClicked = (e: ChangeEvent<HTMLFormElement>) => {

    e.preventDefault()

    if (title && content) {

      dispatch(postUpdated({ postId, title, content }))

    }

}

  

<form onSubmit={onUpdatePostClicked}>

{/* do something */}

</form>

```

3. Use `createAsyncThunk` to `GET` or `POST` requests.

```tsx

export const addNewPost = createAsyncThunk( // We can use `createAsyncThunk` to help with sending data, not just fetching it

    'posts/addNewPost',

    // The payload creator receives the partial `{title, content, user}` object

    async (initialPost: AddPostBody) => {

        // We send the initial data to the fake API server with POST method

        const response = await client.post('/fakeApi/posts', { post: initialPost })

        // The response includes the complete post object, including unique ID

        return response.post as IPostState

    }

)

```

4. Use `createSlice` to generate `actions` and `reducers` for a specific feature:

```tsx

const postsSlice = createSlice({

    name: 'posts',  //represents action types

    initialState,

    reducers: {

        postUpdated(state, action: PayloadAction<{ postId: EntityId, title: string, content: string }>) {

            const { postId, title, content } = action.payload

  

            // the `createSlice` util performs immutable state update in the background, eventhough it seeems like we're directly modify the state here (i.e. if we didn't use RTK then we'll have to use Immer):

            const existingPost = state.posts[postId]

            if (existingPost) {

                existingPost.title = title

                existingPost.content = content

            }

        },

    },

    extraReducers: builder => {

        // Is used when a slice reducer needs to respond to other actions that weren't defined as part of its field

        // In this case, we need to listen for the "fulfilled" action type dispatched by our addNewPost thunk defined OUTSIDE the slice.

        // Use the `addOne` reducer for the fulfilled case to add one new post object to our state

        builder.addCase(addNewPost.fulfilled, state.posts.addOne)

    }

})

  

/* When we write the `postUpdate` reducer function, `createSlice` will automatically generate an "action creator" function with

the SAME name. We can export them and use it in our UI component to dispatch an action (in step 2) */

export const { postUpdated } = postsSlice.actions

  

export default postsSlice.reducer   // to be imported in the store

```

5. The `store` is the global state object which tells the selectors to re-render the components or not:

```tsx

import postsReducer from '../features/posts/postsSlice'

  

/* This tells Redux that we want our TOP-LEVEL STATE OBJECT to have a field named `posts` inside,

and all the data for `state.posts` will be updated by the `postsReducer` function when actions are dispatched (in step 2) */

export const store = configureStore({

  reducer: {

    posts: postsReducer,

  },

});

```

  - Best practices to use RTK:
    - Put Redux logic in single-file "slices" per feature
    - Put as much logic as possible in reducers
    - Model actions as "events", not "setters"

- TS ? Fundamental concepts ?
  - Using `enum` to define a set of named constants, preventing syntax errors and enhance readability.

```tsx

export enum Status {

  IDLE,

  LOADING,

  SUCCEEDED,

  FAILED

}

  

useEffect(() => {

    //only try to fetch the list of posts once

    if (postStatus === Status.IDLE) {

      dispatch(fetchPosts())

    }

}, [dispatch, postStatus])

```

  - Define the shape of an object so that we can get inferrence from the IDE: for instance, in step 4 we destructured `postId` from the `action.payload`, but if for some reason we pass it as `id` in step 2 then TS is gonna bark.


##### LOCO:

- Gatsby ? What's the point ? Why Gatsby and not Next ? JAMStack ? SSR vs SSG ?
  - **CMS** (Content Management System): backend which serves data/content and decoupled from the frontend (i.e. you can have multiple frontends like from a web app or native mobile app all using the data from the same CMS using APIs). Whenever the content is updated, the CMS will notify the SSG (via webhook) to create static pages accordingly.
  - **SSG**: at build time, create the static pages from the JAM (JS, Markup, API) and the contents (from CDN). Static pages are the SAME generated Markup pages for EVERY user, and the "dynamic" functionalities of it (like the queries from an user) comes from the "static" functions which executed at run time. In contrast, a truly dynamic site would create the webpages and everything related to that session ON DEMAND whenever an user visit the site (which results in CREATING the same layouts over and over again). Static sites are then uploaded to the CDN.
  - **CDN**: a globally distributed set of servers which allow users to connect to the fastest and closest server to them instead of having ALL of the traffic centralized to one place, which helps with performance and scalability. Then, the client will access the site from the CDN.
  - **SSR**: means using a server to generate HTML from JavaScript modules *in response* to a URL request. Contrast to client-side rendering, which uses the browser to create HTML using the DOM. SSR addresses client-side rendering drawbacks: creating HTML at run time (so the user don't have to stare at a blank page while waiting for the browser to build the DOM before any content is visible) and SEO optimization

- GraphQL ? Why not REST ?
  - GraphQL by itself is just a data transfer protocol, like: "Here’s how I format my request, here’s the shape of your response.” Gives the specificities in just one endpoint, in contrast to REST where you need multiple endpoints to get a shape of something.
  - It gives you this nice little interface where it’s just like: `useQuery` and it gives you back: “`{data, isLoading, error}`” in your components, so you can decide what to render:

```tsx

  const { data, isLoading, error } = useQuery(GET_DETAILS, {

    variables: { id }, // {id} destructured from component's prop

  })

```

  - Apollo Client is for storing cache data from the server, like if I ask for the *same* thing (at **RUN TIME**), then there's no need to re-fetch it:

```tsx

export const client = new ApolloClient({

  link: new HttpLink({

    uri: process.env.GATSBY_STRAPI_URL + "/graphql",

    fetch,

  }),

  cache: new InMemoryCache(), // so that it won't fire a new request if the user makes the same query

})

```

  - Prefer GraphQL over REST when:
    - It’s not very obvious *where* data is coming from in the templates that render the pages, so updating the data might be confusing later.
	```tsx
  const data = useStaticQuery(graphql`

    query GetPromos {

      allStrapiProduct(

        filter: { promo: { eq: true } }

        sort: { fields: category___name }

      ) {

        nodes {

          name

          strapiId

          description

          category {

            name

          }

          variants {

            images {

              localFile {

                childImageSharp {

                  gatsbyImageData

                }

              }

            }

          }

        }

      }

    }

  `)

```

- MUI ? How did it help with "rapid dev" ?
  - Use Grid API to render out styled components:

```tsx

const buttons = [

    { label: "Store", icon: store, link: "/hoodies" },

    {

      label: "More By Us",

      icon: moreByUs,

      href: "https://github.com/HelpMe-Pls",

    },

]

<Grid

      container

      justifyContent="space-around"

      classes={{ /*Override or extend the styles applied to the component */ }}

    >

      {buttons.map(button => (

        <Grid item key={button.label}>

          <Grid

            container

            direction="column"

            alignItems="center"

            justifyContent="center"

            classes={{ /*Override or extend the styles applied to the component */ }}

          >

            <Grid item>

              <img

                className={/*Override or extend the styles applied to the component */}

                src={button.icon}

                alt={button.label}

              />

            </Grid>

            <Grid item>

              <Typography

                classes={{ root: classes.label }}

                variant="h1"

              >

                {button.label}

              </Typography>

            </Grid>

          </Grid>

        </Grid>

      ))}

    </Grid>

```

- AWS hosting ?
  - EC2: is a VPS (Virtual Private Server) for hosting your backend, so that when you deploy the frontend from Netlify/Vercel, it'll connent to that EC2 instance to fully host your website
  - S3 bucket is a remote container for hosting static assets (like for instance when we deploy the website locally, the images are also stored locally within the same machine, now that we're deploying our website to the internet, we need some place to host those images so that the EC2 can access them)


##### Markdown:
- Why Electron ? Is there anything else besides Electron ?
  - Build cross-platform *desktop* apps with JavaScript, HTML, and CSS (using Chromium like VSCode, MSTeam, Twitch,...)
  - Tauri is the next big thing