# Redux Actions and Reducers Guide

Welcome to our in-depth guide on Redux actions and reducers! Think of Redux like a well-organized kitchen: actions are like the orders coming in (what needs to happen), and reducers are like the chefs (who know how to handle those orders and update the menu board). Let's learn how to create and manage both effectively.

### Understanding Actions

Actions are plain JavaScript objects that describe what happened in your application. They're like messages that tell Redux about changes that need to occur. Every action must have a `type` property, and can optionally include additional data.

Let's start with basic action creation:

```javascript
import { createAction } from "@reduxjs/toolkit";

// Modern way using Redux Toolkit
export const addTodo = createAction("todos/add");
export const toggleTodo = createAction("todos/toggle");
export const removeTodo = createAction("todos/remove");

// Using these actions
const newTodo = addTodo({ text: "Learn Redux", id: 1 });
console.log(newTodo);
// Output: { type: 'todos/add', payload: { text: 'Learn Redux', id: 1 } }
```

### Understanding Reducers

Reducers are pure functions that take the current state and an action, and return a new state. They're like recipes that determine how your application's state should change in response to actions.

```javascript
import { createReducer } from "@reduxjs/toolkit";

// Initial state
const initialState = {
  todos: [],
  loading: false,
  error: null,
};

// Modern way using createReducer
const todosReducer = createReducer(initialState, (builder) => {
  builder
    .addCase(addTodo, (state, action) => {
      // With Redux Toolkit, we can "mutate" state directly
      // thanks to Immer under the hood
      state.todos.push(action.payload);
    })
    .addCase(toggleTodo, (state, action) => {
      const todo = state.todos.find((todo) => todo.id === action.payload);
      if (todo) {
        todo.completed = !todo.completed;
      }
    })
    .addCase(removeTodo, (state, action) => {
      state.todos = state.todos.filter((todo) => todo.id !== action.payload);
    });
});
```

### Creating Feature Slices

In modern Redux, we typically combine actions and reducers using createSlice:

```javascript
import { createSlice } from "@reduxjs/toolkit";

const todoSlice = createSlice({
  name: "todos",
  initialState: {
    list: [],
    loading: false,
    error: null,
  },
  reducers: {
    // Define reducer functions and their corresponding actions
    addTodo: (state, action) => {
      state.list.push({
        id: Date.now(),
        text: action.payload,
        completed: false,
      });
    },
    toggleTodo: (state, action) => {
      const todo = state.list.find((todo) => todo.id === action.payload);
      if (todo) {
        todo.completed = !todo.completed;
      }
    },
    removeTodo: (state, action) => {
      state.list = state.list.filter((todo) => todo.id !== action.payload);
    },
  },
  // Handle async actions
  extraReducers: (builder) => {
    builder
      .addCase(fetchTodos.pending, (state) => {
        state.loading = true;
      })
      .addCase(fetchTodos.fulfilled, (state, action) => {
        state.loading = false;
        state.list = action.payload;
      })
      .addCase(fetchTodos.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message;
      });
  },
});

// Export actions and reducer
export const { addTodo, toggleTodo, removeTodo } = todoSlice.actions;
export default todoSlice.reducer;
```

### Handling Async Actions

For asynchronous operations, we use createAsyncThunk:

```javascript
import { createAsyncThunk, createSlice } from "@reduxjs/toolkit";

// Create async thunk
export const fetchUserProfile = createAsyncThunk(
  "user/fetchProfile",
  async (userId, { rejectWithValue }) => {
    try {
      const response = await api.get(`/users/${userId}`);
      return response.data;
    } catch (error) {
      return rejectWithValue(error.response.data);
    }
  }
);

const userSlice = createSlice({
  name: "user",
  initialState: {
    profile: null,
    loading: false,
    error: null,
  },
  reducers: {
    updateProfile: (state, action) => {
      state.profile = { ...state.profile, ...action.payload };
    },
    clearProfile: (state) => {
      state.profile = null;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUserProfile.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchUserProfile.fulfilled, (state, action) => {
        state.loading = false;
        state.profile = action.payload;
      })
      .addCase(fetchUserProfile.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      });
  },
});
```

### Advanced Patterns

#### 1. Action Creators with Preparation

Sometimes you need to prepare or transform the payload before it reaches the reducer:

```javascript
const addTodoWithMetadata = createAction("todos/add", (text) => ({
  payload: {
    id: Date.now(),
    text,
    createdAt: new Date().toISOString(),
    completed: false,
  },
}));
```

#### 2. Complex State Updates

For complex state updates, consider using helper functions:

```javascript
const todoSlice = createSlice({
  name: "todos",
  initialState: {
    items: [],
    filters: {
      status: "all",
      priority: "all",
    },
  },
  reducers: {
    updateTodoAttributes: {
      reducer: (state, action) => {
        const { id, attributes } = action.payload;
        const todo = state.items.find((item) => item.id === id);
        if (todo) {
          Object.assign(todo, attributes);
        }
      },
      prepare: (id, attributes) => ({
        payload: { id, attributes },
      }),
    },
  },
});
```

#### 3. Normalized State Shape

For complex data relationships, normalize your state:

```javascript
import { createEntityAdapter } from "@reduxjs/toolkit";

const todosAdapter = createEntityAdapter({
  sortComparer: (a, b) => b.createdAt.localeCompare(a.createdAt),
});

const todoSlice = createSlice({
  name: "todos",
  initialState: todosAdapter.getInitialState({
    loading: false,
    error: null,
  }),
  reducers: {
    todoAdded: todosAdapter.addOne,
    todosReceived: todosAdapter.setAll,
    todoUpdated: todosAdapter.updateOne,
  },
});

// Use the generated selectors
export const {
  selectAll: selectAllTodos,
  selectById: selectTodoById,
  selectIds: selectTodoIds,
} = todosAdapter.getSelectors((state) => state.todos);
```

#### 4. Type-Safe Actions and Reducers

For TypeScript users:

```typescript
interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

interface TodosState {
  items: Todo[];
  loading: boolean;
  error: string | null;
}

const todoSlice = createSlice({
  name: "todos",
  initialState: {
    items: [],
    loading: false,
    error: null,
  } as TodosState,
  reducers: {
    addTodo: {
      reducer: (state, action: PayloadAction<Todo>) => {
        state.items.push(action.payload);
      },
      prepare: (text: string) => ({
        payload: {
          id: Date.now(),
          text,
          completed: false,
        },
      }),
    },
  },
});
```

### Best Practices

1. **Keep Actions Focused**
   Each action should represent one specific change:

```javascript
// ❌ Bad - Too many changes in one action
const updateTodo = createAction("todos/update", (todo) => ({
  payload: {
    ...todo,
    lastModified: Date.now(),
    status: "modified",
    version: todo.version + 1,
  },
}));

// ✅ Good - Separate actions for different concerns
const updateTodoText = createAction("todos/updateText");
const markTodoModified = createAction("todos/markModified");
const incrementTodoVersion = createAction("todos/incrementVersion");
```

2. **Use Action Types as Constants**

```javascript
// Define action types as constants
export const todoActionTypes = {
  ADD: 'todos/add',
  TOGGLE: 'todos/toggle',
  REMOVE: 'todos/remove'
} as const;

// Use in createAction
export const addTodo = createAction(todoActionTypes.ADD);
```

Want to explore any of these patterns in more detail or see more specific examples? Let me know!
