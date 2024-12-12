# Breadcrumb Navigation in React Router

Breadcrumbs are a crucial navigation aid that helps users understand their location within an application's hierarchy. Think of them like trail markers on a hiking path - they show users where they are and how they got there. Let's explore how to implement them effectively using React Router.

### Understanding Breadcrumb Navigation

First, let's create a basic breadcrumb component that automatically generates navigation based on the current route:

```javascript
import { Link, useLocation, useMatches } from "react-router-dom";

const Breadcrumbs = () => {
  const location = useLocation();

  // Create breadcrumb items from the current path
  const getPathElements = () => {
    // Remove any trailing slashes and split the path
    const paths = location.pathname
      .replace(/\/+$/, "")
      .split("/")
      .filter(Boolean);

    // Generate breadcrumb items with accumulated paths
    return paths.map((path, index) => {
      // Build the full path up to this point
      const fullPath = "/" + paths.slice(0, index + 1).join("/");

      // Convert path to readable text
      const label = path
        .split("-")
        .map((word) => word.charAt(0).toUpperCase() + word.slice(1))
        .join(" ");

      return {
        label,
        path: fullPath,
      };
    });
  };

  const pathElements = getPathElements();

  // Don't render breadcrumbs on the home page
  if (location.pathname === "/") {
    return null;
  }

  return (
    <nav aria-label="Breadcrumb">
      <ol className="breadcrumbs">
        {/* Home link is always first */}
        <li>
          <Link to="/">Home</Link>
          <span aria-hidden="true">/</span>
        </li>

        {pathElements.map((element, index) => (
          <li
            key={element.path}
            // Mark the last item as current page
            aria-current={
              index === pathElements.length - 1 ? "page" : undefined
            }
          >
            {index === pathElements.length - 1 ? (
              <span>{element.label}</span>
            ) : (
              <>
                <Link to={element.path}>{element.label}</Link>
                <span aria-hidden="true">/</span>
              </>
            )}
          </li>
        ))}
      </ol>
    </nav>
  );
};
```

### Advanced Breadcrumb Implementation

Now, let's create a more sophisticated version that handles dynamic routes and custom labels:

```javascript
import {
  createContext,
  useContext,
  useMemo
} from 'react';

// Create a context for breadcrumb configuration
const BreadcrumbContext = createContext({});

// Provider component for breadcrumb configuration
export const BreadcrumbProvider = ({ children, config }) => {
  const value = useMemo(() => ({ config }), [config]);
  return (
    <BreadcrumbContext.Provider value={value}>
      {children}
    </BreadcrumbContext.Provider>
  );
};

const Breadcrumbs = () => {
  const location = useLocation();
  const { config } = useContext(BreadcrumbContext);

  // Get dynamic route parameters
  const matches = useMatches();

  // Generate breadcrumb data
  const breadcrumbs = useMemo(() => {
    const asArray = location.pathname
      .split('/')
      .filter(Boolean);

    let currentPath = '';

    return asArray.map((segment, index) => {
      currentPath += `/${segment}`;

      // Check if this segment has a dynamic configuration
      const segmentConfig = config[currentPath] || {};

      // If this is a dynamic segment (e.g., :id), get the label from route data
      const match = matches[index + 1]; // +1 because first match is always root
      const isDynamic = segment.startsWith(':');

      let label = segmentConfig.label;
      if (isDynamic && match?.data) {
        label = match.data.breadcrumbLabel || segment;
      }

      return {
        label: label || segment,
        path: currentPath,
        icon: segmentConfig.icon
      };
    });
  }, [location.pathname, matches, config]);

  // Add custom styling and animations
  return (
    <nav
      aria-label="Breadcrumb"
      className="breadcrumbs-container"
    >
      <ol className="breadcrumbs-list">
        <li className="breadcrumb-item">
          <Link to="/" className="home-link">
            <HomeIcon aria-hidden="true" />
            <span>Home</span>
          </Link>
          <ChevronRight
            className="separator"
            aria-hidden="true"
          />
        </li>

        {breadcrumbs.map((crumb, index) => (
          <li
            key={crumb.path}
            className="breadcrumb-item"
            aria-current={
              index === breadcrumbs.length - 1 ? 'page' : undefined
            }
          >
            {index === breadcrumbs.length - 1 ? (
              <span className="current-page">
                {crumb.icon && (
                  <crumb.icon
                    className="breadcrumb-icon"
                    aria-hidden="true"
                  />
                )}
                {crumb.label}
              </span>
            ) : (
              <>
                <Link to={crumb.path} className="breadcrumb-link">
                  {crumb.icon && (
                    <crumb.icon
                      className="breadcrumb-icon"
                      aria-hidden="true"
                    />
                  )}
                  {crumb.label}
                </Link>
                <ChevronRight
                  className="separator"
                  aria-hidden="true"
                />
              </>
            )}
          </li>
        ))}
      </ol>
    </nav>
  );
};

// Usage with configuration
const App = () => {
  const breadcrumbConfig = {
    '/products': {
      label: 'Products',
      icon: ShoppingBagIcon
    },
    '/products/:id': {
      // Dynamic label will be set from route data
      icon: BoxIcon
    },
    '/account': {
      label: 'My Account',
      icon: UserIcon
    }
  };

  return (
    <BreadcrumbProvider config={breadcrumbConfig}>
      <div className="app">
        <Breadcrumbs />
        {/* Rest of your app */}
      </div>
    </BreadcrumbProvider>
  );
};

// In your route components, set dynamic breadcrumb labels
const ProductPage = () => {
  const { id } = useParams();
  const { product, loading } = useProduct(id);

  // Set breadcrumb label in route loader or effect
  useEffect(() => {
    if (product) {
      document.title = product.name;
      // Update route data for breadcrumb
      window.history.replaceState(
        { breadcrumbLabel: product.name },
        ''
      );
    }
  }, [product]);

  return (
    // Component content
  );
};
```

