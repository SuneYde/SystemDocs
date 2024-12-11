# functional-components.md

Hello React explorer! Welcome to your deep dive into functional components. I'm excited to guide you through one of the most important concepts in modern React development. Let's start from the ground up and build a thorough understanding of how functional components work.

### Understanding Functional Components

A functional component is, at its heart, just a JavaScript function that returns React elements. Think of it like a blueprint for a piece of your user interface. Just as an architect might design a room that can be reused throughout a building, we create components that can be reused throughout our application.

### The Anatomy of a Functional Component

Let's look at the basic structure:

```javascript
import React from "react";

const Welcome = () => {
  return (
    <div>
      <h1>Welcome to our app!</h1>
    </div>
  );
};

export default Welcome;
```

This might look simple, but there's a lot happening here. Let's break it down:

1. The function declaration creates a new component
2. The function returns JSX (React's syntax for describing UI)
3. We export the component so other parts of our app can use it

### Building Your First Component

Let's create a more practical component that you might use in a real application:

```javascript
import React, { useState } from "react";

const UserCard = ({ username, email, role = "user" }) => {
  // State for managing expanded/collapsed view
  const [isExpanded, setIsExpanded] = useState(false);

  // Helper function for formatting email
  const formatEmail = (email) => {
    return email.toLowerCase();
  };

  return (
    <div className="user-card">
      <div className="user-card__header">
        <h2>{username}</h2>
        <button
          onClick={() => setIsExpanded(!isExpanded)}
          className="user-card__toggle"
        >
          {isExpanded ? "Show Less" : "Show More"}
        </button>
      </div>

      {isExpanded && (
        <div className="user-card__details">
          <p>Email: {formatEmail(email)}</p>
          <p>Role: {role}</p>
        </div>
      )}
    </div>
  );
};

export default UserCard;
```

### Component Organization Patterns

As your components grow, organizing them well becomes crucial. Here's a proven pattern:

```javascript
import React, { useState, useEffect } from "react";
import PropTypes from "prop-types";

// 1. Constants and configurations
const CARD_STATES = {
  COLLAPSED: "collapsed",
  EXPANDED: "expanded",
};

// 2. Helper functions (outside component)
const formatDateTime = (date) => {
  return new Intl.DateTimeFormat("en-US").format(date);
};

// 3. The component itself
const BlogPost = ({ title, content, author, date }) => {
  // 3a. State declarations
  const [displayState, setDisplayState] = useState(CARD_STATES.COLLAPSED);
  const [readTime, setReadTime] = useState(0);

  // 3b. Effects and calculations
  useEffect(() => {
    // Calculate reading time based on content length
    const wordsPerMinute = 200;
    const words = content.split(" ").length;
    setReadTime(Math.ceil(words / wordsPerMinute));
  }, [content]);

  // 3c. Event handlers
  const handleExpandClick = () => {
    setDisplayState(
      displayState === CARD_STATES.COLLAPSED
        ? CARD_STATES.EXPANDED
        : CARD_STATES.COLLAPSED
    );
  };

  // 3d. Render helpers
  const renderHeader = () => (
    <header className="blog-post__header">
      <h2>{title}</h2>
      <div className="blog-post__meta">
        <span>By {author}</span>
        <span>•</span>
        <span>{formatDateTime(date)}</span>
        <span>•</span>
        <span>{readTime} min read</span>
      </div>
    </header>
  );

  // 3e. Main render
  return (
    <article className="blog-post">
      {renderHeader()}
      <div className={`blog-post__content ${displayState}`}>
        <p>{content}</p>
      </div>
      <button onClick={handleExpandClick}>
        {displayState === CARD_STATES.COLLAPSED ? "Read More" : "Show Less"}
      </button>
    </article>
  );
};

// 4. PropTypes
BlogPost.propTypes = {
  title: PropTypes.string.isRequired,
  content: PropTypes.string.isRequired,
  author: PropTypes.string.isRequired,
  date: PropTypes.instanceOf(Date).isRequired,
};

// 5. Default props (if needed)
BlogPost.defaultProps = {
  author: "Anonymous",
};

export default BlogPost;
```

### Composition and Reusability

One of the most powerful aspects of functional components is how easily they can be composed together. Here's an example of component composition:

```javascript
// A simple Button component
const Button = ({ onClick, children, variant = "primary" }) => (
  <button className={`button button--${variant}`} onClick={onClick}>
    {children}
  </button>
);

// A Card component that can be reused
const Card = ({ title, children }) => (
  <div className="card">
    <div className="card__header">
      <h3>{title}</h3>
    </div>
    <div className="card__content">{children}</div>
  </div>
);

// Composing them together in a feature component
const FeatureSection = ({ feature }) => {
  const [isEnabled, setIsEnabled] = useState(false);

  return (
    <Card title={feature.name}>
      <p>{feature.description}</p>
      <Button
        onClick={() => setIsEnabled(!isEnabled)}
        variant={isEnabled ? "success" : "default"}
      >
        {isEnabled ? "Enabled" : "Disabled"}
      </Button>
    </Card>
  );
};
```

### Performance Optimization

Even with functional components, performance matters. Here's how to optimize:

```javascript
import React, { memo, useCallback, useMemo } from "react";

const ExpensiveComponent = memo(({ data, onItemSelect }) => {
  // Memoize expensive calculations
  const processedData = useMemo(() => {
    return data.map((item) => ({
      ...item,
      fullName: `${item.firstName} ${item.lastName}`,
      totalScore: item.scores.reduce((a, b) => a + b, 0),
    }));
  }, [data]);

  // Memoize callbacks
  const handleSelect = useCallback(
    (item) => {
      onItemSelect(item.id);
    },
    [onItemSelect]
  );

  return (
    <div className="data-grid">
      {processedData.map((item) => (
        <div
          key={item.id}
          onClick={() => handleSelect(item)}
          className="data-grid__item"
        >
          <h3>{item.fullName}</h3>
          <p>Total Score: {item.totalScore}</p>
        </div>
      ))}
    </div>
  );
});
```

### Common Patterns and Best Practices

1. **Early Returns for Conditional Rendering**

```javascript
const UserProfile = ({ user, isLoading, error }) => {
  if (error) {
    return <ErrorMessage message={error} />;
  }

  if (isLoading) {
    return <LoadingSpinner />;
  }

  if (!user) {
    return <NotFound message="User not found" />;
  }

  return <div className="user-profile">{/* Main content */}</div>;
};
```

2. **Controlled Components for Forms**

```javascript
const LoginForm = ({ onSubmit }) => {
  const [formData, setFormData] = useState({
    email: "",
    password: "",
  });

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData((prev) => ({
      ...prev,
      [name]: value,
    }));
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    onSubmit(formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        name="email"
        value={formData.email}
        onChange={handleChange}
      />
      <input
        type="password"
        name="password"
        value={formData.password}
        onChange={handleChange}
      />
      <button type="submit">Login</button>
    </form>
  );
};
```

Would you like me to continue with the class components guide next? Or would you like to explore any specific aspect of functional components in more detail?
