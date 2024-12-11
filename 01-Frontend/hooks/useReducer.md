# useReducer.md

Welcome to your deep dive into React's useReducer hook! Think of useReducer as a specialized tool for managing complex state in your applications - like having a skilled traffic controller directing all the changes in your component's data. Let's explore how this powerful hook can help you write more predictable and maintainable code.

### Understanding useReducer: The Core Concept

At its heart, useReducer follows a simple pattern: when something happens (an action), we need to update our state in a specific way (reduction). Imagine you're managing a bank account - every transaction (action) changes your balance (state) according to certain rules (reducer).

Let's start with a basic example:

```javascript
import { useReducer } from "react";

// The reducer function defines how state changes in response to actions
const counterReducer = (state, action) => {
  switch (action.type) {
    case "INCREMENT":
      return state + 1;
    case "DECREMENT":
      return state - 1;
    case "RESET":
      return 0;
    default:
      return state;
  }
};

const Counter = () => {
  // useReducer takes a reducer function and initial state
  const [count, dispatch] = useReducer(counterReducer, 0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => dispatch({ type: "INCREMENT" })}>Increment</button>
      <button onClick={() => dispatch({ type: "DECREMENT" })}>Decrement</button>
      <button onClick={() => dispatch({ type: "RESET" })}>Reset</button>
    </div>
  );
};
```

### Working with Complex State

Real applications often need to manage more complex state. Here's how to handle objects and arrays:

```javascript
const initialState = {
  user: null,
  cart: [],
  isLoading: false,
  error: null,
};

const shopReducer = (state, action) => {
  switch (action.type) {
    case "SET_USER":
      return {
        ...state,
        user: action.payload,
      };

    case "ADD_TO_CART":
      return {
        ...state,
        cart: [...state.cart, action.payload],
      };

    case "REMOVE_FROM_CART":
      return {
        ...state,
        cart: state.cart.filter((item) => item.id !== action.payload),
      };

    case "SET_LOADING":
      return {
        ...state,
        isLoading: action.payload,
      };

    case "SET_ERROR":
      return {
        ...state,
        error: action.payload,
      };

    case "CLEAR_CART":
      return {
        ...state,
        cart: [],
      };

    default:
      return state;
  }
};

const ShoppingCart = () => {
  const [state, dispatch] = useReducer(shopReducer, initialState);

  const addItem = (item) => {
    dispatch({ type: "ADD_TO_CART", payload: item });
  };

  const removeItem = (itemId) => {
    dispatch({ type: "REMOVE_FROM_CART", payload: itemId });
  };

  // Component rendering...
};
```

### Advanced Patterns with useReducer

#### 1. Using TypeScript with useReducer

Adding type safety to your reducers:

```typescript
// Define types for state and actions
type State = {
  count: number;
  step: number;
  error: string | null;
};

type Action =
  | { type: "INCREMENT" }
  | { type: "DECREMENT" }
  | { type: "SET_STEP"; payload: number }
  | { type: "SET_ERROR"; payload: string };

const initialState: State = {
  count: 0,
  step: 1,
  error: null,
};

const reducer = (state: State, action: Action): State => {
  switch (action.type) {
    case "INCREMENT":
      return {
        ...state,
        count: state.count + state.step,
      };

    case "DECREMENT":
      return {
        ...state,
        count: state.count - state.step,
      };

    case "SET_STEP":
      return {
        ...state,
        step: action.payload,
      };

    case "SET_ERROR":
      return {
        ...state,
        error: action.payload,
      };

    default:
      return state;
  }
};
```

#### 2. Combining useReducer with useContext

Create a global state management solution:

```javascript
import { createContext, useContext, useReducer } from "react";

const StateContext = createContext();
const DispatchContext = createContext();

const AppProvider = ({ children }) => {
  const [state, dispatch] = useReducer(appReducer, initialState);

  return (
    <StateContext.Provider value={state}>
      <DispatchContext.Provider value={dispatch}>
        {children}
      </DispatchContext.Provider>
    </StateContext.Provider>
  );
};

// Custom hooks for accessing state and dispatch
const useState = () => {
  const context = useContext(StateContext);
  if (context === undefined) {
    throw new Error("useState must be used within AppProvider");
  }
  return context;
};

const useDispatch = () => {
  const context = useContext(DispatchContext);
  if (context === undefined) {
    throw new Error("useDispatch must be used within AppProvider");
  }
  return context;
};
```