### Mobile-Friendly Breadcrumbs

For mobile devices, we might want to implement a condensed version:

```javascript
const ResponsiveBreadcrumbs = () => {
  const [isMobile, setIsMobile] = useState(false);
  const breadcrumbs = useBreadcrumbs(); // Our previous implementation

  // Check viewport width
  useEffect(() => {
    const checkWidth = () => {
      setIsMobile(window.innerWidth < 768);
    };

    checkWidth();
    window.addEventListener("resize", checkWidth);
    return () => window.removeEventListener("resize", checkWidth);
  }, []);

  if (isMobile) {
    // Show only current page and parent
    const currentCrumb = breadcrumbs[breadcrumbs.length - 1];
    const parentCrumb = breadcrumbs[breadcrumbs.length - 2];

    return (
      <nav aria-label="Breadcrumb" className="mobile-breadcrumbs">
        <button onClick={() => window.history.back()} className="back-button">
          <ChevronLeft aria-hidden="true" />
          <span>{parentCrumb ? parentCrumb.label : "Back"}</span>
        </button>
        <span className="current-page" aria-current="page">
          {currentCrumb.label}
        </span>
      </nav>
    );
  }

  // Regular breadcrumbs for desktop
  return <Breadcrumbs />;
};
```

### Styling Your Breadcrumbs

Here's some CSS to make your breadcrumbs look professional:

```css
.breadcrumbs-container {
  padding: 1rem;
  background: var(--background-color);
  border-bottom: 1px solid var(--border-color);
}

.breadcrumbs-list {
  display: flex;
  flex-wrap: wrap;
  list-style: none;
  padding: 0;
  margin: 0;
  gap: 0.5rem;
}

.breadcrumb-item {
  display: flex;
  align-items: center;
  color: var(--text-secondary);
  font-size: 0.875rem;
}

.breadcrumb-link {
  display: flex;
  align-items: center;
  color: var(--text-primary);
  text-decoration: none;
  transition: color 0.2s;
}

.breadcrumb-link:hover {
  color: var(--primary-color);
}

.breadcrumb-icon {
  width: 1rem;
  height: 1rem;
  margin-right: 0.25rem;
}

.separator {
  margin: 0 0.5rem;
  color: var(--text-secondary);
}

.current-page {
  display: flex;
  align-items: center;
  color: var(--text-primary);
  font-weight: 500;
}

/* Mobile styles */
@media (max-width: 768px) {
  .mobile-breadcrumbs {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1rem;
  }

  .back-button {
    display: flex;
    align-items: center;
    gap: 0.25rem;
    border: none;
    background: none;
    color: var(--text-primary);
    font-size: 0.875rem;
    cursor: pointer;
  }
}
```

These implementations provide a solid foundation for breadcrumb navigation in your React application. Would you like me to explain any part in more detail?
