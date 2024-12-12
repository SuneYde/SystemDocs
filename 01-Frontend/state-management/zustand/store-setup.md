# Zustand Store Setup Guide

Welcome to our comprehensive guide on setting up state management with Zustand! Zustand (German for "state") is a minimalist state management solution that makes managing application state feel natural and straightforward. Let's explore how to set it up and use it effectively.

### Understanding Zustand's Philosophy

Zustand takes a different approach from Redux and other state managers. Instead of creating a global store with actions and reducers, Zustand gives you a simple hook-based store that feels natural in the React ecosystem. Think of it as a sophisticated useState hook that can be shared across your entire application.

### Basic Store Setup

Let's start with a simple store setup:

```javascript
import create from "zustand";

// Create a store
const useStore = create((set) => ({
  // Define your initial state
  bears: 0,
  fish: 0,

  // Define your actions
  increaseBears: () => set((state) => ({ bears: state.bears + 1 })),
  increaseFish: () => set((state) => ({ fish: state.fish + 1 })),

  // Actions can also be more complex
  increasePopulation: () =>
    set((state) => ({
      bears: state.bears + 1,
      fish: state.fish + 5,
    })),

  // You can also set state directly
  removeAllAnimals: () => set({ bears: 0, fish: 0 }),
}));

// Using the store in a component is as simple as:
function BearCounter() {
  const bears = useStore((state) => state.bears);
  const increaseBears = useStore((state) => state.increaseBears);

  return (
    <div>
      <h1>{bears} bears</h1>
      <button onClick={increaseBears}>Add a bear</button>
    </div>
  );
}
```

### Creating a More Complex Store

Let's create a more realistic example with TypeScript and multiple slices of state:

```typescript
import create from "zustand";
import { devtools, persist } from "zustand/middleware";

// Define your store's types
interface User {
  id: string;
  name: string;
  email: string;
}

interface AuthState {
  user: User | null;
  isLoading: boolean;
  error: string | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  updateUser: (userData: Partial<User>) => void;
}

// Create the store with middleware
const useAuthStore = create<AuthState>()(
  devtools(
    persist(
      (set) => ({
        // Initial state
        user: null,
        isLoading: false,
        error: null,

        // Actions
        login: async (email, password) => {
          set({ isLoading: true, error: null });
          try {
            const response = await fetch("/api/login", {
              method: "POST",
              body: JSON.stringify({ email, password }),
            });
            const user = await response.json();
            set({ user, isLoading: false });
          } catch (error) {
            set({ error: error.message, isLoading: false });
          }
        },

        logout: () => {
          set({ user: null, error: null });
        },

        updateUser: (userData) => {
          set((state) => ({
            user: state.user ? { ...state.user, ...userData } : null,
          }));
        },
      }),
      {
        name: "auth-storage", // unique name for localStorage
        getStorage: () => localStorage, // Choose which storage to use
      }
    )
  )
);
```

### Using Slices for Better Organization

As your application grows, you might want to split your store into logical slices:

```typescript
import create from "zustand";
import { subscribeWithSelector } from "zustand/middleware";

// Define slice for user interface state
const createUISlice = (set) => ({
  theme: "light",
  sidebar: false,
  toggleTheme: () =>
    set((state) => ({
      theme: state.theme === "light" ? "dark" : "light",
    })),
  toggleSidebar: () =>
    set((state) => ({
      sidebar: !state.sidebar,
    })),
});

// Define slice for authentication
const createAuthSlice = (set) => ({
  user: null,
  setUser: (user) => set({ user }),
  logout: () => set({ user: null }),
});

// Combine slices into a single store
const useStore = create(
  subscribeWithSelector((set) => ({
    ...createUISlice(set),
    ...createAuthSlice(set),
  }))
);
```

### Advanced Patterns

#### 1. Computed Values with Selectors

```typescript
const useStore = create((set, get) => ({
  products: [],
  cart: {},

  // Computed value using get
  get totalPrice() {
    return Object.entries(get().cart).reduce((total, [id, quantity]) => {
      const product = get().products.find((p) => p.id === id);
      return total + (product?.price || 0) * quantity;
    }, 0);
  },

  addToCart: (productId) =>
    set((state) => ({
      cart: {
        ...state.cart,
        [productId]: (state.cart[productId] || 0) + 1,
      },
    })),
}));
```

#### 2. Async Actions with Loading States

```typescript
const useStore = create((set, get) => ({
  data: null,
  loading: false,
  error: null,

  fetchData: async () => {
    set({ loading: true, error: null });

    try {
      const response = await fetch("/api/data");
      const data = await response.json();

      set({ data, loading: false });
    } catch (error) {
      set({ error: error.message, loading: false });
    }
  },

  // Reset state
  reset: () => set({ data: null, loading: false, error: null }),
}));
```

#### 3. Middleware Integration

```typescript
const myMiddleware = (config) => (set, get, api) =>
  config(
    (...args) => {
      console.log("Before state change:", get());
      set(...args);
      console.log("After state change:", get());
    },
    get,
    api
  );

const useStore = create(
  myMiddleware(
    devtools(
      persist(
        (set) => ({
          // Your store configuration
        }),
        { name: "store" }
      )
    )
  )
);
```

#### 4. TypeScript Integration

```typescript
import create from "zustand";

interface BearState {
  bears: number;
  increasePopulation: () => void;
  removeAllBears: () => void;
}

const useBearStore = create<BearState>()((set) => ({
  bears: 0,
  increasePopulation: () => set((state) => ({ bears: state.bears + 1 })),
  removeAllBears: () => set({ bears: 0 }),
}));

// Type inference works automatically
const bears = useBearStore((state) => state.bears);
```

### Best Practices

1. **Keep Stores Focused**
   Create separate stores for different domains:

```typescript
// authStore.ts
const useAuthStore = create(/* auth store config */);

// uiStore.ts
const useUIStore = create(/* UI store config */);

// cartStore.ts
const useCartStore = create(/* cart store config */);
```

2. **Use Selectors Effectively**
   Create reusable selectors to access store state:

```typescript
const useUser = () => useStore((state) => state.user);
const useTheme = () => useStore((state) => state.theme);

// In components
function UserProfile() {
  const user = useUser();
  const theme = useTheme();
  // ...
}
```

3. **Handle Side Effects**
   Manage side effects within actions:

```typescript
const useStore = create((set, get) => ({
  saveSetting: async (setting) => {
    // Optimistic update
    set({ settings: { ...get().settings, [setting.key]: setting.value } });

    try {
      await api.saveSetting(setting);
    } catch (error) {
      // Revert on failure
      set({ settings: get().settings });
      throw error;
    }
  },
}));
```

Want to explore any of these patterns in more detail or see more specific examples? Let me know!
