# render-props.md

Welcome to our deep dive into the render props pattern! Imagine you're building with LEGO blocks. Sometimes you want to build something that looks different but works the same way underneath. That's exactly what render props help us do - they let us reuse component logic while giving us complete control over how it looks.

### Understanding Render Props: The Core Concept

A render prop is a function prop that tells a component what to render. Instead of a component deciding exactly how to display its data, it delegates that responsibility to whatever component is using it. Let's start with a simple example:

```javascript
import React, { useState } from "react";

const Counter = ({ render }) => {
  const [count, setCount] = useState(0);

  const increment = () => setCount(count + 1);
  const decrement = () => setCount(count - 1);

  // Instead of rendering directly, we call the render prop
  return render({ count, increment, decrement });
};

// Now we can use Counter in different ways:
const App = () => {
  return (
    <div>
      {/* Render as buttons */}
      <Counter
        render={({ count, increment, decrement }) => (
          <div>
            <button onClick={decrement}>-</button>
            <span>{count}</span>
            <button onClick={increment}>+</button>
          </div>
        )}
      />

      {/* Same counter, completely different look */}
      <Counter
        render={({ count, increment }) => (
          <div className="fancy-counter">
            <h2>Count: {count}</h2>
            <button onClick={increment}>Add One More!</button>
          </div>
        )}
      />
    </div>
  );
};
```

### Real-World Example: Mouse Position Tracking

Let's build a more practical example that tracks mouse position. This is a classic use case for render props because different components might want to use mouse position data in different ways:

```javascript
const MouseTracker = ({ render }) => {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleMouseMove = (event) => {
      setPosition({
        x: event.clientX,
        y: event.clientY,
      });
    };

    window.addEventListener("mousemove", handleMouseMove);

    return () => {
      window.removeEventListener("mousemove", handleMouseMove);
    };
  }, []);

  // Provide position data to render prop
  return render(position);
};

// Now we can use it in multiple ways:
const App = () => {
  return (
    <div>
      {/* As coordinates */}
      <MouseTracker
        render={({ x, y }) => (
          <div>
            Mouse position: ({x}, {y})
          </div>
        )}
      />

      {/* As a following element */}
      <MouseTracker
        render={({ x, y }) => (
          <div
            style={{
              position: "fixed",
              left: x,
              top: y,
              width: 20,
              height: 20,
              background: "red",
              borderRadius: "50%",
              transform: "translate(-50%, -50%)",
            }}
          />
        )}
      />
    </div>
  );
};
```

### Advanced Pattern: Data Fetching with Render Props

Here's how we can create a reusable data fetching component using render props:

```javascript
const DataFetcher = ({ url, render }) => {
  const [state, setState] = useState({
    data: null,
    loading: true,
    error: null,
  });

  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await fetch(url);
        if (!response.ok) {
          throw new Error("Network response was not ok");
        }
        const data = await response.json();
        setState({ data, loading: false, error: null });
      } catch (error) {
        setState({ data: null, loading: false, error });
      }
    };

    fetchData();
  }, [url]);

  return render(state);
};

// Using it for different types of data:
const UserProfile = ({ userId }) => {
  return (
    <DataFetcher
      url={`/api/users/${userId}`}
      render={({ data, loading, error }) => {
        if (loading) return <div>Loading...</div>;
        if (error) return <div>Error: {error.message}</div>;
        if (!data) return <div>No user found</div>;

        return (
          <div className="user-profile">
            <h2>{data.name}</h2>
            <p>{data.email}</p>
          </div>
        );
      }}
    />
  );
};

const ProductList = () => {
  return (
    <DataFetcher
      url="/api/products"
      render={({ data, loading, error }) => {
        if (loading) return <ProductSkeleton />;
        if (error) return <ErrorBoundary error={error} />;
        if (!data) return <EmptyState />;

        return (
          <div className="product-grid">
            {data.map((product) => (
              <ProductCard key={product.id} product={product} />
            ))}
          </div>
        );
      }}
    />
  );
};
```

### Alternative Syntax: Children as a Function

Instead of using a render prop, we can use children as a function. This is functionally equivalent but can look cleaner:

```javascript
const Toggle = ({ children }) => {
  const [on, setOn] = useState(false);

  const toggle = () => setOn(!on);

  // Pass state and handlers to children function
  return children({ on, toggle });
};

// Using it:
const App = () => {
  return (
    <Toggle>
      {({ on, toggle }) => (
        <div>
          {on ? "The light is on!" : "The light is off!"}
          <button onClick={toggle}>{on ? "Turn off" : "Turn on"}</button>
        </div>
      )}
    </Toggle>
  );
};
```

### Advanced Usage: Composition of Render Props

We can compose multiple render prop components to build complex functionality:

```javascript
const ComposeRenderProps = () => {
  return (
    <MouseTracker>
      {(mousePosition) => (
        <WindowSize>
          {(windowSize) => (
            <DataFetcher url="/api/theme">
              {({ data: theme }) => (
                <div>
                  Mouse: ({mousePosition.x}, {mousePosition.y})
                  <br />
                  Window: {windowSize.width}x{windowSize.height}
                  <br />
                  Theme: {theme}
                </div>
              )}
            </DataFetcher>
          )}
        </WindowSize>
      )}
    </MouseTracker>
  );
};
```

### Best Practices and Tips

1. **Use TypeScript for Better Type Safety**

```typescript
interface RenderProps {
  count: number;
  increment: () => void;
  decrement: () => void;
}

interface CounterProps {
  render: (props: RenderProps) => React.ReactNode;
}

const Counter: React.FC<CounterProps> = ({ render }) => {
  // Implementation...
};
```

2. **Avoid Unnecessary Re-renders**

```javascript
const OptimizedMouseTracker = ({ render }) => {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  // Memoize the event handler
  const handleMouseMove = useCallback((event) => {
    setPosition({
      x: event.clientX,
      y: event.clientY,
    });
  }, []);

  useEffect(() => {
    window.addEventListener("mousemove", handleMouseMove);
    return () => window.removeEventListener("mousemove", handleMouseMove);
  }, [handleMouseMove]);

  // Memoize the rendered content
  return useMemo(() => render(position), [position, render]);
};
```

3. **Handle Edge Cases**

```javascript
const SafeRenderProp = ({ render, fallback = null }) => {
  if (typeof render !== "function") {
    console.warn("Render prop must be a function");
    return fallback;
  }

  try {
    return render();
  } catch (error) {
    console.error("Error in render prop:", error);
    return fallback;
  }
};
```

Want to explore any of these concepts further or see more specific examples? Let me know!
