# useEffect.md

Welcome to your deep dive into React's useEffect hook! Think of useEffect as your component's connection to the outside world - it's how your component can synchronize with things beyond React's control, like API calls, subscriptions, or browser APIs.

### Understanding useEffect: The Core Concept

At its heart, useEffect is about handling "side effects" in your components. A side effect is anything that reaches outside your component's own little world - like fetching data, manually changing the DOM, or setting up subscriptions. Let's start with a simple example:

```javascript
import React, { useEffect, useState } from "react";

const Clock = () => {
  const [time, setTime] = useState(new Date());

  useEffect(() => {
    // This effect syncs our component with the system clock
    const interval = setInterval(() => {
      setTime(new Date());
    }, 1000);

    // This cleanup function runs before the effect runs again
    // or when the component unmounts
    return () => {
      clearInterval(interval);
    };
  }, []); // Empty dependency array means "run only on mount"

  return <div>Current time: {time.toLocaleTimeString()}</div>;
};
```

### The Three Key Questions of useEffect

When working with useEffect, always ask yourself three questions:

1. What side effect am I synchronizing with?
2. What dependencies should trigger this synchronization?
3. Do I need to clean up when the values change or when the component unmounts?

Let's see how these questions apply in practice:

```javascript
const UserProfile = ({ userId }) => {
  const [user, setUser] = useState(null);
  const [error, setError] = useState(null);

  useEffect(() => {
    // Question 1: We're syncing with an API
    let isMounted = true;

    const fetchUser = async () => {
      try {
        const response = await fetch(`https://api.example.com/users/${userId}`);
        const data = await response.json();

        // Only update state if component is still mounted
        if (isMounted) {
          setUser(data);
          setError(null);
        }
      } catch (err) {
        if (isMounted) {
          setError(err.message);
        }
      }
    };

    fetchUser();

    // Question 3: Clean up if the component unmounts
    return () => {
      isMounted = false;
    };
  }, [userId]); // Question 2: Re-run when userId changes

  if (error) return <div>Error: {error}</div>;
  if (!user) return <div>Loading...</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
};
```

### Understanding the Dependency Array

The dependency array is like a list of "triggers" that tell your effect when to run. Let's explore different patterns:

```javascript
const DependencyExamples = ({ id, data }) => {
  // 1. Run on every render (usually a mistake!)
  useEffect(() => {
    console.log("This runs after every render");
  }); // No dependency array

  // 2. Run only once on mount
  useEffect(() => {
    console.log("This runs only on mount");
  }, []); // Empty dependency array

  // 3. Run when specific values change
  useEffect(() => {
    console.log("This runs when id changes");
  }, [id]); // Array with dependencies

  // 4. Run when object/array dependencies change
  useEffect(() => {
    console.log("This runs when data changes");
  }, [data]); // Be careful with objects and arrays!
};
```

### Handling Cleanup

Cleanup is crucial for preventing memory leaks and unexpected behavior. Here's how to handle different scenarios:

```javascript
const SubscriptionComponent = ({ channelId }) => {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    // Set up subscription
    const subscription = subscribeToChannel(channelId, (message) => {
      setMessages((prev) => [...prev, message]);
    });

    // Clean up subscription
    return () => {
      subscription.unsubscribe();
    };
  }, [channelId]);

  return (
    <div>
      {messages.map((msg) => (
        <div key={msg.id}>{msg.text}</div>
      ))}
    </div>
  );
};
```

### Common Patterns and Best Practices

#### 1. Data Fetching Pattern

Here's a reusable pattern for data fetching with proper cleanup and error handling:

```javascript
const useFetch = (url) => {
  const [state, setState] = useState({
    data: null,
    error: null,
    loading: true,
  });

  useEffect(() => {
    let isMounted = true;
    const abortController = new AbortController();

    const fetchData = async () => {
      try {
        const response = await fetch(url, {
          signal: abortController.signal,
        });
        const data = await response.json();

        if (isMounted) {
          setState({
            data,
            error: null,
            loading: false,
          });
        }
      } catch (error) {
        if (error.name === "AbortError") {
          return; // Ignore abort errors
        }

        if (isMounted) {
          setState({
            data: null,
            error: error.message,
            loading: false,
          });
        }
      }
    };

    setState((prev) => ({ ...prev, loading: true }));
    fetchData();

    return () => {
      isMounted = false;
      abortController.abort();
    };
  }, [url]);

  return state;
};
```

#### 2. Event Listener Pattern

Properly handling window events with cleanup:

```javascript
const WindowSize = () => {
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

    // Throttle the resize handler for better performance
    const throttledResize = throttle(handleResize, 250);

    window.addEventListener("resize", throttledResize);

    return () => {
      window.removeEventListener("resize", throttledResize);
    };
  }, []);

  return (
    <div>
      Window size: {size.width} x {size.height}
    </div>
  );
};
```

#### 3. Previous Value Pattern

When you need to compare current and previous values:

```javascript
const usePrevious = (value) => {
  const ref = useRef();

  useEffect(() => {
    ref.current = value;
  }, [value]);

  return ref.current;
};

const Counter = () => {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);

  return (
    <div>
      Now: {count}, before: {prevCount}
    </div>
  );
};
```

### Advanced useEffect Patterns

#### 1. Debounced Effect Pattern

When you want to delay an effect until values stop changing:

```javascript
const useDebounceEffect = (fn, deps, delay = 250) => {
  useEffect(() => {
    const handler = setTimeout(() => {
      fn();
    }, delay);

    return () => {
      clearTimeout(handler);
    };
  }, [...deps, delay]);
};

const SearchComponent = () => {
  const [query, setQuery] = useState("");
  const [results, setResults] = useState([]);

  useDebounceEffect(
    () => {
      const searchAPI = async () => {
        const data = await fetchSearchResults(query);
        setResults(data);
      };

      if (query) searchAPI();
    },
    [query],
    500
  );

  return (
    <div>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      {/* Results display */}
    </div>
  );
};
```

#### 2. Async Effect Pattern

Handling async operations safely:

```javascript
const useAsyncEffect = (fn, deps) => {
  useEffect(() => {
    let mounted = true;

    const runEffect = async () => {
      try {
        await fn(mounted);
      } catch (err) {
        console.error("Async effect error:", err);
      }
    };

    runEffect();

    return () => {
      mounted = false;
    };
  }, deps);
};

const AsyncComponent = ({ id }) => {
  const [data, setData] = useState(null);

  useAsyncEffect(
    async (mounted) => {
      const result = await fetchData(id);
      if (mounted) {
        setData(result);
      }
    },
    [id]
  );

  return <div>{/* Render data */}</div>;
};
```

### Common Mistakes and How to Fix Them

1. **Missing Dependencies**

```javascript
// ❌ Bad: Missing dependency
useEffect(() => {
  fetchData(userId);
}, []); // userId should be in deps

// ✅ Good: Include all dependencies
useEffect(() => {
  fetchData(userId);
}, [userId]);
```

2. **Infinite Loops**

```javascript
// ❌ Bad: Creates infinite loop
useEffect(() => {
  setData({}); // State update triggers re-render
}, [data]); // Which triggers effect

// ✅ Good: Only update when needed
useEffect(() => {
  if (!data) setData({});
}, [data]);
```

3. **Object Dependencies**

```javascript
// ❌ Bad: Object dependency causes unnecessary re-runs
useEffect(() => {
  doSomething(options);
}, [{ ...options }]); // New object every time

// ✅ Good: Depend on primitive values
useEffect(() => {
  doSomething(options);
}, [options.id, options.type]);
```

Want to explore any of these patterns in more detail or see more specific examples? Let me know!
