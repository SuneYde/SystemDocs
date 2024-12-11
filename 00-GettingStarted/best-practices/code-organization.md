# code-organization.md

Hey future coding architect! 👋

Let's talk about organizing your code like a pro. Think of your code like building a house - we want a strong foundation, clear blueprints, and everything in its right place. I'll show you how to write code that both you and others will love working with.

### The Art of Clean Code

When we write code, we're not just writing it for computers to execute - we're writing it for other developers (including future you!) to read and understand. Here's how to make your code shine:

### File Structure - Your Code's Home

Your project structure should tell a story. Imagine walking into a well-organized library where you can find any book instantly. That's what we're aiming for:

```
src/
├── components/
│   ├── common/           # Reusable components
│   │   ├── Button/
│   │   │   ├── Button.jsx
│   │   │   ├── Button.test.js
│   │   │   └── Button.css
│   │   └── Card/
│   ├── features/         # Feature-specific components
│   │   ├── Auth/
│   │   └── Dashboard/
├── pages/                # Page components
├── hooks/                # Custom React hooks
├── utils/                # Helper functions
├── services/             # API calls and external services
└── styles/              # Global styles
```

Let's understand why this structure works:

1. Each component has its own folder with related files
2. Common components are separated from feature-specific ones
3. Business logic (services) is separated from presentation (components)
4. Utils folder keeps helper functions organized and reusable

### Component Organization

Let's look at how to structure a React component properly:

```javascript
// GoodComponent.jsx

// 1. Imports - Grouped and ordered
import React, { useState, useEffect } from "react";
import PropTypes from "prop-types";
// External libraries
import { format } from "date-fns";
// Internal components
import Button from "../common/Button";
// Styles and utils
import { formatUserName } from "../../utils";
import "./GoodComponent.css";

// 2. Component definition
const GoodComponent = ({ user, onUpdate }) => {
  // State declarations at the top
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);

  // Effects after state
  useEffect(() => {
    // Clear and focused effect logic
    const loadUserData = async () => {
      try {
        setIsLoading(true);
        // Load data...
      } catch (err) {
        setError(err.message);
      } finally {
        setIsLoading(false);
      }
    };

    loadUserData();
  }, [user.id]); // Clear dependencies

  // Event handlers and helper functions
  const handleSubmit = async (event) => {
    event.preventDefault();
    // Handle form submission...
  };

  // Conditional rendering logic
  if (isLoading) return <LoadingSpinner />;
  if (error) return <ErrorMessage message={error} />;

  // Main render
  return (
    <div className="good-component">
      <h2>{formatUserName(user.name)}</h2>
      <form onSubmit={handleSubmit}>{/* Form content */}</form>
    </div>
  );
};

// 3. PropTypes at the bottom
GoodComponent.propTypes = {
  user: PropTypes.shape({
    id: PropTypes.string.required,
    name: PropTypes.string.required,
  }).isRequired,
  onUpdate: PropTypes.func.isRequired,
};

export default GoodComponent;
```

### Naming Conventions

Good names are like good signposts - they tell you exactly where you're going. Here's how to name things effectively:

```javascript
// Components - PascalCase
const UserProfile = () => {};

// Functions - camelCase and verb-based
const getUserData = () => {};
const handleSubmit = () => {};
const formatUserName = () => {};

// Variables - camelCase and descriptive
const isLoading = true;
const userList = [];
const currentUser = {};

// Constants - UPPERCASE_SNAKE_CASE
const MAX_RETRY_ATTEMPTS = 3;
const API_BASE_URL = "https://api.example.com";
```

### State Management

Managing state is like organizing your tools - everything should have its place and purpose:

```javascript
// Good state management
const UserDashboard = () => {
  // Group related state together
  const [user, setUser] = useState({
    name: "",
    email: "",
    preferences: {},
  });

  // Separate loading and error states
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);

  // Derived state - calculate from existing state
  const isComplete = user.name && user.email;

  // State updates in clear, focused functions
  const updateUserName = (name) => {
    setUser((prev) => ({
      ...prev,
      name,
    }));
  };
};
```

### Error Handling

Errors are like fire alarms - they should be clear, helpful, and appropriate:

```javascript
// Proper error handling
const fetchUserData = async (userId) => {
  try {
    // Always include error context
    if (!userId) {
      throw new Error("User ID is required");
    }

    const response = await api.get(`/users/${userId}`);

    // Validate response data
    if (!response.data) {
      throw new Error("No data received from server");
    }

    return response.data;
  } catch (error) {
    // Add context to errors
    throw new Error(`Failed to fetch user data: ${error.message}`);
  }
};
```

### Performance Optimization

Think of performance optimization like tuning a car - focus on what matters most:

```javascript
// Memoize expensive calculations
const MemoizedComponent = memo(({ data }) => {
  // Memoize complex calculations
  const processedData = useMemo(() => {
    return data.map(item => expensiveOperation(item));
  }, [data]);

  // Memoize callbacks
  const handleClick = useCallback(() => {
    // Handle click...
  }, []); // Empty deps if no dependencies

  return (
    // Component content...
  );
});

// Use fragments to avoid unnecessary divs
const CleanStructure = () => (
  <>
    <Header />
    <Main />
    <Footer />
  </>
);
```

### Code Comments and Documentation

Comments should explain why, not what. The code shows what; comments explain why decisions were made:

```javascript
// Bad comment - explains what
// Set user to active
user.setActive(true);

// Good comment - explains why
// Enable user account after email verification
user.setActive(true);

// Good documentation
/**
 * Processes user data for dashboard display
 * @param {Object} userData - Raw user data from API
 * @returns {Object} Processed data with computed fields
 * @throws {Error} If userData is invalid
 */
const processUserData = (userData) => {
  // Processing logic...
};
```

### Development Process Best Practices

1. **Version Control**

   - Make small, focused commits
   - Write meaningful commit messages
   - Use feature branches

2. **Testing**
   Write tests that:

   - Test behavior, not implementation
   - Cover edge cases
   - Stay maintainable

3. **Code Review**
   When reviewing code:
   - Look for clarity and maintainability
   - Check for potential bugs
   - Verify error handling
   - Ensure consistent styling

### Daily Coding Checklist

Before committing your code, ask yourself:

- Is the code clean and formatted?
- Are variable and function names clear?
- Is there proper error handling?
- Are there appropriate comments?
- Did I remove console.logs and debugging code?
- Do the tests pass?

Remember: Writing good code is like writing a good story - it should be clear, coherent, and easy to follow. Want to practice any of these concepts or dive deeper into a particular area?
