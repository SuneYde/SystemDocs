# props-management.md

Hey there! 👋 Let's dive deep into one of React's most important concepts: Props. Think of props as the communication system of your React application - they're how components talk to each other and share information. I'm going to show you everything you need to know about managing props effectively.

### Understanding Props: The Basics

Props (short for properties) are like arguments you pass to a function. When you create a React component, you can pass it any number of props, and those props can contain any type of data - strings, numbers, objects, even other React components!

Let's start with a simple example:

```javascript
const Greeting = (props) => {
  return <h1>Hello, {props.name}!</h1>;
};

// Using the component
<Greeting name="Sarah" />;
```

This might look simple, but there's actually a lot happening here. The component receives a props object with all the properties we pass to it. In this case, props.name contains "Sarah".

### Advanced Props Techniques

Let's explore more sophisticated ways to work with props:

```javascript
const UserProfile = ({
  user,
  onUpdate,
  theme = "light", // Default prop value
  children, // Special prop for nested content
}) => {
  // Destructuring nested props
  const { firstName, lastName, email, preferences = {} } = user;

  return (
    <div className={`profile-card profile-card--${theme}`}>
      <div className="profile-header">
        <h2>
          {firstName} {lastName}
        </h2>
        <span className="email">{email}</span>
      </div>

      {/* Conditional rendering based on props */}
      {preferences.showBio && <p className="bio">{user.bio}</p>}

      {/* Render children props */}
      <div className="profile-content">{children}</div>

      {/* Event handler props */}
      <button onClick={() => onUpdate(user.id)}>Update Profile</button>
    </div>
  );
};

// PropTypes for type checking
UserProfile.propTypes = {
  user: PropTypes.shape({
    id: PropTypes.string.isRequired,
    firstName: PropTypes.string.isRequired,
    lastName: PropTypes.string.isRequired,
    email: PropTypes.string.isRequired,
    bio: PropTypes.string,
    preferences: PropTypes.shape({
      showBio: PropTypes.bool,
    }),
  }).isRequired,
  onUpdate: PropTypes.func.isRequired,
  theme: PropTypes.oneOf(["light", "dark"]),
  children: PropTypes.node,
};
```

### Common Props Patterns

Let's look at some patterns you'll use frequently:

#### 1. Props Spreading

Sometimes you want to pass all props through to a child component. Here's how:

```javascript
const Button = (props) => {
  const { children, className, ...restProps } = props;

  return (
    <button
      className={`button ${className || ""}`}
      {...restProps} // Spreads remaining props
    >
      {children}
    </button>
  );
};
```

#### 2. Render Props Pattern

This powerful pattern lets you share code between components:

```javascript
const MouseTracker = ({ render }) => {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  const handleMouseMove = (event) => {
    setPosition({
      x: event.clientX,
      y: event.clientY,
    });
  };

  return <div onMouseMove={handleMouseMove}>{render(position)}</div>;
};

// Usage:
<MouseTracker
  render={({ x, y }) => (
    <div>
      Mouse position: {x}, {y}
    </div>
  )}
/>;
```

### Props Best Practices

1. **Keep Components Pure**
   Props should be treated as immutable. Never modify props directly:

```javascript
// ❌ Bad - modifying props
const BadComponent = (props) => {
  props.value = 123; // Never do this!
  return <div>{props.value}</div>;
};

// ✅ Good - use state if you need to modify values
const GoodComponent = (props) => {
  const [value, setValue] = useState(props.initialValue);
  return <div>{value}</div>;
};
```

2. **Use Default Props Wisely**
   Provide sensible defaults to make your components more robust:

```javascript
const Card = ({
  title,
  subtitle = "No subtitle provided",
  className = "",
  children,
  isLoading = false,
}) => {
  if (isLoading) {
    return <LoadingSpinner />;
  }

  return (
    <div className={`card ${className}`}>
      <h2>{title}</h2>
      <h3>{subtitle}</h3>
      {children}
    </div>
  );
};
```

3. **Prop Validation**
   Always validate your props to catch errors early:

```javascript
const DataGrid = (props) => {
  // Component implementation
};

DataGrid.propTypes = {
  // Basic types
  title: PropTypes.string.isRequired,
  rowCount: PropTypes.number,

  // Arrays and objects
  data: PropTypes.arrayOf(
    PropTypes.shape({
      id: PropTypes.string.isRequired,
      name: PropTypes.string.isRequired,
      age: PropTypes.number,
    })
  ).isRequired,

  // Functions
  onRowClick: PropTypes.func,

  // Custom validator
  customProp: (props, propName, componentName) => {
    if (!/^[A-Z]/.test(props[propName])) {
      return new Error(
        `${propName} in ${componentName} must start with a capital letter`
      );
    }
  },
};
```

4. **Props Documentation**
   Document your components' props thoroughly:

```javascript
/**
 * Displays a user's profile information with customizable styling
 * and interaction handlers.
 *
 * @param {Object} props
 * @param {Object} props.user - User data object
 * @param {string} props.user.firstName - User's first name
 * @param {string} props.user.lastName - User's last name
 * @param {string} [props.theme='light'] - Visual theme of the profile
 * @param {Function} props.onUpdate - Callback when profile is updated
 * @param {React.ReactNode} props.children - Content to render inside profile
 */
const UserProfile = ({ user, theme = "light", onUpdate, children }) => {
  // Component implementation
};
```

### Handling Complex Prop Scenarios

Here's how to handle more complex prop situations:

```javascript
const ComplexComponent = ({
  // Nested destructuring
  user: {
    profile: { firstName, lastName },
    settings: { theme, notifications },
  },
  // Multiple handlers
  onUpdate,
  onDelete,
  onStatusChange,
  // Optional props with defaults
  layout = "grid",
  sortOrder = "asc",
  // Props with conditions
  isEditable = false,
  isDeleted = false,
}) => {
  // Derived values from props
  const fullName = `${firstName} ${lastName}`;
  const canEdit = isEditable && !isDeleted;

  // Combine multiple handler props
  const handleAction = (action) => {
    switch (action) {
      case "update":
        return onUpdate?.();
      case "delete":
        return onDelete?.();
      case "status":
        return onStatusChange?.();
      default:
        console.warn("Unknown action:", action);
    }
  };

  return (
    <div className={`complex-component layout-${layout}`}>
      <h2>{fullName}</h2>

      {canEdit && (
        <div className="actions">
          <button onClick={() => handleAction("update")}>Update</button>
          <button onClick={() => handleAction("delete")}>Delete</button>
        </div>
      )}
    </div>
  );
};
```

Want to explore any of these concepts further or see more examples of specific prop patterns? Let me know!
