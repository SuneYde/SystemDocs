# Protected Routes in React Router

Protected routes are a crucial part of any web application that needs to restrict access to certain pages based on user authentication or permissions. Think of them as security checkpoints in your application - they ensure that only authorized users can access sensitive areas. Let's explore how to implement them effectively.

### Understanding Protected Routes

At its core, a protected route is a wrapper around a regular route that checks certain conditions (like authentication status) before allowing access to the component. If the conditions aren't met, it redirects the user to another location, typically the login page.

Let's start with a basic implementation:

```javascript
import { Navigate, useLocation, useNavigate } from "react-router-dom";

// First, let's create our authentication context to manage user state
const AuthContext = createContext(null);

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  // Check if user is logged in when the app loads
  useEffect(() => {
    const checkAuth = async () => {
      try {
        // Check for saved auth token
        const token = localStorage.getItem("token");
        if (token) {
          const userData = await validateToken(token);
          setUser(userData);
        }
      } catch (error) {
        console.error("Auth check failed:", error);
      } finally {
        setLoading(false);
      }
    };

    checkAuth();
  }, []);

  const value = {
    user,
    loading,
    login: async (credentials) => {
      const userData = await loginUser(credentials);
      setUser(userData);
      localStorage.setItem("token", userData.token);
    },
    logout: () => {
      setUser(null);
      localStorage.removeItem("token");
    },
  };

  if (loading) {
    return <div>Loading...</div>;
  }

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
};

// Create a hook for easy access to auth context
const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error("useAuth must be used within AuthProvider");
  }
  return context;
};

// Now, let's create our Protected Route component
const ProtectedRoute = ({ children }) => {
  const { user, loading } = useAuth();
  const location = useLocation();

  // If authentication is still being checked, show loading
  if (loading) {
    return <div>Checking authentication...</div>;
  }

  // If not authenticated, redirect to login
  if (!user) {
    // Save the attempted URL for redirecting after login
    return <Navigate to="/login" state={{ from: location.pathname }} replace />;
  }

  // If authenticated, render the protected component
  return children;
};

// Implementation in your app
const App = () => {
  return (
    <AuthProvider>
      <BrowserRouter>
        <Routes>
          {/* Public routes */}
          <Route path="/" element={<Home />} />
          <Route path="/login" element={<Login />} />
          <Route path="/about" element={<About />} />

          {/* Protected routes */}
          <Route
            path="/dashboard"
            element={
              <ProtectedRoute>
                <Dashboard />
              </ProtectedRoute>
            }
          />
          <Route
            path="/profile"
            element={
              <ProtectedRoute>
                <UserProfile />
              </ProtectedRoute>
            }
          />
        </Routes>
      </BrowserRouter>
    </AuthProvider>
  );
};
```

### Advanced Protection with Role-Based Access

Often you need more granular control over who can access what. Let's implement role-based route protection:

```javascript
// Create a more sophisticated ProtectedRoute component
const ProtectedRoute = ({
  children,
  requiredRoles = [],
  requiredPermissions = [],
}) => {
  const { user, loading } = useAuth();
  const location = useLocation();

  if (loading) {
    return <div>Checking authentication...</div>;
  }

  if (!user) {
    return <Navigate to="/login" state={{ from: location.pathname }} replace />;
  }

  // Check for required roles
  if (requiredRoles.length > 0 && !requiredRoles.includes(user.role)) {
    return (
      <Navigate
        to="/unauthorized"
        state={{
          message: "Insufficient role permissions",
        }}
        replace
      />
    );
  }

  // Check for specific permissions
  if (requiredPermissions.length > 0) {
    const hasAllPermissions = requiredPermissions.every((permission) =>
      user.permissions.includes(permission)
    );

    if (!hasAllPermissions) {
      return (
        <Navigate
          to="/unauthorized"
          state={{
            message: "Missing required permissions",
          }}
          replace
        />
      );
    }
  }

  return children;
};

// Using role-based protection
const App = () => {
  return (
    <AuthProvider>
      <BrowserRouter>
        <Routes>
          {/* Public routes */}
          <Route path="/" element={<Home />} />
          <Route path="/login" element={<Login />} />

          {/* Admin-only routes */}
          <Route
            path="/admin"
            element={
              <ProtectedRoute requiredRoles={["admin"]}>
                <AdminDashboard />
              </ProtectedRoute>
            }
          />

          {/* Routes requiring specific permissions */}
          <Route
            path="/users/manage"
            element={
              <ProtectedRoute requiredPermissions={["users.edit"]}>
                <UserManagement />
              </ProtectedRoute>
            }
          />
        </Routes>
      </BrowserRouter>
    </AuthProvider>
  );
};
```

### Handling Redirect After Login

When users are redirected to login, we want to send them back to their original destination:

```javascript
const Login = () => {
  const navigate = useNavigate();
  const location = useLocation();
  const { login } = useAuth();

  // Get the page they were trying to access
  const from = location.state?.from || "/dashboard";

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      await login({
        email: e.target.email.value,
        password: e.target.password.value,
      });

      // Redirect them back to the page they tried to visit
      navigate(from, { replace: true });
    } catch (error) {
      console.error("Login failed:", error);
    }
  };

  return (
    <div>
      <h1>Login</h1>
      <p>You need to login to access: {from}</p>
      <form onSubmit={handleSubmit}>{/* Form fields */}</form>
    </div>
  );
};
```

### Persisting Authentication State

To prevent users from losing their authentication state on page refresh:

```javascript
const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  // Check authentication status on mount
  useEffect(() => {
    const initAuth = async () => {
      try {
        // Check for stored authentication data
        const token = localStorage.getItem("token");
        if (!token) {
          setLoading(false);
          return;
        }

        // Validate token with backend
        const response = await fetch("/api/auth/validate", {
          headers: {
            Authorization: `Bearer ${token}`,
          },
        });

        if (response.ok) {
          const userData = await response.json();
          setUser(userData);
        } else {
          // Token is invalid, clear storage
          localStorage.removeItem("token");
        }
      } catch (error) {
        console.error("Auth initialization failed:", error);
      } finally {
        setLoading(false);
      }
    };

    initAuth();
  }, []);

  return (
    <AuthContext.Provider
      value={{
        user,
        loading,
        login: async (credentials) => {
          const response = await fetch("/api/auth/login", {
            method: "POST",
            body: JSON.stringify(credentials),
          });

          const data = await response.json();

          if (response.ok) {
            setUser(data.user);
            localStorage.setItem("token", data.token);
            return data.user;
          } else {
            throw new Error(data.message);
          }
        },
        logout: () => {
          setUser(null);
          localStorage.removeItem("token");
        },
      }}
    >
      {children}
    </AuthContext.Provider>
  );
};
```

### Handling Expired Sessions

Add middleware to handle token expiration:

```javascript
// Create an axios instance with interceptors
const api = axios.create({
  baseURL: "/api",
});

api.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;

    // If the error is due to an expired token
    if (error.response.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;

      try {
        // Attempt to refresh the token
        const response = await axios.post("/api/auth/refresh");
        const { token } = response.data;

        // Update stored token
        localStorage.setItem("token", token);

        // Retry the original request
        originalRequest.headers["Authorization"] = `Bearer ${token}`;
        return axios(originalRequest);
      } catch (refreshError) {
        // If refresh fails, redirect to login
        window.location.href = "/login";
        return Promise.reject(refreshError);
      }
    }

    return Promise.reject(error);
  }
);
```

Want to explore any of these patterns in more detail or see more specific examples? Let me know!
