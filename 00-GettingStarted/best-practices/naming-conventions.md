# naming-conventions.md

Hey there, coding friend! 👋

Let's talk about one of the most important skills in programming - naming things! You know how in Harry Potter, naming something gives you power over it? It's the same in coding. The names we choose for our variables, functions, and components have real power over how understandable and maintainable our code becomes.

### Why Naming Matters

Imagine you're reading a story where all the characters are named "Person1", "Person2", "Person3". Pretty confusing, right? Good names in code work like good character names in a story - they tell you about their purpose and role. Let's learn how to name everything in our code clearly and consistently.

### The Golden Rules of Naming

Before we dive into specific conventions, let's understand the core principles that make names good:

1. **Clarity Over Brevity**

   ```javascript
   // Bad - too short to understand
   const u = getUsr();
   const d = new Date();

   // Good - clear and descriptive
   const currentUser = getUser();
   const startDate = new Date();
   ```

2. **Meaningful Names**

   ```javascript
   // Bad - generic names
   const data = [];
   const doThing = () => {};

   // Good - specific and meaningful
   const userScores = [];
   const calculateAverageScore = () => {};
   ```

### Variable Naming

Variables are like containers for our data. Their names should tell us what's inside:

```javascript
// Case: camelCase for variables
const firstName = "John";
const isActive = true;
const userScores = [95, 88, 92];

// For arrays, use plural nouns
const users = ["John", "Jane", "Bob"];
const todoItems = getTodoList();

// For booleans, use 'is', 'has', or 'should'
const isLoggedIn = true;
const hasPermission = false;
const shouldRedirect = true;

// For numbers, be specific about what they represent
const maxLoginAttempts = 3;
const userAge = 25;
const priceInCents = 2995;

// For objects, use nouns that describe the data
const userProfile = {
  name: "John Doe",
  age: 30,
  email: "john@example.com",
};
```

### Function Naming

Functions are actions, so their names should contain verbs:

```javascript
// Use verb + noun format
const getUserProfile = () => {};
const validateEmail = () => {};
const calculateTotal = () => {};

// For event handlers, use 'handle' prefix
const handleSubmit = (event) => {};
const handleUserClick = () => {};
const handleInputChange = (event) => {};

// For boolean returns, use question words
const isValidEmail = (email) => {};
const hasAccess = (user) => {};
const shouldShowModal = () => {};

// For transformations, be clear about what's happening
const convertToUpperCase = (text) => {};
const formatAsCurrency = (number) => {};
const mapToViewModel = (data) => {};
```

### Component Naming

React components are like LEGO pieces - their names should tell us what they build:

```javascript
// Use PascalCase for components
const UserProfile = () => {};
const NavigationBar = () => {};
const LoginForm = () => {};

// For container components, add Container suffix
const UserProfileContainer = () => {};
const DashboardContainer = () => {};

// For higher-order components, use 'with' prefix
const withAuth = (WrappedComponent) => {};
const withTheme = (WrappedComponent) => {};

// For context providers, use 'Provider' suffix
const ThemeProvider = ({ children }) => {};
const AuthProvider = ({ children }) => {};
```

### File Naming

Files are like chapters in a book - their names should reflect their content:

```plaintext
// Component files - use PascalCase
UserProfile.jsx
NavigationBar.jsx
LoginForm.jsx

// Utility files - use camelCase
formatDate.js
validateInput.js
apiHelpers.js

// Style files - match component name
UserProfile.css
NavigationBar.module.css

// Test files - add .test or .spec
UserProfile.test.jsx
formatDate.spec.js
```

### Constants and Configuration

Constants are like the rules of your application - make them stand out:

```javascript
// Use UPPERCASE_SNAKE_CASE for constants
const MAX_LOGIN_ATTEMPTS = 3;
const API_BASE_URL = "https://api.example.com";
const DEFAULT_USER_ROLE = "user";

// Group related constants
const HttpStatus = {
  OK: 200,
  CREATED: 201,
  BAD_REQUEST: 400,
  UNAUTHORIZED: 401,
  NOT_FOUND: 404,
};

const UserRoles = {
  ADMIN: "admin",
  EDITOR: "editor",
  VIEWER: "viewer",
};
```

### Special Naming Patterns

Some scenarios require special naming consideration:

```javascript
// Private methods/properties (by convention)
class UserService {
  _validatePassword(password) {}
  _encryptData(data) {}
}

// Temporary variables (when needed)
const tempUser = { ...user };
let tempCount = count;

// Interface names (TypeScript)
interface IUserService {}
interface IRepository {}

// Type names (TypeScript)
type UserResponse = {
  id: string,
  name: string,
};
```

### Real-World Examples

Let's look at a complete example putting it all together:

```javascript
// UserProfileManager.jsx
import React, { useState, useEffect } from "react";

// Constants
const UPDATE_INTERVAL_MS = 5000;
const MAX_NAME_LENGTH = 50;

// Types
interface IUserData {
  id: string;
  firstName: string;
  lastName: string;
  emailAddress: string;
}

// Component
const UserProfileManager = ({ userId }) => {
  // State variables
  const [userData, setUserData] = (useState < IUserData) | (null > null);
  const [isLoading, setIsLoading] = useState(false);
  const [hasError, setHasError] = useState(false);

  // Event handlers
  const handleProfileUpdate = async (newData) => {
    try {
      await updateUserProfile(newData);
    } catch (error) {
      setHasError(true);
    }
  };

  // Helper functions
  const formatUserName = (firstName, lastName) => {
    return `${firstName} ${lastName}`.trim();
  };

  return <div className="user-profile-manager">{/* Component content */}</div>;
};

export default UserProfileManager;
```

### Common Pitfalls to Avoid

1. **Avoid Single Letter Names** (except in loops)

   ```javascript
   // Bad
   const x = 5;
   const fn = () => {};

   // Good (in loops)
   for (let i = 0; i < items.length; i++) {}
   ```

2. **Don't Use Ambiguous Abbreviations**

   ```javascript
   // Bad
   const usr = getUser();
   const btn = document.querySelector("button");

   // Good
   const user = getUser();
   const button = document.querySelector("button");
   ```

3. **Avoid Context Duplication**

   ```javascript
   // Bad
   const userFirstName = "John";
   const userLastName = "Doe";

   // Good
   const user = {
     firstName: "John",
     lastName: "Doe",
   };
   ```

Remember: Good names are like good documentation - they make your code self-explaining. When in doubt, ask yourself: "Will another developer (or future me) understand what this name means without additional context?"

Want to practice naming in a specific scenario? Have a piece of code you'd like to improve? Let me know!
