# useRef.md

Hey there! Let's dive into React's useRef hook. Think of useRef as a special kind of container that can hold onto values between renders, but unlike state, it doesn't cause re-renders when changed. It's like having a secret notebook where you can write things down without React noticing.

### Understanding useRef: The Basics

There are two main use cases for useRef:

1. Accessing DOM elements directly
2. Storing mutable values that don't need re-renders

Let's look at both:

```javascript
import React, { useRef, useEffect } from "react";

const TextInput = () => {
  // Create a ref to hold the input element
  const inputRef = useRef(null);

  // Example: Focus the input when component mounts
  useEffect(() => {
    inputRef.current.focus();
  }, []);

  return (
    <input ref={inputRef} type="text" placeholder="I'll be focused on mount!" />
  );
};
```

### Working with DOM Elements

Here's a more comprehensive example of DOM manipulation with useRef:

```javascript
const VideoPlayer = ({ src }) => {
  const videoRef = useRef(null);
  const [isPlaying, setIsPlaying] = useState(false);

  // Control video playback
  const handlePlayPause = () => {
    if (videoRef.current) {
      if (isPlaying) {
        videoRef.current.pause();
      } else {
        videoRef.current.play();
      }
      setIsPlaying(!isPlaying);
    }
  };

  // Jump to specific time
  const jumpToTime = (timeInSeconds) => {
    if (videoRef.current) {
      videoRef.current.currentTime = timeInSeconds;
    }
  };

  // Update volume
  const setVolume = (level) => {
    if (videoRef.current) {
      videoRef.current.volume = level;
    }
  };

  return (
    <div>
      <video ref={videoRef} src={src} style={{ width: "100%" }} />
      <div className="controls">
        <button onClick={handlePlayPause}>
          {isPlaying ? "Pause" : "Play"}
        </button>
        <button onClick={() => jumpToTime(30)}>Jump to 0:30</button>
        <input
          type="range"
          min="0"
          max="1"
          step="0.1"
          onChange={(e) => setVolume(e.target.value)}
        />
      </div>
    </div>
  );
};
```

### Storing Mutable Values

UseRef is perfect for values that need to persist between renders but don't require UI updates:

```javascript
const StopWatch = () => {
  const [time, setTime] = useState(0);
  const intervalRef = useRef(null);
  const startTimeRef = useRef(null);

  const start = () => {
    startTimeRef.current = Date.now() - time;
    intervalRef.current = setInterval(() => {
      setTime(Date.now() - startTimeRef.current);
    }, 10);
  };

  const stop = () => {
    clearInterval(intervalRef.current);
  };

  const reset = () => {
    clearInterval(intervalRef.current);
    setTime(0);
  };

  // Clean up on unmount
  useEffect(() => {
    return () => clearInterval(intervalRef.current);
  }, []);

  return (
    <div>
      <div>{(time / 1000).toFixed(2)}s</div>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
};
```

### Advanced Patterns with useRef

#### 1. Previous Value Pattern

Store and compare with previous values:

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
      <p>
        Current: {count}, Previous: {prevCount}
      </p>
      <button onClick={() => setCount((c) => c + 1)}>Increment</button>
    </div>
  );
};
```

#### 2. Intersection Observer Pattern

Track element visibility:

```javascript
const useIntersectionObserver = (options = {}) => {
  const elementRef = useRef(null);
  const [isVisible, setIsVisible] = useState(false);

  useEffect(() => {
    const element = elementRef.current;

    if (!element) return;

    const observer = new IntersectionObserver(([entry]) => {
      setIsVisible(entry.isIntersecting);
    }, options);

    observer.observe(element);

    return () => observer.unobserve(element);
  }, [options]);

  return [elementRef, isVisible];
};

const LazyImage = ({ src, alt }) => {
  const [ref, isVisible] = useIntersectionObserver({
    threshold: 0.1,
  });

  return <div ref={ref}>{isVisible && <img src={src} alt={alt} />}</div>;
};
```

#### 3. Event Listener Management

Handle event listeners cleanly:

```javascript
const useEventListener = (eventName, handler, element = window) => {
  const savedHandler = useRef();

  useEffect(() => {
    savedHandler.current = handler;
  }, [handler]);

  useEffect(() => {
    const isSupported = element && element.addEventListener;
    if (!isSupported) return;

    const eventListener = (event) => savedHandler.current(event);
    element.addEventListener(eventName, eventListener);

    return () => {
      element.removeEventListener(eventName, eventListener);
    };
  }, [eventName, element]);
};

const MouseTracker = () => {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEventListener("mousemove", (event) => {
    setPosition({ x: event.clientX, y: event.clientY });
  });

  return (
    <div>
      Mouse position: {position.x}, {position.y}
    </div>
  );
};
```

#### 4. Form Input Management

Handle complex form interactions:

```javascript
const ComplexForm = () => {
  const formRef = useRef(null);
  const inputRefs = useRef({});

  const registerInput = (name, ref) => {
    inputRefs.current[name] = ref;
  };

  const focusInput = (name) => {
    if (inputRefs.current[name]) {
      inputRefs.current[name].focus();
    }
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    const formData = new FormData(formRef.current);
    // Process form data...
  };

  return (
    <form ref={formRef} onSubmit={handleSubmit}>
      <input
        ref={(ref) => registerInput("email", ref)}
        type="email"
        name="email"
      />
      <input
        ref={(ref) => registerInput("password", ref)}
        type="password"
        name="password"
      />
      <button type="button" onClick={() => focusInput("email")}>
        Focus Email
      </button>
      <button type="submit">Submit</button>
    </form>
  );
};
```

### Best Practices and Tips

1. **Don't Overuse Refs**

```javascript
// ❌ Don't use ref for this
const Counter = () => {
  const countRef = useRef(0);
  return (
    <button
      onClick={() => {
        countRef.current += 1; // Won't trigger re-render!
      }}
    >
      Count: {countRef.current}
    </button>
  );
};

// ✅ Use state instead
const Counter = () => {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount((c) => c + 1)}>Count: {count}</button>;
};
```

2. **Cleanup Properly**

```javascript
const Timer = () => {
  const timerRef = useRef();

  useEffect(() => {
    // Set up timer
    timerRef.current = setInterval(() => {
      console.log("tick");
    }, 1000);

    // Clean up
    return () => clearInterval(timerRef.current);
  }, []);

  return <div>Timer running...</div>;
};
```

3. **Type Safety with TypeScript**

```typescript
const TextInput = () => {
  const inputRef = useRef<HTMLInputElement>(null);

  const focus = () => {
    // TypeScript knows this might be null
    if (inputRef.current) {
      inputRef.current.focus();
    }
  };

  return <input ref={inputRef} />;
};
```

Want to explore any of these concepts further or see more specific examples? Let me know!
