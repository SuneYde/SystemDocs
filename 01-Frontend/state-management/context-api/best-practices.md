# Context API Best Practices

Context API is a powerful tool in React, but like any powerful tool, it requires careful consideration to use effectively. In this guide, we'll explore the best practices that will help you create maintainable, performant, and scalable applications with Context API.

### Understanding Context's Role in Your Application

Before diving into specific practices, it's important to understand when to use Context. Context is perfect for data that needs to be accessed by many components at different nesting levels, such as:

- Theme preferences
- User authentication state
- Language/localization settings
- Feature flags
- UI state that affects multiple components

However, Context isn't always the best choice for every type of state. Here's how to think about it:

```javascript
// Good use of Context - Theme settings needed everywhere
const ThemeContext = createContext({
  theme: "light",
  toggleTheme: () => {},
});

// Probably doesn't need Context - Local form state
const FormComponent = () => {
  const [formData, setFormData] = useState({});
  // Handle form locally instead of with Context
};
```

### Practice 1: Separate Concerns in Multiple Contexts

Instead of creating one large context for your entire application state, split it into logical domains. This improves maintainability and performance:

```javascript
// Don't do this:
const AppContext = createContext({
  user: null,
  theme: "light",
  notifications: [],
  cart: [],
  // Too many unrelated pieces of state
});

// Do this instead:
const UserContext = createContext({
  user: null,
  login: () => {},
  logout: () => {},
});

const ThemeContext = createContext({
  theme: "light",
  toggleTheme: () => {},
});

const NotificationContext = createContext({
  notifications: [],
  addNotification: () => {},
  removeNotification: () => {},
});
```

### Practice 2: Optimize Re-renders with Context Splitting

Context causes all consumers to re-render when its value changes. By splitting your context, you ensure components only re-render when the data they actually use changes:

```javascript
// Problematic approach - everything re-renders when any value changes
const AppContext = createContext({
  user: null,
  theme: "light",
  notifications: [],
});

// Better approach - split into focused contexts
const UserProvider = ({ children }) => {
  const [user, setUser] = useState(null);

  // Memoize the value to prevent unnecessary re-renders
  const value = useMemo(() => ({ user, setUser }), [user]);

  return <UserContext.Provider value={value}>{children}</UserContext.Provider>;
};

// Components only re-render when their specific context changes
const UserProfile = () => {
  const { user } = useContext(UserContext); // Only re-renders for user changes
  return <div>{user.name}</div>;
};

const ThemeToggle = () => {
  const { theme } = useContext(ThemeContext); // Only re-renders for theme changes
  return <button>{theme}</button>;
};
```

### Practice 3: Create Custom Hooks for Context Consumption

Always wrap context usage in custom hooks. This provides better error handling, type safety, and a cleaner API:

```javascript
// Create a custom hook
const useUser = () => {
  const context = useContext(UserContext);
  if (context === undefined) {
    throw new Error("useUser must be used within a UserProvider");
  }
  return context;
};

// Use computed values in your hook
const useTheme = () => {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error("useTheme must be used within a ThemeProvider");
  }

  // Add computed values
  const isDarkMode = context.theme === "dark";
  const colors = isDarkMode ? darkColors : lightColors;

  return {
    ...context,
    isDarkMode,
    colors,
  };
};
```

### Practice 4: Implement Proper Error Boundaries

When using Context, implement error boundaries to gracefully handle failures:

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

// Use it to wrap your providers
const SafeProvider = ({ children }) => {
  return (
    <ContextErrorBoundary>
      <AppProviders>{children}</AppProviders>
    </ContextErrorBoundary>
  );
};
```

### Practice 5: Handle Asynchronous State Updates

When your context includes asynchronous operations, handle loading and error states properly:

```javascript
const DataContext = createContext();

const DataProvider = ({ children }) => {
  const [state, setState] = useState({
    data: null,
    loading: false,
    error: null,
  });

  const fetchData = async () => {
    setState((prev) => ({ ...prev, loading: true }));
    try {
      const data = await api.getData();
      setState({ data, loading: false, error: null });
    } catch (error) {
      setState({ data: null, loading: false, error });
    }
  };

  useEffect(() => {
    fetchData();
  }, []);

  // Provide both state and actions
  return (
    <DataContext.Provider value={{ ...state, fetchData }}>
      {children}
    </DataContext.Provider>
  );
};
```

### Practice 6: Use TypeScript for Type Safety

Adding TypeScript to your contexts provides better development experience and catches errors early:

```typescript
interface User {
  id: string;
  name: string;
  email: string;
}

interface UserContextType {
  user: User | null;
  loading: boolean;
  error: Error | null;
  login: (credentials: Credentials) => Promise<void>;
  logout: () => Promise<void>;
}

const UserContext = createContext<UserContextType | undefined>(undefined);

export const useUser = (): UserContextType => {
  const context = useContext(UserContext);
  if (context === undefined) {
    throw new Error("useUser must be used within a UserProvider");
  }
  return context;
};
```

### Practice 7: Consider Performance with Context.Provider

Be careful with the value prop of Context.Provider. Memoize objects to prevent unnecessary re-renders:

```javascript
const UserProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);

  // Memoize methods to prevent unnecessary re-renders
  const login = useCallback(async (credentials) => {
    // Login logic
  }, []);

  const logout = useCallback(async () => {
    // Logout logic
  }, []);

  // Memoize the entire value object
  const value = useMemo(
    () => ({
      user,
      loading,
      login,
      logout,
    }),
    [user, loading, login, logout]
  );

  return <UserContext.Provider value={value}>{children}</UserContext.Provider>;
};
```

These practices will help you build maintainable and performant applications with Context API. Remember, the key is to find the right balance between code organization, performance, and developer experience. Need help implementing any of these practices? Let me know!
