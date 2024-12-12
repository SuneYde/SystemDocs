# Route Parameters in React Router

Route parameters are essential building blocks that allow your React application to handle dynamic content based on URL values. Think of them as variables in your URLs - they let you create flexible routes that can handle many different cases without needing to define each one separately. Let's explore how to use them effectively.

### Understanding Route Parameters

Route parameters come in three main types:

1. URL Parameters (`:paramName`)
2. Query Parameters (`?name=value`)
3. Optional Parameters (`:paramName?`)

Let's explore each type in detail:

### URL Parameters

URL parameters are the most common type, defined by adding a colon before the parameter name in your route path:

```javascript
import { useParams, useNavigate, Routes, Route } from "react-router-dom";

// Component that uses URL parameters
const UserProfile = () => {
  // Extract parameters from the URL
  const { userId } = useParams();
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Fetch user data based on the URL parameter
    const fetchUser = async () => {
      try {
        setLoading(true);
        const response = await fetch(`/api/users/${userId}`);
        const data = await response.json();
        setUser(data);
      } catch (error) {
        console.error("Failed to fetch user:", error);
      } finally {
        setLoading(false);
      }
    };

    fetchUser();
  }, [userId]); // Re-fetch when userId changes

  if (loading) return <div>Loading...</div>;
  if (!user) return <div>User not found</div>;

  return (
    <div className="user-profile">
      <h1>{user.name}'s Profile</h1>
      <div className="user-details">
        <p>Email: {user.email}</p>
        <p>Member since: {new Date(user.joinDate).toLocaleDateString()}</p>
      </div>
    </div>
  );
};

// Set up routes with parameters
const App = () => {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/users/:userId" element={<UserProfile />} />

      {/* Nested routes can also use parameters */}
      <Route path="/users/:userId/posts/:postId" element={<UserPost />} />
    </Routes>
  );
};
```

### Query Parameters

Query parameters are key-value pairs in the URL after the `?` symbol. They're perfect for handling filters, searches, and pagination:

```javascript
import { useSearchParams, useLocation } from "react-router-dom";

const ProductList = () => {
  // Get and set query parameters
  const [searchParams, setSearchParams] = useSearchParams();
  const location = useLocation();

  // Extract values from query parameters
  const page = parseInt(searchParams.get("page")) || 1;
  const category = searchParams.get("category") || "all";
  const sortBy = searchParams.get("sort") || "newest";

  // Update query parameters
  const updateFilters = (newFilters) => {
    // Preserve existing parameters while updating new ones
    const newParams = new URLSearchParams(searchParams);
    Object.entries(newFilters).forEach(([key, value]) => {
      if (value) {
        newParams.set(key, value);
      } else {
        newParams.delete(key);
      }
    });
    setSearchParams(newParams);
  };

  // Use the parameters to fetch data
  useEffect(() => {
    const fetchProducts = async () => {
      const queryString = searchParams.toString();
      const response = await fetch(`/api/products?${queryString}`);
      const data = await response.json();
      setProducts(data);
    };

    fetchProducts();
  }, [searchParams]); // Re-fetch when any parameter changes

  return (
    <div className="product-list">
      {/* Filter controls */}
      <div className="filters">
        <select
          value={category}
          onChange={(e) => updateFilters({ category: e.target.value })}
        >
          <option value="all">All Categories</option>
          <option value="electronics">Electronics</option>
          <option value="books">Books</option>
        </select>

        <select
          value={sortBy}
          onChange={(e) => updateFilters({ sort: e.target.value })}
        >
          <option value="newest">Newest First</option>
          <option value="price-low">Price: Low to High</option>
          <option value="price-high">Price: High to Low</option>
        </select>
      </div>

      {/* Product grid */}
      <div className="products-grid">
        {products.map((product) => (
          <ProductCard key={product.id} product={product} />
        ))}
      </div>

      {/* Pagination */}
      <div className="pagination">
        <button
          onClick={() => updateFilters({ page: page - 1 })}
          disabled={page === 1}
        >
          Previous
        </button>
        <span>Page {page}</span>
        <button
          onClick={() => updateFilters({ page: page + 1 })}
          disabled={!hasMorePages}
        >
          Next
        </button>
      </div>
    </div>
  );
};
```

### Optional Parameters

Optional parameters can be handled by creating multiple routes or using a single route with optional segments:

```javascript
import { useParams, useNavigate } from "react-router-dom";

const ProjectView = () => {
  const { projectId, taskId } = useParams();
  const navigate = useNavigate();

  // Handle both project-level and task-level views
  const [project, setProject] = useState(null);
  const [selectedTask, setSelectedTask] = useState(null);

  useEffect(() => {
    // Fetch project data
    const fetchProject = async () => {
      const data = await fetchProjectData(projectId);
      setProject(data);
    };

    fetchProject();
  }, [projectId]);

  useEffect(() => {
    // Fetch task data if taskId is present
    if (taskId) {
      const fetchTask = async () => {
        const data = await fetchTaskData(projectId, taskId);
        setSelectedTask(data);
      };

      fetchTask();
    } else {
      setSelectedTask(null);
    }
  }, [projectId, taskId]);

  return (
    <div className="project-view">
      {/* Project information */}
      <div className="project-info">
        <h1>{project?.name}</h1>
        <p>{project?.description}</p>
      </div>

      {/* Task list */}
      <div className="task-list">
        {project?.tasks.map((task) => (
          <div
            key={task.id}
            className={`task ${task.id === taskId ? "selected" : ""}`}
            onClick={() => navigate(`/projects/${projectId}/tasks/${task.id}`)}
          >
            {task.title}
          </div>
        ))}
      </div>

      {/* Selected task details */}
      {selectedTask && (
        <div className="task-details">
          <h2>{selectedTask.title}</h2>
          <p>{selectedTask.description}</p>
          {/* More task details */}
        </div>
      )}
    </div>
  );
};

// Route setup with optional parameter
const App = () => {
  return (
    <Routes>
      <Route path="/projects/:projectId" element={<ProjectView />} />
      <Route
        path="/projects/:projectId/tasks/:taskId"
        element={<ProjectView />}
      />
    </Routes>
  );
};
```

### Advanced Parameter Handling

Sometimes you need to handle more complex parameter scenarios:

```javascript
import { generatePath } from 'react-router-dom';

// Create a flexible path generator
const createProjectPath = ({ projectId, taskId, view }) => {
  const basePath = '/projects/:projectId';
  const taskPath = taskId ? '/tasks/:taskId' : '';
  const viewPath = view ? '/:view' : '';

  return generatePath(basePath + taskPath + viewPath, {
    projectId,
    taskId,
    view
  });
};

// Component using advanced parameter handling
const ProjectManager = () => {
  const navigate = useNavigate();
  const { projectId, taskId, view } = useParams();
  const [searchParams] = useSearchParams();

  // Navigate with preserved query parameters
  const navigateToTask = (newTaskId) => {
    const path = createProjectPath({
      projectId,
      taskId: newTaskId,
      view
    });
    navigate(`${path}?${searchParams.toString()}`);
  };

  // Handle parameter validation
  useEffect(() => {
    const validateParameters = async () => {
      try {
        await validateProjectAccess(projectId);
        if (taskId) {
          await validateTaskAccess(projectId, taskId);
        }
      } catch (error) {
        // Handle invalid parameters
        navigate('/projects', {
          replace: true,
          state: { error: error.message }
        });
      }
    };

    validateParameters();
  }, [projectId, taskId]);

  return (
    // Component content
  );
};
```

Would you like me to explain any of these concepts in more detail?
