# higher-order-components.md

Welcome to our deep dive into Higher-Order Components (HOCs)! Think of HOCs as special wrappers that can add new features to your existing components, similar to how you might wrap a gift - the original item stays the same, but you've added a new layer that changes how it looks or behaves.

### Understanding Higher-Order Components

A Higher-Order Component is a function that takes a component and returns a new component with additional props or behavior. It's a pattern derived from React's compositional nature, allowing us to reuse component logic across our application.

Let's start with a simple example:

```javascript
import React from "react";

// This is a Higher-Order Component
const withLogging = (WrappedComponent) => {
  // Return a new component
  return function LoggingComponent(props) {
    // Log when component renders
    console.log("Rendering:", WrappedComponent.name);

    // Pass through all props
    return <WrappedComponent {...props} />;
  };
};

// A regular component
const Button = ({ onClick, children }) => (
  <button onClick={onClick}>{children}</button>
);

// Create an enhanced version of Button
const LoggedButton = withLogging(Button);

// Usage
const App = () => (
  <LoggedButton onClick={() => alert("Clicked!")}>Click Me</LoggedButton>
);
```

### Real-World Example: Loading States

One common use case for HOCs is adding loading states to components that fetch data:

```javascript
const withLoadingState = (WrappedComponent) => {
  return function WithLoadingComponent({ isLoading, ...props }) {
    // Show loading spinner if loading
    if (isLoading) {
      return (
        <div className="loading-container">
          <div className="spinner" />
          <p>Loading...</p>
        </div>
      );
    }

    // Otherwise, render the wrapped component
    return <WrappedComponent {...props} />;
  };
};

// Basic component
const UserProfile = ({ user }) => (
  <div className="user-profile">
    <h2>{user.name}</h2>
    <p>{user.email}</p>
  </div>
);

// Enhanced component with loading state
const UserProfileWithLoading = withLoadingState(UserProfile);

// Usage
const UserPage = () => {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    fetchUser().then((data) => {
      setUser(data);
      setIsLoading(false);
    });
  }, []);

  return <UserProfileWithLoading isLoading={isLoading} user={user} />;
};
```

### Advanced HOC Patterns

#### 1. Composing Multiple HOCs

We can stack multiple HOCs to combine their functionality:

```javascript
const withAuth = (WrappedComponent) => {
  return function WithAuthComponent(props) {
    const { isAuthenticated, user } = useAuth();

    if (!isAuthenticated) {
      return <Navigate to="/login" />;
    }

    return <WrappedComponent {...props} user={user} />;
  };
};

const withErrorBoundary = (WrappedComponent) => {
  return class WithErrorBoundary extends React.Component {
    state = { hasError: false };

    static getDerivedStateFromError(error) {
      return { hasError: true };
    }

    render() {
      if (this.state.hasError) {
        return <ErrorDisplay />;
      }

      return <WrappedComponent {...this.props} />;
    }
  };
};

// Compose multiple HOCs
const enhance = compose(withAuth, withErrorBoundary, withLoadingState);

// Create an enhanced component
const EnhancedUserProfile = enhance(UserProfile);
```

#### 2. Configurable HOCs

We can make our HOCs more flexible by accepting configuration options:

```javascript
const withData = (fetchFunction, options = {}) => {
  return function WithDataComponent(WrappedComponent) {
    return function DataComponent(props) {
      const [data, setData] = useState(null);
      const [loading, setLoading] = useState(true);
      const [error, setError] = useState(null);

      useEffect(() => {
        const fetchData = async () => {
          try {
            setLoading(true);
            const result = await fetchFunction(props);
            setData(result);
            setError(null);
          } catch (err) {
            setError(err);
          } finally {
            setLoading(false);
          }
        };

        fetchData();
      }, [props]);

      const loadingComponent = options.loadingComponent || (
        <div>Loading...</div>
      );
      const errorComponent = options.errorComponent || (
        <div>Error: {error?.message}</div>
      );

      if (loading) return loadingComponent;
      if (error) return errorComponent;

      return <WrappedComponent {...props} data={data} />;
    };
  };
};

// Usage
const fetchUserData = (props) =>
  fetch(`/api/users/${props.userId}`).then((r) => r.json());

const UserProfileWithData = withData(fetchUserData, {
  loadingComponent: <CustomSpinner />,
  errorComponent: <CustomError />,
})(UserProfile);
```

#### 3. HOCs with Render Props

We can combine HOCs with render props for maximum flexibility:

```javascript
const withWindowSize = (WrappedComponent) => {
  return function WithWindowSizeComponent(props) {
    const [size, setSize] = useState({
      width: window.innerWidth,
      height: window.innerHeight,
    });

    useEffect(() => {
      const handleResize = () => {
        setSize({
          width: window.innerWidth,
          height: window.innerHeight,
        });
      };

      window.addEventListener("resize", handleResize);
      return () => window.removeEventListener("resize", handleResize);
    }, []);

    // Allow both normal rendering and render prop pattern
    if (typeof WrappedComponent === "function") {
      return WrappedComponent({ ...props, windowSize: size });
    }

    return <WrappedComponent {...props} windowSize={size} />;
  };
};

// Use as HOC
const ResponsiveComponent = withWindowSize(({ windowSize }) => (
  <div>
    Window size: {windowSize.width}x{windowSize.height}
  </div>
));

// Use as render prop
const App = () => (
  <WithWindowSize>
    {({ windowSize }) => (
      <div>
        Window size: {windowSize.width}x{windowSize.height}
      </div>
    )}
  </WithWindowSize>
);
```

### Best Practices and Tips

1. **Copy Static Methods**

```javascript
const hoistNonReactStatics = require("hoist-non-react-statics");

const withHOC = (WrappedComponent) => {
  function WithHOC(props) {
    // HOC implementation
  }

  // Copy static methods
  return hoistNonReactStatics(WithHOC, WrappedComponent);
};
```

2. **Pass Through Refs**

```javascript
const withRef = (WrappedComponent) => {
  return React.forwardRef((props, ref) => {
    return <WrappedComponent {...props} forwardedRef={ref} />;
  });
};
```

3. **Display Names for Debugging**

```javascript
const withHOC = (WrappedComponent) => {
  function WithHOC(props) {
    // HOC implementation
  }

  // Set a display name for dev tools
  WithHOC.displayName = `WithHOC(${
    WrappedComponent.displayName || WrappedComponent.name || "Component"
  })`;

  return WithHOC;
};
```

4. **Avoid Using HOCs Inside Render**

```javascript
// ❌ Bad: Creates new component on every render
render() {
  const EnhancedComponent = withHOC(MyComponent);
  return <EnhancedComponent />;
}

// ✅ Good: Define enhanced component outside
const EnhancedComponent = withHOC(MyComponent);
render() {
  return <EnhancedComponent />;
}
```

Want to explore any of these concepts further or see more specific examples? Let me know!
