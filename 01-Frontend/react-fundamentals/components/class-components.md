# class-components.md

Welcome back! Now that we understand functional components, let's explore class components. While modern React tends to favor functional components, understanding class components is still important - especially when maintaining existing codebases or working with older React applications.

### Understanding Class Components

Class components are JavaScript classes that extend React.Component. They were the primary way of creating components in React before hooks were introduced. Think of them as blueprints that include both the structure (render method) and behavior (lifecycle methods) of a component.

### Basic Structure of a Class Component

Let's start with a simple example and then build up to more complex implementations:

```javascript
import React, { Component } from "react";

class Welcome extends Component {
  // The constructor is called when the component is initialized
  constructor(props) {
    super(props);
    // State must be initialized in the constructor
    this.state = {
      greeting: "Welcome to our app!",
    };
  }

  // The render method is required - it returns the JSX
  render() {
    return (
      <div>
        <h1>{this.state.greeting}</h1>
      </div>
    );
  }
}

export default Welcome;
```

### A More Complete Class Component Example

Let's create a more practical component that showcases the key features of class components:

```javascript
import React, { Component } from "react";
import PropTypes from "prop-types";

class UserDashboard extends Component {
  constructor(props) {
    super(props);

    // Initialize state
    this.state = {
      userData: null,
      isLoading: true,
      error: null,
      activeTab: "profile",
    };

    // Binding methods to this
    this.handleTabChange = this.handleTabChange.bind(this);
    this.fetchUserData = this.fetchUserData.bind(this);
  }

  // Lifecycle method: Called after component mounts
  componentDidMount() {
    this.fetchUserData();
  }

  // Lifecycle method: Called when component updates
  componentDidUpdate(prevProps) {
    // Check if userId prop has changed
    if (prevProps.userId !== this.props.userId) {
      this.fetchUserData();
    }
  }

  // Lifecycle method: Called before component unmounts
  componentWillUnmount() {
    // Clean up any subscriptions or timers
    if (this.dataSubscription) {
      this.dataSubscription.unsubscribe();
    }
  }

  // Class method to fetch user data
  async fetchUserData() {
    try {
      this.setState({ isLoading: true });
      const response = await fetch(`/api/users/${this.props.userId}`);
      const userData = await response.json();

      this.setState({
        userData,
        isLoading: false,
        error: null,
      });
    } catch (error) {
      this.setState({
        error: "Failed to fetch user data",
        isLoading: false,
      });
    }
  }

  // Event handler for tab changes
  handleTabChange(tab) {
    this.setState({ activeTab: tab });
  }

  // Helper method to render content based on active tab
  renderTabContent() {
    const { userData, activeTab } = this.state;

    switch (activeTab) {
      case "profile":
        return (
          <div className="profile-tab">
            <h2>{userData.name}'s Profile</h2>
            <p>Email: {userData.email}</p>
          </div>
        );
      case "settings":
        return (
          <div className="settings-tab">
            <h2>User Settings</h2>
            {/* Settings content */}
          </div>
        );
      default:
        return null;
    }
  }

  // The main render method
  render() {
    const { isLoading, error, activeTab } = this.state;

    if (isLoading) {
      return <div>Loading...</div>;
    }

    if (error) {
      return <div>Error: {error}</div>;
    }

    return (
      <div className="user-dashboard">
        <nav className="dashboard-tabs">
          <button
            className={activeTab === "profile" ? "active" : ""}
            onClick={() => this.handleTabChange("profile")}
          >
            Profile
          </button>
          <button
            className={activeTab === "settings" ? "active" : ""}
            onClick={() => this.handleTabChange("settings")}
          >
            Settings
          </button>
        </nav>

        <main className="dashboard-content">{this.renderTabContent()}</main>
      </div>
    );
  }
}

// PropTypes for type checking
UserDashboard.propTypes = {
  userId: PropTypes.string.isRequired,
};

export default UserDashboard;
```

### Working with State in Class Components

State management in class components is different from functional components. Here's a detailed example:

```javascript
class Counter extends Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0,
      lastUpdated: null,
    };
  }

  // Updating state correctly with setState
  increment = () => {
    // Wrong way:
    // this.state.count += 1;

    // Correct way - using previous state:
    this.setState((prevState) => ({
      count: prevState.count + 1,
      lastUpdated: new Date(),
    }));
  };

  // setState with callback
  incrementWithCallback = () => {
    this.setState(
      (prevState) => ({
        count: prevState.count + 1,
      }),
      () => {
        console.log("Counter updated:", this.state.count);
      }
    );
  };

  render() {
    return (
      <div>
        <h2>Count: {this.state.count}</h2>
        <button onClick={this.increment}>Increment</button>
      </div>
    );
  }
}
```

### Class Component Patterns and Best Practices

1. **Higher-Order Components (HOC) Pattern**

```javascript
// HOC to add authentication to a component
const withAuth = (WrappedComponent) => {
  return class WithAuth extends Component {
    constructor(props) {
      super(props);
      this.state = {
        isAuthenticated: false,
      };
    }

    componentDidMount() {
      // Check authentication status
      const token = localStorage.getItem("token");
      this.setState({ isAuthenticated: !!token });
    }

    render() {
      if (!this.state.isAuthenticated) {
        return <Redirect to="/login" />;
      }

      return <WrappedComponent {...this.props} />;
    }
  };
};
```

2. **Render Props Pattern**

```javascript
class MouseTracker extends Component {
  constructor(props) {
    super(props);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove = (event) => {
    this.setState({
      x: event.clientX,
      y: event.clientY,
    });
  };

  render() {
    return (
      <div onMouseMove={this.handleMouseMove}>
        {this.props.render(this.state)}
      </div>
    );
  }
}

// Usage:
<MouseTracker
  render={({ x, y }) => (
    <h1>
      Mouse position: {x}, {y}
    </h1>
  )}
/>;
```
