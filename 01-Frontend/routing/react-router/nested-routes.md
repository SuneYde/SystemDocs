# Nested Routes in React Router

Welcome to our comprehensive guide on nested routes in React Router! Think of nested routes like organizing folders on your computer - just as you can have folders inside folders, you can have routes inside other routes. This creates a natural hierarchy in your application that matches how users think about navigating through different levels of content.

### Understanding the Concept

Nested routes allow you to create complex layouts where part of the page stays constant while another part changes based on the current route. Let's start with a simple example to illustrate this concept:

```javascript
import { Routes, Route, Outlet, Link, useParams } from "react-router-dom";

// First, let's create a layout component that will be shared
const DashboardLayout = () => {
  return (
    <div className="dashboard-layout">
      {/* This navigation will always be present */}
      <nav className="dashboard-nav">
        <Link to="/dashboard">Overview</Link>
        <Link to="/dashboard/profile">Profile</Link>
        <Link to="/dashboard/settings">Settings</Link>
      </nav>

      {/* Sidebar content */}
      <aside className="dashboard-sidebar">
        <h3>Quick Actions</h3>
        <ul>
          <li>
            <Link to="/dashboard/new">Create New</Link>
          </li>
          <li>
            <Link to="/dashboard/tasks">View Tasks</Link>
          </li>
        </ul>
      </aside>

      {/* This is where nested routes will render */}
      <main className="dashboard-main">
        <Outlet />
      </main>
    </div>
  );
};

// Create components for each nested route
const DashboardOverview = () => (
  <div>
    <h1>Dashboard Overview</h1>
    <div className="dashboard-stats">{/* Dashboard content */}</div>
  </div>
);

const UserProfile = () => (
  <div>
    <h1>User Profile</h1>
    <div className="profile-content">{/* Profile content */}</div>
  </div>
);

const Settings = () => (
  <div>
    <h1>Settings</h1>
    <div className="settings-content">{/* Settings content */}</div>
  </div>
);

// Set up the route configuration
const App = () => {
  return (
    <Routes>
      <Route path="/" element={<Home />} />

      {/* Dashboard and its nested routes */}
      <Route path="/dashboard" element={<DashboardLayout />}>
        <Route index element={<DashboardOverview />} />
        <Route path="profile" element={<UserProfile />} />
        <Route path="settings" element={<Settings />} />
      </Route>
    </Routes>
  );
};
```

### Creating Multi-Level Nested Routes

Let's explore a more complex example with multiple levels of nesting:

```javascript
// First, create our nested layout components
const ProductsLayout = () => {
  return (
    <div className="products-layout">
      <nav className="products-nav">
        <Link to="/products/electronics">Electronics</Link>
        <Link to="/products/books">Books</Link>
        <Link to="/products/clothing">Clothing</Link>
      </nav>

      {/* This will render the nested routes */}
      <Outlet />
    </div>
  );
};

const CategoryLayout = () => {
  const { category } = useParams();

  return (
    <div className="category-layout">
      <aside className="category-sidebar">
        <h3>{category} Categories</h3>
        <nav>
          <Link to="new">New Arrivals</Link>
          <Link to="popular">Popular</Link>
          <Link to="sale">On Sale</Link>
        </nav>
      </aside>

      {/* This renders the next level of nested routes */}
      <Outlet />
    </div>
  );
};

// Product list component that uses URL parameters
const ProductList = () => {
  const { category, subcategory } = useParams();
  const [products, setProducts] = useState([]);

  useEffect(() => {
    // Fetch products based on category and subcategory
    fetchProducts(category, subcategory).then((data) => setProducts(data));
  }, [category, subcategory]);

  return (
    <div className="product-list">
      <h2>
        {subcategory} {category}
      </h2>
      <div className="products-grid">
        {products.map((product) => (
          <ProductCard key={product.id} product={product} />
        ))}
      </div>
    </div>
  );
};

// Set up the multi-level routing
const App = () => {
  return (
    <Routes>
      <Route path="/" element={<MainLayout />}>
        {/* Products section with nested routes */}
        <Route path="products" element={<ProductsLayout />}>
          {/* Category routes */}
          <Route path=":category" element={<CategoryLayout />}>
            {/* Subcategory routes */}
            <Route index element={<ProductList />} />
            <Route path="new" element={<ProductList />} />
            <Route path="popular" element={<ProductList />} />
            <Route path="sale" element={<ProductList />} />

            {/* Individual product route */}
            <Route path="product/:productId" element={<ProductDetail />} />
          </Route>
        </Route>
      </Route>
    </Routes>
  );
};
```

### Working with Dynamic Nested Routes

Here's how to handle dynamic routes that might have varying levels of nesting:

```javascript
// Create a recursive route component
const DynamicRoute = ({ routes }) => {
  return (
    <Routes>
      {routes.map((route) => (
        <Route
          key={route.path}
          path={route.path}
          element={
            route.children ? (
              // If there are children, create another level of nesting
              <div className={`${route.name}-layout`}>
                <nav>
                  {route.children.map((child) => (
                    <Link key={child.path} to={`${route.path}/${child.path}`}>
                      {child.name}
                    </Link>
                  ))}
                </nav>
                <Outlet />
              </div>
            ) : (
              // If no children, render the component
              <route.component />
            )
          }
        >
          {route.children &&
            route.children.map((child) => (
              <Route
                key={child.path}
                path={child.path}
                element={<child.component />}
              />
            ))}
        </Route>
      ))}
    </Routes>
  );
};

// Define your route configuration
const routeConfig = [
  {
    path: "/dashboard",
    name: "dashboard",
    component: DashboardOverview,
    children: [
      {
        path: "analytics",
        name: "Analytics",
        component: Analytics,
        children: [
          {
            path: "reports",
            name: "Reports",
            component: Reports,
          },
        ],
      },
    ],
  },
];

// Use the dynamic router
const App = () => {
  return <DynamicRoute routes={routeConfig} />;
};
```

### Handling Data Loading in Nested Routes

When working with nested routes, you often need to load data at different levels:

```javascript
const CategoryLayout = () => {
  const { category } = useParams();
  const [categoryData, setCategoryData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const loadCategoryData = async () => {
      setLoading(true);
      try {
        const data = await fetchCategoryData(category);
        setCategoryData(data);
      } catch (error) {
        console.error("Failed to load category:", error);
      } finally {
        setLoading(false);
      }
    };

    loadCategoryData();
  }, [category]);

  if (loading) {
    return <LoadingSpinner />;
  }

  if (!categoryData) {
    return <NotFound />;
  }

  return (
    <div className="category-layout">
      <h1>{categoryData.name}</h1>
      <div className="category-description">{categoryData.description}</div>

      {/* Render nested routes */}
      <Outlet context={{ categoryData }} />
    </div>
  );
};

// Child component can access parent data
const ProductList = () => {
  const { categoryData } = useOutletContext();
  const [products, setProducts] = useState([]);

  useEffect(() => {
    // Use category data to fetch relevant products
    fetchProducts(categoryData.id).then(setProducts);
  }, [categoryData.id]);

  return (
    <div className="product-list">
      {products.map((product) => (
        <ProductCard
          key={product.id}
          product={product}
          categoryName={categoryData.name}
        />
      ))}
    </div>
  );
};
```

I hope this guide helps you understand how to effectively use nested routes in your React applications! Would you like to explore any specific aspect in more detail?
