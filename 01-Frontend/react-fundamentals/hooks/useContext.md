# useContext.md

Welcome to our deep dive into React's useContext hook! Imagine your React application as a big family tree, where components are family members who need to share information. While props work like passing notes one person at a time, useContext is like making an announcement that everyone in the family can hear, no matter where they are in the tree.

### Understanding Context: The Big Picture

Context provides a way to pass data through the component tree without having to pass props manually at every level. Think of it as creating a "sphere of influence" where certain data is available to any component that needs it.

Let's start with a simple example:

```javascript
import React, { createContext, useContext, useState } from "react";

// Create a context for theme settings
const ThemeContext = createContext();

// Create a provider component that will wrap our app
const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState("light");

  const toggleTheme = () => {
    setTheme((prev) => (prev === "light" ? "dark" : "light"));
  };

  // The value prop contains everything we want to share
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

// A component that uses the theme context
const ThemedButton = () => {
  // Access the context value
  const { theme, toggleTheme } = useContext(ThemeContext);

  return (
    <button
      onClick={toggleTheme}
      style={{
        background: theme === "light" ? "#fff" : "#333",
        color: theme === "light" ? "#333" : "#fff",
      }}
    >
      Current theme: {theme}
    </button>
  );
};

// Using it all together
const App = () => {
  return (
    <ThemeProvider>
      <div>
        <h1>My App</h1>
        <ThemedButton />
      </div>
    </ThemeProvider>
  );
};
```

### Creating Well-Structured Contexts

As your application grows, you'll want to organize your contexts carefully. Here's a pattern for creating maintainable context structures:

```javascript
// userContext.js
import { createContext, useContext, useState } from "react";

// 1. Create the context with a default value
const UserContext = createContext({
  user: null,
  login: () => {},
  logout: () => {},
  updateProfile: () => {},
});

// 2. Create a custom hook for using this context
export const useUser = () => {
  const context = useContext(UserContext);
  if (!context) {
    throw new Error("useUser must be used within a UserProvider");
  }
  return context;
};

// 3. Create the provider component
export const UserProvider = ({ children }) => {
  const [user, setUser] = useState(null);

  const login = async (credentials) => {
    try {
      const userData = await authService.login(credentials);
      setUser(userData);
    } catch (error) {
      console.error("Login failed:", error);
      throw error;
    }
  };

  const logout = () => {
    setUser(null);
    authService.logout();
  };

  const updateProfile = async (updates) => {
    try {
      const updatedUser = await userService.updateProfile(updates);
      setUser(updatedUser);
    } catch (error) {
      console.error("Profile update failed:", error);
      throw error;
    }
  };

  return (
    <UserContext.Provider value={{ user, login, logout, updateProfile }}>
      {children}
    </UserContext.Provider>
  );
};
```

### Advanced Patterns with Context

#### 1. Multiple Contexts Working Together

Sometimes you need multiple contexts to work together. Here's how to organize that:

```javascript
// Create a combined provider for all app contexts
const AppProviders = ({ children }) => {
  return (
    <ThemeProvider>
      <UserProvider>
        <NotificationProvider>
          <SettingsProvider>{children}</SettingsProvider>
        </NotificationProvider>
      </UserProvider>
    </ThemeProvider>
  );
};

// Create a custom hook that combines multiple contexts
const useApp = () => {
  const theme = useContext(ThemeContext);
  const user = useContext(UserContext);
  const notifications = useContext(NotificationContext);
  const settings = useContext(SettingsContext);

  return {
    theme,
    user,
    notifications,
    settings,
  };
};
```

#### 2. Context with Reducers

For complex state management, you can combine context with useReducer:

```javascript
import { createContext, useContext, useReducer } from "react";

// Define action types
const ACTIONS = {
  ADD_TODO: "ADD_TODO",
  TOGGLE_TODO: "TOGGLE_TODO",
  DELETE_TODO: "DELETE_TODO",
};

// Create reducer
const todoReducer = (state, action) => {
  switch (action.type) {
    case ACTIONS.ADD_TODO:
      return [
        ...state,
        {
          id: Date.now(),
          text: action.payload,
          completed: false,
        },
      ];
    case ACTIONS.TOGGLE_TODO:
      return state.map((todo) =>
        todo.id === action.payload
          ? { ...todo, completed: !todo.completed }
          : todo
      );
    case ACTIONS.DELETE_TODO:
      return state.filter((todo) => todo.id !== action.payload);
    default:
      return state;
  }
};

// Create context
const TodoContext = createContext();

// Create provider
export const TodoProvider = ({ children }) => {
  const [todos, dispatch] = useReducer(todoReducer, []);

  return (
    <TodoContext.Provider value={{ todos, dispatch }}>
      {children}
    </TodoContext.Provider>
  );
};

// Create custom hook
export const useTodos = () => {
  const context = useContext(TodoContext);
  if (!context) {
    throw new Error("useTodos must be used within a TodoProvider");
  }
  return context;
};
```

#### 3. Context with TypeScript

Using Context with TypeScript adds type safety:

```typescript
// Define types for context value
interface ThemeContextType {
  theme: "light" | "dark";
  toggleTheme: () => void;
}

// Create context with type
const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

// Create custom hook with type safety
const useTheme = (): ThemeContextType => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error("useTheme must be used within a ThemeProvider");
  }
  return context;
};
```

### Performance Optimization with Context

Context can trigger re-renders, so it's important to optimize its usage:

```javascript
// 1. Split contexts by update frequency
const UserDataProvider = ({ children }) => {
  const [userData, setUserData] = useState(null);

  return (
    <UserDataContext.Provider value={userData}>
      <UserActionsContext.Provider value={{ setUserData }}>
        {children}
      </UserActionsContext.Provider>
    </UserDataContext.Provider>
  );
};

// 2. Memoize context values
const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState("light");

  const value = useMemo(
    () => ({
      theme,
      toggleTheme: () => setTheme((t) => (t === "light" ? "dark" : "light")),
    }),
    [theme]
  );

  return (
    <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>
  );
};

// 3. Use context selectively
const UserProfile = () => {
  // Only subscribe to what you need
  const { name, avatar } = useUser();
  return (
    <div>
      <img src={avatar} alt={name} />
      <span>{name}</span>
    </div>
  );
};
```

### Common Patterns and Best Practices

1. **Context Composition**
   Create specialized contexts for different concerns:

```javascript
// Create focused contexts
const AuthContext = createContext();
const ThemeContext = createContext();
const FeatureFlagsContext = createContext();

// Combine them in a provider component
const AppProvider = ({ children }) => {
  return (
    <AuthContext.Provider value={authValue}>
      <ThemeContext.Provider value={themeValue}>
        <FeatureFlagsContext.Provider value={featureFlags}>
          {children}
        </FeatureFlagsContext.Provider>
      </ThemeContext.Provider>
    </AuthContext.Provider>
  );
};
```

2. **Default Values**
   Provide meaningful default values for your contexts:

```javascript
const defaultSettings = {
  theme: "light",
  language: "en",
  notifications: true,
};

const SettingsContext = createContext(defaultSettings);
```

3. **Error Boundaries with Context**
   Handle errors gracefully in your context providers:

```javascript
class ContextErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return <div>Something went wrong with the context.</div>;
    }
    return this.props.children;
  }
}

const SafeProvider = ({ children }) => {
  return (
    <ContextErrorBoundary>
      <MyContextProvider>{children}</MyContextProvider>
    </ContextErrorBoundary>
  );
};
```

Want to explore any of these concepts further or see more specific examples? Let me know!