#### 3. Async Actions with useReducer

Handling asynchronous operations:

```javascript
const dataReducer = (state, action) => {
  switch (action.type) {
    case "FETCH_START":
      return {
        ...state,
        isLoading: true,
        error: null,
      };

    case "FETCH_SUCCESS":
      return {
        ...state,
        isLoading: false,
        data: action.payload,
      };

    case "FETCH_ERROR":
      return {
        ...state,
        isLoading: false,
        error: action.payload,
      };

    default:
      return state;
  }
};

const DataFetcher = () => {
  const [state, dispatch] = useReducer(dataReducer, {
    data: null,
    isLoading: false,
    error: null,
  });

  const fetchData = async () => {
    dispatch({ type: "FETCH_START" });

    try {
      const response = await fetch("https://api.example.com/data");
      const data = await response.json();
      dispatch({ type: "FETCH_SUCCESS", payload: data });
    } catch (error) {
      dispatch({ type: "FETCH_ERROR", payload: error.message });
    }
  };

  return (
    <div>
      {state.isLoading && <Spinner />}
      {state.error && <Error message={state.error} />}
      {state.data && <DataDisplay data={state.data} />}
      <button onClick={fetchData}>Fetch Data</button>
    </div>
  );
};
```

#### 4. Multi-Step Form with useReducer

Managing complex form state:

```javascript
const formReducer = (state, action) => {
  switch (action.type) {
    case "NEXT_STEP":
      return {
        ...state,
        currentStep: state.currentStep + 1,
      };

    case "PREV_STEP":
      return {
        ...state,
        currentStep: state.currentStep - 1,
      };

    case "UPDATE_FIELD":
      return {
        ...state,
        formData: {
          ...state.formData,
          [action.field]: action.value,
        },
      };

    case "SET_ERROR":
      return {
        ...state,
        errors: {
          ...state.errors,
          [action.field]: action.error,
        },
      };

    case "RESET_FORM":
      return initialState;

    default:
      return state;
  }
};

const MultiStepForm = () => {
  const [state, dispatch] = useReducer(formReducer, {
    currentStep: 1,
    formData: {},
    errors: {},
  });

  const updateField = (field, value) => {
    dispatch({
      type: "UPDATE_FIELD",
      field,
      value,
    });
  };

  const validateStep = () => {
    // Validation logic here
    const errors = {};

    if (!state.formData.email) {
      errors.email = "Email is required";
    }

    return Object.keys(errors).length === 0;
  };

  const nextStep = () => {
    if (validateStep()) {
      dispatch({ type: "NEXT_STEP" });
    }
  };

  // Render different form steps based on currentStep
  return (
    <div>
      {state.currentStep === 1 && (
        <PersonalInfoStep
          formData={state.formData}
          updateField={updateField}
          errors={state.errors}
        />
      )}
      {/* More steps... */}
    </div>
  );
};
```

### Best Practices and Tips

1. **Organize Actions**
   Create constants for action types:

```javascript
const ACTIONS = {
  INCREMENT: 'INCREMENT',
  DECREMENT: 'DECREMENT',
  RESET: 'RESET'
} as const;
```

2. **Action Creators**
   Create functions to generate actions:

```javascript
const createAction = {
  increment: () => ({ type: ACTIONS.INCREMENT }),
  decrement: () => ({ type: ACTIONS.DECREMENT }),
  reset: () => ({ type: ACTIONS.RESET }),
};
```

3. **Reducer Composition**
   Break down complex reducers:

```javascript
const userReducer = (state, action) => {
  // Handle user-related actions
};

const cartReducer = (state, action) => {
  // Handle cart-related actions
};

const mainReducer = (state, action) => {
  return {
    user: userReducer(state.user, action),
    cart: cartReducer(state.cart, action),
  };
};
```

Want to explore any of these concepts further or see more specific examples? Let me know!
