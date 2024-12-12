# Programmatic Navigation in React Router

Programmatic navigation allows you to control your application's routing behavior through code rather than user interactions with links. Think of it like having a remote control for your application's navigation - you can move users between different pages based on certain conditions, actions, or events.

### Understanding the useNavigate Hook

The foundation of programmatic navigation in React Router is the `useNavigate` hook. Let's explore how to use it effectively:

```javascript
import { useNavigate, useLocation } from "react-router-dom";

const LoginForm = () => {
  const navigate = useNavigate();
  const location = useLocation();

  // Get the page user was trying to access before login
  const from = location.state?.from || "/dashboard";

  const handleSubmit = async (event) => {
    event.preventDefault();
    try {
      // Attempt to log in
      await loginUser({
        email: event.target.email.value,
        password: event.target.password.value,
      });

      // After successful login, redirect to previous page
      navigate(from, {
        replace: true, // Replace login page in history
      });
    } catch (error) {
      console.error("Login failed:", error);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <h1>Login</h1>
      {from !== "/dashboard" && <p>You need to log in to visit {from}</p>}
      {/* Form fields */}
    </form>
  );
};
```

### Navigation with State and Replace

Sometimes you need to pass data along with navigation or modify the browser history. Here's how to handle these scenarios:

```javascript
const ProductManager = () => {
  const navigate = useNavigate();
  const location = useLocation();

  const handleDeleteProduct = async (productId) => {
    try {
      await deleteProduct(productId);

      // Navigate with success message
      navigate("/products", {
        state: {
          message: "Product deleted successfully",
          type: "success",
        },
      });
    } catch (error) {
      // Stay on same page but show error
      navigate(location.pathname, {
        state: {
          message: error.message,
          type: "error",
        },
        replace: true, // Don't add to history
      });
    }
  };

  return (
    <div>
      {/* Show message if it exists */}
      {location.state?.message && (
        <Alert type={location.state.type}>{location.state.message}</Alert>
      )}
      {/* Rest of component */}
    </div>
  );
};
```

### Conditional Navigation with Guards

Often you'll want to control navigation based on certain conditions. Here's how to implement navigation guards:

```javascript
const CheckoutFlow = () => {
  const navigate = useNavigate();
  const { cart, user } = useStore();

  // Check conditions before allowing navigation
  const proceedToPayment = () => {
    if (!user) {
      navigate("/login", {
        state: {
          from: "/checkout",
          message: "Please log in to continue checkout",
        },
      });
      return;
    }

    if (cart.items.length === 0) {
      navigate("/shop", {
        state: {
          message: "Your cart is empty",
        },
      });
      return;
    }

    if (!user.hasVerifiedEmail) {
      navigate("/verify-email", {
        state: {
          nextStep: "/checkout/payment",
        },
      });
      return;
    }

    // All conditions met, proceed to payment
    navigate("/checkout/payment");
  };

  return <button onClick={proceedToPayment}>Proceed to Payment</button>;
};
```

### Navigation with Query Parameters

Managing query parameters during navigation is a common requirement:

```javascript
const ProductSearch = () => {
  const navigate = useNavigate();
  const [searchParams] = useSearchParams();

  const updateFilters = (newFilters) => {
    // Get current query parameters
    const currentParams = Object.fromEntries(searchParams.entries());

    // Create updated query string
    const queryString = new URLSearchParams({
      ...currentParams,
      ...newFilters,
    }).toString();

    // Navigate with new query string
    navigate({
      pathname: location.pathname,
      search: `?${queryString}`,
    });
  };

  // Navigate with preserved parameters
  const navigateToProduct = (productId) => {
    navigate({
      pathname: `/products/${productId}`,
      search: location.search, // Preserve current query params
    });
  };

  return (
    <div>
      <FilterControls onChange={updateFilters} />
      <ProductList onProductClick={navigateToProduct} />
    </div>
  );
};
```

### Advanced Navigation Patterns

#### Handling Navigation Confirmation

Sometimes you need to confirm navigation to prevent accidental data loss:

```javascript
const EditForm = () => {
  const navigate = useNavigate();
  const [isDirty, setIsDirty] = useState(false);

  // Prompt user before navigating away
  useEffect(() => {
    if (isDirty) {
      const handleBeforeUnload = (e) => {
        e.preventDefault();
        e.returnValue = "";
      };

      window.addEventListener("beforeunload", handleBeforeUnload);
      return () => {
        window.removeEventListener("beforeunload", handleBeforeUnload);
      };
    }
  }, [isDirty]);

  const handleNavigation = (to) => {
    if (isDirty) {
      const confirmed = window.confirm(
        "You have unsaved changes. Are you sure you want to leave?"
      );

      if (!confirmed) return;
    }

    navigate(to);
  };

  return (
    <div>
      <form onChange={() => setIsDirty(true)}>{/* Form fields */}</form>
      <button onClick={() => handleNavigation("/dashboard")}>Cancel</button>
    </div>
  );
};
```

#### Navigation with History Management

Sometimes you need more control over the browser's history:

```javascript
const WizardFlow = () => {
  const navigate = useNavigate();
  const location = useLocation();

  // Keep track of wizard steps in history
  const navigateToStep = (stepNumber) => {
    navigate(`/wizard/step/${stepNumber}`, {
      state: {
        ...location.state,
        completedSteps: [
          ...(location.state?.completedSteps || []),
          stepNumber - 1,
        ],
      },
    });
  };

  // Go back to last valid step
  const handleBack = () => {
    const completedSteps = location.state?.completedSteps || [];
    const lastStep = completedSteps[completedSteps.length - 1];

    if (lastStep !== undefined) {
      navigate(`/wizard/step/${lastStep}`, {
        state: {
          ...location.state,
          completedSteps: completedSteps.slice(0, -1),
        },
      });
    } else {
      navigate("/");
    }
  };

  return (
    <div>
      <button onClick={handleBack}>Back</button>
      <button onClick={() => navigateToStep(currentStep + 1)}>Next</button>
    </div>
  );
};
```

Would you like me to explain any of these concepts in more detail?
