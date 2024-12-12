# Basic Routing with React Router

Welcome to our guide on basic routing with React Router! Think of routing in React like creating a map for your application - it helps users navigate between different "locations" (pages or views) in your app. Let's learn how to set this up step by step.

### Understanding React Router Fundamentals

React Router enables navigation between different components in your React application without refreshing the page. This creates a smooth, app-like experience for users. Let's start with the basic setup:

```javascript
import { BrowserRouter, Routes, Route, Link, NavLink } from "react-router-dom";

// First, let's create some simple pages
const Home = () => {
  return (
    <div>
      <h1>Welcome Home!</h1>
      <p>This is the main page of our application.</p>
    </div>
  );
};

const About = () => {
  return (
    <div>
      <h1>About Us</h1>
      <p>Learn more about our company and mission.</p>
    </div>
  );
};

const Contact = () => {
  return (
    <div>
      <h1>Contact Us</h1>
      <p>Get in touch with our team.</p>
    </div>
  );
};

// Now, let's create our main App component with routing
const App = () => {
  return (
    <BrowserRouter>
      {/* Navigation menu */}
      <nav className="main-nav">
        <NavLink
          to="/"
          className={({ isActive }) => (isActive ? "active-link" : "")}
        >
          Home
        </NavLink>

        <NavLink
          to="/about"
          className={({ isActive }) => (isActive ? "active-link" : "")}
        >
          About
        </NavLink>

        <NavLink
          to="/contact"
          className={({ isActive }) => (isActive ? "active-link" : "")}
        >
          Contact
        </NavLink>
      </nav>

      {/* Route definitions */}
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/contact" element={<Contact />} />

        {/* Catch all route for 404 pages */}
        <Route
          path="*"
          element={
            <div>
              <h1>404: Page Not Found</h1>
              <Link to="/">Go Home</Link>
            </div>
          }
        />
      </Routes>
    </BrowserRouter>
  );
};
```

### Adding a Layout Component

It's common to have a consistent layout across pages. Here's how to implement that:

```javascript
// Create a layout component that wraps your pages
const Layout = ({ children }) => {
  return (
    <div className="app-layout">
      <header>
        <nav>
          <NavLink to="/">Home</NavLink>
          <NavLink to="/about">About</NavLink>
          <NavLink to="/contact">Contact</NavLink>
        </nav>
      </header>

      <main>
        {/* This is where your page content will render */}
        {children}
      </main>

      <footer>
        <p>© 2024 Your Company</p>
      </footer>
    </div>
  );
};

// Use the layout in your routes
const App = () => {
  return (
    <BrowserRouter>
      <Layout>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/contact" element={<Contact />} />
          <Route path="*" element={<NotFound />} />
        </Routes>
      </Layout>
    </BrowserRouter>
  );
};
```

### Working with Route Parameters

Often you'll need to pass parameters in your routes. Here's how to handle that:

```javascript
// Component that uses route parameters
const UserProfile = () => {
  // Get the userId from the URL
  const { userId } = useParams();
  const [user, setUser] = useState(null);

  useEffect(() => {
    // Fetch user data based on the ID
    const fetchUser = async () => {
      try {
        const response = await fetch(`/api/users/${userId}`);
        const data = await response.json();
        setUser(data);
      } catch (error) {
        console.error("Failed to fetch user:", error);
      }
    };

    fetchUser();
  }, [userId]);

  if (!user) return <div>Loading...</div>;

  return (
    <div>
      <h1>{user.name}'s Profile</h1>
      <p>Email: {user.email}</p>
    </div>
  );
};

// Add the route with a parameter
const App = () => {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/users/:userId" element={<UserProfile />} />
      </Routes>
    </BrowserRouter>
  );
};
```

### Handling Navigation Programmatically

Sometimes you need to navigate programmatically (like after a form submission):

```javascript
const LoginForm = () => {
  const navigate = useNavigate();
  const [formData, setFormData] = useState({
    email: "",
    password: "",
  });

  const handleSubmit = async (e) => {
    e.preventDefault();

    try {
      const response = await loginUser(formData);
      // If login successful, navigate to dashboard
      navigate("/dashboard", {
        // Pass state to the next route
        state: { user: response.user },
      });
    } catch (error) {
      console.error("Login failed:", error);
    }
  };

  return <form onSubmit={handleSubmit}>{/* Form fields here */}</form>;
};
```

### Handling Query Parameters

Query parameters are useful for filters, search terms, and pagination:

```javascript
const ProductList = () => {
  const [searchParams, setSearchParams] = useSearchParams();
  const page = parseInt(searchParams.get("page")) || 1;
  const category = searchParams.get("category") || "all";

  const updateFilters = (newFilters) => {
    setSearchParams({
      ...Object.fromEntries(searchParams),
      ...newFilters,
    });
  };

  return (
    <div>
      <select
        value={category}
        onChange={(e) => updateFilters({ category: e.target.value })}
      >
        <option value="all">All Categories</option>
        <option value="electronics">Electronics</option>
        <option value="books">Books</option>
      </select>

      {/* Product list here */}

      <div className="pagination">
        <button
          onClick={() => updateFilters({ page: page - 1 })}
          disabled={page === 1}
        >
          Previous
        </button>
        <span>Page {page}</span>
        <button onClick={() => updateFilters({ page: page + 1 })}>Next</button>
      </div>
    </div>
  );
};
```

### Route Guards and Protected Routes

Implement protected routes that require authentication:

```javascript
const PrivateRoute = ({ children }) => {
  const { isAuthenticated, isLoading } = useAuth();
  const location = useLocation();

  if (isLoading) {
    return <div>Loading...</div>;
  }

  if (!isAuthenticated) {
    // Redirect to login page, but save the attempted URL
    return <Navigate to="/login" state={{ from: location.pathname }} replace />;
  }

  return children;
};

// Use the protected route
const App = () => {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/login" element={<Login />} />

        {/* Protected routes */}
        <Route
          path="/dashboard"
          element={
            <PrivateRoute>
              <Dashboard />
            </PrivateRoute>
          }
        />

        <Route
          path="/profile"
          element={
            <PrivateRoute>
              <UserProfile />
            </PrivateRoute>
          }
        />
      </Routes>
    </BrowserRouter>
  );
};
```

Remember: Good routing is essential for creating a smooth user experience. Need help implementing any of these patterns or want to explore more advanced routing concepts? Let me know!
