# component-lifecycle.md

Hey there! 👋

Let's explore how lifecycle works in modern React functional components. While class components had explicit lifecycle methods, functional components handle lifecycle events through hooks. This actually gives us more flexibility and makes our code cleaner and more reusable.

### Understanding the Component Lifecycle with Hooks

In functional components, we primarily use the `useEffect` hook to handle lifecycle events. Think of `useEffect` as a way to tell React "hey, something happened that needs attention!" Let's break down how we handle different lifecycle events:

### Component Mounting (Birth)

When a component first appears on screen, we handle this with `useEffect` with an empty dependency array:

```javascript
import React, { useEffect, useState } from "react";

const UserProfile = ({ userId }) => {
  const [userData, setUserData] = useState(null);
  const [isLoading, setIsLoading] = useState(true);

  // This is similar to componentDidMount
  useEffect(() => {
    console.log("Component is born! Time to fetch user data.");

    const fetchUserData = async () => {
      try {
        const response = await fetch(`/api/users/${userId}`);
        const data = await response.json();
        setUserData(data);
      } catch (error) {
        console.error("Failed to fetch user:", error);
      } finally {
        setIsLoading(false);
      }
    };

    fetchUserData();

    // This is similar to componentWillUnmount
    return () => {
      console.log("Component is leaving us. Cleanup time!");
      // Any cleanup code goes here
    };
  }, []); // Empty array means "run once when mounting"

  if (isLoading) return <div>Loading...</div>;
  if (!userData) return <div>No user data found</div>;

  return (
    <div>
      <h1>{userData.name}</h1>
      <p>{userData.email}</p>
    </div>
  );
};
```

### Component Updating (Growth and Change)

When we want to react to changes in props or state, we add dependencies to our `useEffect`:

```javascript
const DataViewer = ({ dataId, filter }) => {
  const [data, setData] = useState(null);
  const [processedData, setProcessedData] = useState(null);

  // Effect for fetching data when dataId changes
  useEffect(() => {
    console.log("DataId changed! Time to fetch new data.");

    const fetchData = async () => {
      const response = await fetch(`/api/data/${dataId}`);
      const newData = await response.json();
      setData(newData);
    };

    fetchData();
  }, [dataId]); // Only run when dataId changes

  // Separate effect for processing data when filter changes
  useEffect(() => {
    console.log("Filter changed! Processing data with new filter.");

    if (!data) return;

    const processData = () => {
      const filtered = data.filter((item) => item.category === filter);
      setProcessedData(filtered);
    };

    processData();
  }, [data, filter]); // Run when either data or filter changes

  return (
    <div>
      {processedData?.map((item) => (
        <div key={item.id}>{item.name}</div>
      ))}
    </div>
  );
};
```

### Component Cleanup (Saying Goodbye)

When a component needs to clean up after itself (like unsubscribing from events or closing connections), we use the cleanup function in `useEffect`:

```javascript
const LiveDataFeed = ({ topic }) => {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    console.log(`Subscribing to ${topic} feed`);

    // Set up WebSocket connection
    const ws = new WebSocket(`wss://api.example.com/${topic}`);

    ws.addEventListener("message", (event) => {
      setMessages((prev) => [...prev, event.data]);
    });

    // Cleanup function
    return () => {
      console.log(`Unsubscribing from ${topic} feed`);
      ws.close();
    };
  }, [topic]);

  return (
    <div>
      {messages.map((msg, index) => (
        <div key={index}>{msg}</div>
      ))}
    </div>
  );
};
```

### Practical Patterns for Lifecycle Management

Here's a more complex example showing how to manage multiple lifecycle aspects in a real-world component:

```javascript
const UserDashboard = ({ userId }) => {
  const [userData, setUserData] = useState(null);
  const [preferences, setPreferences] = useState(null);
  const [activity, setActivity] = useState([]);
  const [isLoading, setIsLoading] = useState(true);

  // Track mounted state to prevent updates after unmount
  const isMounted = useRef(true);

  // Setup and cleanup
  useEffect(() => {
    console.log("Dashboard is mounting...");

    // Cleanup function
    return () => {
      console.log("Dashboard is unmounting...");
      isMounted.current = false;
    };
  }, []);

  // Fetch initial user data
  useEffect(() => {
    const fetchUserData = async () => {
      try {
        const response = await fetch(`/api/users/${userId}`);
        const data = await response.json();

        // Prevent setting state if component unmounted
        if (isMounted.current) {
          setUserData(data);
          setIsLoading(false);
        }
      } catch (error) {
        console.error("Failed to fetch user:", error);
      }
    };

    fetchUserData();
  }, [userId]);

  // Set up real-time activity tracking
  useEffect(() => {
    if (!userData) return;

    console.log("Setting up activity tracking...");

    const activitySocket = new WebSocket(
      `wss://api.example.com/activity/${userId}`
    );

    activitySocket.addEventListener("message", (event) => {
      if (isMounted.current) {
        setActivity((prev) => [...prev, event.data]);
      }
    });

    // Cleanup subscriptions
    return () => {
      console.log("Cleaning up activity tracking...");
      activitySocket.close();
    };
  }, [userData, userId]);

  // Load user preferences
  useEffect(() => {
    if (!userData) return;

    const loadPreferences = async () => {
      const prefs = await fetch(`/api/users/${userId}/preferences`).then((r) =>
        r.json()
      );

      if (isMounted.current) {
        setPreferences(prefs);
      }
    };

    loadPreferences();
  }, [userData, userId]);

  if (isLoading) return <LoadingSpinner />;
  if (!userData) return <ErrorMessage />;

  return (
    <div>
      <UserInfo user={userData} />
      <PreferencesPanel preferences={preferences} />
      <ActivityFeed activities={activity} />
    </div>
  );
};
```

This organization keeps our code clean and maintainable while properly handling all lifecycle events. Want to explore any specific aspect in more detail? Let me know!
