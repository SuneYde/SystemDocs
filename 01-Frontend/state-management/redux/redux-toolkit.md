# Redux Toolkit Guide

Welcome to your comprehensive guide to Redux Toolkit! Think of Redux Toolkit as your Swiss Army knife for Redux development - it provides all the tools you need to write Redux logic more efficiently and with less boilerplate. Let's explore how it makes Redux development easier and more enjoyable.

### Understanding Redux Toolkit's Core Features

Redux Toolkit provides several essential tools that simplify Redux development:

1. `configureStore()`: Simplified store setup
2. `createSlice()`: Combines reducers and actions
3. `createAsyncThunk`: Handles async operations
4. `createEntityAdapter`: Manages normalized state

Let's explore each of these in detail:

### Configuring Your Store

```javascript
import { configureStore } from "@reduxjs/toolkit";
import userReducer from "./features/user/userSlice";
import authReducer from "./features/auth/authSlice";
import postsReducer from "./features/posts/postsSlice";

const store = configureStore({
  reducer: {
    user: userReducer,
    auth: authReducer,
    posts: postsReducer,
  },
  // Redux Toolkit includes thunk middleware by default
  // and adds development-only middleware for catching mistakes

  // Optional: customize middleware
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(yourCustomMiddleware),

  // Optional: enable Redux DevTools integration
  devTools: process.env.NODE_ENV !== "production",
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

### Creating Feature Slices

The `createSlice` function is where Redux Toolkit really shines. It combines your reducer logic and actions into a single, easy-to-manage package:

```javascript
import { createSlice, PayloadAction } from "@reduxjs/toolkit";

interface User {
  id: string;
  name: string;
  email: string;
}

interface UserState {
  currentUser: User | null;
  isLoading: boolean;
  error: string | null;
}

const initialState: UserState = {
  currentUser: null,
  isLoading: false,
  error: null,
};

const userSlice = createSlice({
  name: "user",
  initialState,
  reducers: {
    // Redux Toolkit allows us to write "mutating" logic in reducers.
    // It doesn't actually mutate the state because it uses Immer under the hood.
    setUser: (state, action: PayloadAction<User>) => {
      state.currentUser = action.payload;
      state.error = null;
    },
    clearUser: (state) => {
      state.currentUser = null;
    },
    setError: (state, action: PayloadAction<string>) => {
      state.error = action.payload;
      state.currentUser = null;
    },
  },
});

// Actions are generated automatically from reducer names
export const { setUser, clearUser, setError } = userSlice.actions;

// Export the reducer
export default userSlice.reducer;
```

### Handling Async Operations

Redux Toolkit makes handling async operations much simpler with `createAsyncThunk`:

```javascript
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import type { RootState } from '../../store';

// Create the async thunk
export const fetchUserById = createAsyncThunk(
  'users/fetchById',
  async (userId: string, { rejectWithValue, getState }) => {
    try {
      const response = await fetch(`/api/users/${userId}`);
      if (!response.ok) {
        throw new Error('User not found');
      }
      return await response.json();
    } catch (error) {
      return rejectWithValue(error.message);
    }
  },
  // Optional: condition for skipping the thunk
  {
    condition: (userId, { getState }) => {
      const { users } = getState() as RootState;
      // Don't fetch if we already have the data
      return !users.entities[userId];
    }
  }
);

const usersSlice = createSlice({
  name: 'users',
  initialState,
  reducers: {
    // Regular reducers here
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUserById.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchUserById.fulfilled, (state, action) => {
        state.loading = false;
        state.entities[action.payload.id] = action.payload;
      })
      .addCase(fetchUserById.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload as string;
      });
  }
});
```

### Working with Normalized Data

Redux Toolkit's `createEntityAdapter` helps manage normalized state:

```javascript
import { createEntityAdapter, createSlice } from "@reduxjs/toolkit";

interface Post {
  id: string;
  title: string;
  content: string;
  userId: string;
}

// Create an adapter for managing post entities
const postsAdapter =
  createEntityAdapter <
  Post >
  {
    // Optional: specify how to sort entities
    sortComparer: (a, b) => b.title.localeCompare(a.title),
  };

// Initial state is created automatically
const initialState = postsAdapter.getInitialState({
  loading: false,
  error: null,
});

