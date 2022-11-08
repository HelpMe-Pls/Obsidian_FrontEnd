### Data flow
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

  - Best practices when using RTK:
    - Put Redux logic in single-file "slices" per feature
    - Put as much logic as possible in reducers
    - Model actions as "events", not "setters"