# Context API Setup Guide

Welcome to our comprehensive guide for setting up state management with React's Context API. We'll walk through creating a robust and maintainable state management system that can scale with your application.

### Understanding Context Organization

When working with Context in larger applications, it's crucial to organize your state management in a way that's both maintainable and scalable. Let's start by creating a proper directory structure:

```javascript
src/
├── contexts/
│   ├── providers/        // Individual context providers
│   │   ├── AuthProvider.js
│   │   ├── ThemeProvider.js
│   │   └── UIProvider.js
│   ├── hooks/           // Custom hooks for accessing context
│   │   ├── useAuth.js
│   │   ├── useTheme.js
│   │   └── useUI.js
│   └── AppProviders.js  // Combined provider wrapper
```

### Creating Your First Context Provider

Let's walk through creating a complete context provider that follows best practices. We'll use authentication as an example since it's a common use case:

```javascript
// src/contexts/providers/AuthProvider.js
import React, { createContext, useState, useCallback } from "react";

// Step 1: Create the context with a meaningful default value
export const AuthContext = createContext({
  user: null,
  isLoading: false,
  error: null,
  login: async () => {},
  logout: async () => {},
  register: async () => {},
});

// Step 2: Create the provider component
export const AuthProvider = ({ children }) => {
  // Set up state for your context
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);

  // Create methods to update your state
  const login = useCallback(async (credentials) => {
    try {
      setIsLoading(true);
      setError(null);

      // Your API call here
      const response = await loginAPI(credentials);
      setUser(response.data.user);

      return response.data.user;
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setIsLoading(false);
    }
  }, []);

  const logout = useCallback(async () => {
    try {
      setIsLoading(true);
      setError(null);

      await logoutAPI();
      setUser(null);
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setIsLoading(false);
    }
  }, []);

  // Create the value object that will be shared
  const value = {
    user,
    isLoading,
    error,
    login,
    logout,
    // Computed values
    isAuthenticated: !!user,
    isAdmin: user?.role === "admin",
  };

  // Return the provider with your value
  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
};
```

### Creating Custom Hooks for Context Access

Always create custom hooks to access your context. This provides better error handling and a cleaner API:

```javascript
// src/contexts/hooks/useAuth.js
import { useContext } from "react";
import { AuthContext } from "../providers/AuthProvider";

export const useAuth = () => {
  const context = useContext(AuthContext);

  if (context === undefined) {
    throw new Error("useAuth must be used within an AuthProvider");
  }

  return context;
};
```

### Combining Multiple Providers

As your application grows, you'll likely have multiple contexts. Create a single wrapper component to combine them:

```javascript
// src/contexts/AppProviders.js
import { AuthProvider } from "./providers/AuthProvider";
import { ThemeProvider } from "./providers/ThemeProvider";
import { UIProvider } from "./providers/UIProvider";

export const AppProviders = ({ children }) => {
  return (
    <AuthProvider>
      <ThemeProvider>
        <UIProvider>{children}</UIProvider>
      </ThemeProvider>
    </AuthProvider>
  );
};
```

### Setting Up Global State Types

If you're using TypeScript, define your state types clearly:

```typescript
// src/contexts/types.ts
export interface User {
  id: string;
  email: string;
  name: string;
  role: "user" | "admin";
}

export interface AuthState {
  user: User | null;
  isLoading: boolean;
  error: string | null;
  login: (credentials: LoginCredentials) => Promise<User>;
  logout: () => Promise<void>;
  isAuthenticated: boolean;
  isAdmin: boolean;
}
```

### Implementing the Root Setup

Finally, set up your providers at the root of your application:

```javascript
// src/App.js
import { AppProviders } from "./contexts/AppProviders";

const App = () => {
  return (
    <AppProviders>
      <Router>{/* Your app content */}</Router>
    </AppProviders>
  );
};
```

### Best Practices for Context Usage

1. **Split Contexts by Domain**
   Keep your contexts focused on specific domains of your application:

```javascript
// Separate contexts for different concerns
const UserProvider = ({ children }) => {
  // User-related state and actions
};

const UIProvider = ({ children }) => {
  // UI-related state (modals, sidebars, etc.)
};

const SettingsProvider = ({ children }) => {
  // App settings and preferences
};
```

2. **Optimize Performance**
   Use context splitting and memoization to prevent unnecessary re-renders:

```javascript
const UIProvider = ({ children }) => {
  const [state, dispatch] = useReducer(uiReducer, initialState);

  // Memoize your value object
  const value = useMemo(
    () => ({
      state,
      dispatch,
    }),
    [state]
  );

  return <UIContext.Provider value={value}>{children}</UIContext.Provider>;
};
```

3. **Handle Loading and Error States**
   Always include loading and error states in your contexts:

```javascript
const DataProvider = ({ children }) => {
  const [data, setData] = useState(null);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);

  const fetchData = async () => {
    try {
      setIsLoading(true);
      setError(null);
      const result = await api.getData();
      setData(result);
    } catch (err) {
      setError(err.message);
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <DataContext.Provider value={{ data, isLoading, error, fetchData }}>
      {children}
    </DataContext.Provider>
  );
};
```

4. **Use TypeScript for Better Type Safety**
   Add proper typing to your contexts and hooks:

```typescript
interface ThemeContextType {
  theme: "light" | "dark";
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error("useTheme must be used within a ThemeProvider");
  }
  return context;
};
```

This setup provides a solid foundation for state management that can scale with your application. Need help implementing any specific part or want to explore advanced patterns? Let me know!