const postsSlice = createSlice({
  name: "posts",
  initialState,
  reducers: {
    // Use adapter methods as reducers
    addPost: postsAdapter.addOne,
    addPosts: postsAdapter.addMany,
    updatePost: postsAdapter.updateOne,
    removePost: postsAdapter.removeOne,
  },
  extraReducers: (builder) => {
    builder.addCase(fetchPosts.fulfilled, (state, action) => {
      postsAdapter.setAll(state, action.payload);
      state.loading = false;
    });
  },
});

// The adapter provides pre-built selectors
export const {
  selectAll: selectAllPosts,
  selectById: selectPostById,
  selectIds: selectPostIds,
} = postsAdapter.getSelectors((state: RootState) => state.posts);
```

### Advanced Patterns

#### 1. Slice Factories

Create reusable slice patterns:

```javascript
function createCrudSlice<T extends { id: string }>(name: string) {
  return createSlice({
    name,
    initialState: {
      entities: {} as Record<string, T>,
      loading: false,
      error: null as string | null
    },
    reducers: {
      addEntity: (state, action: PayloadAction<T>) => {
        state.entities[action.payload.id] = action.payload;
      },
      updateEntity: (state, action: PayloadAction<Partial<T> & { id: string }>) => {
        const { id, ...changes } = action.payload;
        if (state.entities[id]) {
          state.entities[id] = { ...state.entities[id], ...changes };
        }
      },
      removeEntity: (state, action: PayloadAction<string>) => {
        delete state.entities[action.payload];
      }
    }
  });
}

// Use the factory
const postsSlice = createCrudSlice<Post>('posts');
const commentsSlice = createCrudSlice<Comment>('comments');
```

#### 2. Custom Middleware with RTK

```javascript
import { createListenerMiddleware } from "@reduxjs/toolkit";

const listenerMiddleware = createListenerMiddleware();

// Add a listener that responds to specific actions
listenerMiddleware.startListening({
  actionCreator: userSlice.actions.setUser,
  effect: async (action, listenerApi) => {
    // Save user preferences when user changes
    const user = action.payload;
    await saveUserPreferences(user.preferences);

    // You can dispatch additional actions
    listenerApi.dispatch(otherAction());
  },
});

// Add to store
const store = configureStore({
  reducer: rootReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().prepend(listenerMiddleware.middleware),
});
```

#### 3. Type-Safe Selectors

```typescript
import { createSelector } from "@reduxjs/toolkit";

interface RootState {
  posts: ReturnType<typeof postsSlice.reducer>;
  users: ReturnType<typeof usersSlice.reducer>;
}

// Create type-safe selectors
const selectPostsState = (state: RootState) => state.posts;
const selectUsersState = (state: RootState) => state.users;

export const selectPostWithAuthor = createSelector(
  [
    selectPostsState,
    selectUsersState,
    (state: RootState, postId: string) => postId,
  ],
  (posts, users, postId) => {
    const post = posts.entities[postId];
    if (!post) return null;

    return {
      ...post,
      author: users.entities[post.userId],
    };
  }
);
```

### Best Practices

1. **Organize by Feature**
   Structure your Redux code by feature rather than by type:

```typescript
features/
  ├── users/
  │   ├── usersSlice.ts
  │   ├── usersSelectors.ts
  │   └── usersHooks.ts
  ├── posts/
  │   ├── postsSlice.ts
  │   ├── postsSelectors.ts
  │   └── postsHooks.ts
```

2. **Create Custom Hooks**
   Wrap Redux logic in custom hooks:

```typescript
export function useUser(userId: string) {
  const dispatch = useAppDispatch();
  const user = useAppSelector((state) => selectUserById(state, userId));
  const status = useAppSelector((state) => state.users.status);

  const updateUser = useCallback(
    (updates: Partial<User>) => {
      dispatch(updateUserById({ id: userId, changes: updates }));
    },
    [userId, dispatch]
  );

  return {
    user,
    status,
    updateUser,
  };
}
```

3. **Maintain Type Safety**
   Use TypeScript effectively with Redux Toolkit:

```typescript
// Create typed versions of hooks
import { TypedUseSelectorHook, useDispatch, useSelector } from "react-redux";
import type { RootState, AppDispatch } from "./store";

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

Want to explore any of these concepts further or see more specific examples? Let me know!
