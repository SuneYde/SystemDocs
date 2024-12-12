# custom-hooks.md

Hey there! Let's dive into custom hooks - one of React's most powerful features for code reuse. Think of custom hooks as your own special recipes that combine React's built-in hooks to create reusable functionality.

### Understanding Custom Hooks

Custom hooks are JavaScript functions that:

1. Start with the word "use"
2. Can use other React hooks
3. Share logic between components

Let's explore some practical examples:

### 1. useLocalStorage Hook

Store and sync state with localStorage:

```javascript
import { useState, useEffect } from "react";

const useLocalStorage = (key, initialValue) => {
  // Get stored value or use initial
  const [value, setValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error("Error reading localStorage:", error);
      return initialValue;
    }
  });

  // Update localStorage when value changes
  useEffect(() => {
    try {
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error("Error writing to localStorage:", error);
    }
  }, [key, value]);

  return [value, setValue];
};

// Usage example:
const ThemeToggle = () => {
  const [isDark, setIsDark] = useLocalStorage("darkMode", false);

  return (
    <button onClick={() => setIsDark(!isDark)}>
      Switch to {isDark ? "Light" : "Dark"} mode
    </button>
  );
};
```

### 2. useFetch Hook

Handle API requests with loading and error states:

```javascript
const useFetch = (url, options = {}) => {
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    let isMounted = true;
    const controller = new AbortController();

    const fetchData = async () => {
      setLoading(true);

      try {
        const response = await fetch(url, {
          ...options,
          signal: controller.signal,
        });

        if (!response.ok) {
          throw new Error(response.statusText);
        }

        const json = await response.json();

        if (isMounted) {
          setData(json);
          setError(null);
        }
      } catch (error) {
        if (isMounted && error.name !== "AbortError") {
          setError(error.message);
          setData(null);
        }
      } finally {
        if (isMounted) {
          setLoading(false);
        }
      }
    };

    fetchData();

    return () => {
      isMounted = false;
      controller.abort();
    };
  }, [url]);

  return { data, error, loading };
};

// Usage example:
const UserProfile = ({ userId }) => {
  const {
    data: user,
    error,
    loading,
  } = useFetch(`https://api.example.com/users/${userId}`);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return <div>No user found</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
};
```

### 3. useForm Hook

Handle form state and validation:

```javascript
const useForm = (initialValues = {}, validate) => {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  // Update field value
  const handleChange = (event) => {
    const { name, value } = event.target;
    setValues((prev) => ({
      ...prev,
      [name]: value,
    }));
  };

  // Mark field as touched
  const handleBlur = (event) => {
    const { name } = event.target;
    setTouched((prev) => ({
      ...prev,
      [name]: true,
    }));
  };

  // Validate all fields
  const validateForm = () => {
    if (validate) {
      const validationErrors = validate(values);
      setErrors(validationErrors);
      return Object.keys(validationErrors).length === 0;
    }
    return true;
  };

  // Handle form submission
  const handleSubmit = async (onSubmit) => {
    setIsSubmitting(true);

    if (validateForm()) {
      await onSubmit(values);
    }

    setIsSubmitting(false);
  };

  // Reset form to initial state
  const resetForm = () => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
    setIsSubmitting(false);
  };

  return {
    values,
    errors,
    touched,
    isSubmitting,
    handleChange,
    handleBlur,
    handleSubmit,
    resetForm,
  };
};

// Usage example:
const LoginForm = () => {
  const {
    values,
    errors,
    touched,
    isSubmitting,
    handleChange,
    handleBlur,
    handleSubmit,
  } = useForm({ email: "", password: "" }, (values) => {
    const errors = {};
    if (!values.email) {
      errors.email = "Required";
    }
    if (!values.password) {
      errors.password = "Required";
    }
    return errors;
  });

  return (
    <form
      onSubmit={(e) => {
        e.preventDefault();
        handleSubmit(loginUser);
      }}
    >
      <div>
        <input
          name="email"
          value={values.email}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {touched.email && errors.email && <span>{errors.email}</span>}
      </div>
      <button type="submit" disabled={isSubmitting}>
        Log In
      </button>
    </form>
  );
};
```

### 4. useMediaQuery Hook

React to screen size changes:

```javascript
const useMediaQuery = (query) => {
  const [matches, setMatches] = useState(
    () => window.matchMedia(query).matches
  );

  useEffect(() => {
    const mediaQuery = window.matchMedia(query);

    const handler = (event) => {
      setMatches(event.matches);
    };

    mediaQuery.addListener(handler);

    return () => {
      mediaQuery.removeListener(handler);
    };
  }, [query]);

  return matches;
};

// Usage example:
const ResponsiveComponent = () => {
  const isMobile = useMediaQuery("(max-width: 768px)");
  const isTablet = useMediaQuery("(min-width: 769px) and (max-width: 1024px)");
  const isDesktop = useMediaQuery("(min-width: 1025px)");

  return (
    <div>
      {isMobile && <MobileView />}
      {isTablet && <TabletView />}
      {isDesktop && <DesktopView />}
    </div>
  );
};
```

### 5. useDebounce Hook

Debounce rapidly changing values:

```javascript
const useDebounce = (value, delay = 500) => {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(timer);
    };
  }, [value, delay]);

  return debouncedValue;
};

// Usage example:
const SearchComponent = () => {
  const [search, setSearch] = useState("");
  const debouncedSearch = useDebounce(search, 500);

  useEffect(() => {
    // Only search when debounced value changes
    if (debouncedSearch) {
      performSearch(debouncedSearch);
    }
  }, [debouncedSearch]);

  return (
    <input
      value={search}
      onChange={(e) => setSearch(e.target.value)}
      placeholder="Search..."
    />
  );
};
```

### Best Practices for Custom Hooks

1. **Naming Convention**
   Always start custom hook names with "use":

```javascript
// ✅ Good
const useWindowSize = () => {
  // hook implementation
};

// ❌ Bad
const windowSize = () => {
  // hook implementation
};
```

2. **Composition**
   Combine multiple hooks to create more complex functionality:

```javascript
const useAuthenticatedFetch = (url) => {
  const { token } = useAuth();
  const { data, error, loading } = useFetch(url, {
    headers: {
      Authorization: `Bearer ${token}`,
    },
  });

  return { data, error, loading };
};
```

3. **TypeScript Support**
   Add proper typing to your custom hooks:

```typescript
interface UseCounterProps {
  initialValue?: number;
  min?: number;
  max?: number;
}

const useCounter = ({
  initialValue = 0,
  min = -Infinity,
  max = Infinity,
}: UseCounterProps) => {
  const [count, setCount] = useState(initialValue);

  const increment = () => {
    setCount((prev) => Math.min(max, prev + 1));
  };

  const decrement = () => {
    setCount((prev) => Math.max(min, prev - 1));
  };

  return { count, increment, decrement };
};
```

Want to explore any of these patterns in more detail or see more specific examples? Let me know!
