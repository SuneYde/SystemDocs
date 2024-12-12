# Advanced Zustand Patterns

Welcome to our deep dive into advanced Zustand patterns! While Zustand is simple on the surface, it offers powerful capabilities for handling complex state management scenarios. Let's explore these advanced patterns and see how they can make your state management more elegant and maintainable.

### Pattern 1: Computed Values and Derived State

Computed values are values that automatically update based on changes to other state. Zustand can handle these elegantly:

```typescript
import create from "zustand";

interface CartState {
  items: Array<{
    id: string;
    price: number;
    quantity: number;
  }>;
  // Derived state methods
  getTotalItems: () => number;
  getTotalPrice: () => number;
  // Actions
  addItem: (item: CartItem) => void;
  removeItem: (id: string) => void;
}

const useCartStore = create<CartState>()((set, get) => ({
  items: [],

  // Computed values as methods
  getTotalItems: () => {
    return get().items.reduce((sum, item) => sum + item.quantity, 0);
  },

  getTotalPrice: () => {
    return get().items.reduce(
      (sum, item) => sum + item.price * item.quantity,
      0
    );
  },

  addItem: (newItem) => {
    set((state) => {
      const existingItem = state.items.find((item) => item.id === newItem.id);

      if (existingItem) {
        return {
          items: state.items.map((item) =>
            item.id === newItem.id
              ? { ...item, quantity: item.quantity + 1 }
              : item
          ),
        };
      }

      return {
        items: [...state.items, { ...newItem, quantity: 1 }],
      };
    });
  },

  removeItem: (id) => {
    set((state) => ({
      items: state.items.filter((item) => item.id !== id),
    }));
  },
}));
```

### Pattern 2: Action Composition and Middleware

You can create complex workflows by composing actions and adding middleware for cross-cutting concerns:

```typescript
import create from "zustand";

// Create middleware for logging
const logMiddleware = (config) => (set, get, api) =>
  config(
    (...args) => {
      console.log("Before State:", get());
      set(...args);
      console.log("After State:", get());
    },
    get,
    api
  );

// Create middleware for validation
const validateMiddleware = (config) => (set, get, api) =>
  config(
    (...args) => {
      const nextState =
        typeof args[0] === "function" ? args[0](get()) : args[0];

      // Validate state before updating
      if (validateState(nextState)) {
        set(...args);
      } else {
        console.error("Invalid state transition");
      }
    },
    get,
    api
  );

// Complex store with composed actions
const useStore = create(
  logMiddleware(
    validateMiddleware((set, get) => ({
      user: null,
      cart: [],
      orders: [],

      // Complex action composition
      checkout: async () => {
        const state = get();
        const { user, cart } = state;

        if (!user) throw new Error("Must be logged in");
        if (cart.length === 0) throw new Error("Cart is empty");

        try {
          // Start checkout process
          set({ checkoutStatus: "processing" });

          // Create order
          const order = await createOrder(cart, user);

          // Update multiple pieces of state
          set((state) => ({
            orders: [...state.orders, order],
            cart: [],
            checkoutStatus: "completed",
            lastOrderId: order.id,
          }));
        } catch (error) {
          set({ checkoutStatus: "error", error: error.message });
        }
      },
    }))
  )
);
```

### Pattern 3: Sliced Stores with TypeScript

Breaking down large stores into smaller, typed slices while maintaining type safety:

```typescript
import create from "zustand";

// Define slice types
interface UISlice {
  theme: "light" | "dark";
  sidebar: boolean;
  toggleTheme: () => void;
  toggleSidebar: () => void;
}

interface AuthSlice {
  user: User | null;
  isLoading: boolean;
  login: (credentials: Credentials) => Promise<void>;
  logout: () => void;
}

// Create slices
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

const createAuthSlice = (set) => ({
  user: null,
  isLoading: false,
  login: async (credentials) => {
    set({ isLoading: true });
    try {
      const user = await loginUser(credentials);
      set({ user, isLoading: false });
    } catch (error) {
      set({ isLoading: false });
      throw error;
    }
  },
  logout: () => set({ user: null }),
});

// Combine slices
type StoreState = UISlice & AuthSlice;

const useStore = create<StoreState>()(
  devtools((...args) => ({
    ...createUISlice(...args),
    ...createAuthSlice(...args),
  }))
);
```

### Pattern 4: Optimistic Updates and Rollbacks

Handle optimistic updates with automatic rollback on failure:

```typescript
interface TodoState {
  todos: Todo[];
  addTodo: (text: string) => Promise<void>;
  updateTodo: (id: string, text: string) => Promise<void>;
  deleteTodo: (id: string) => Promise<void>;
}

const useTodoStore = create<TodoState>()((set, get) => ({
  todos: [],

  addTodo: async (text) => {
    // Generate temporary ID
    const tempId = `temp-${Date.now()}`;
    const newTodo = { id: tempId, text, completed: false };

    // Optimistically add todo
    set((state) => ({
      todos: [...state.todos, newTodo],
    }));

    try {
      // Attempt to save to server
      const savedTodo = await api.createTodo(text);

      // Update with real ID from server
      set((state) => ({
        todos: state.todos.map((todo) =>
          todo.id === tempId ? savedTodo : todo
        ),
      }));
    } catch (error) {
      // Rollback on failure
      set((state) => ({
        todos: state.todos.filter((todo) => todo.id !== tempId),
      }));
      throw error;
    }
  },

  updateTodo: async (id, text) => {
    // Store previous state
    const previousTodos = get().todos;

    // Optimistically update
    set((state) => ({
      todos: state.todos.map((todo) =>
        todo.id === id ? { ...todo, text } : todo
      ),
    }));

    try {
      await api.updateTodo(id, text);
    } catch (error) {
      // Rollback to previous state
      set({ todos: previousTodos });
      throw error;
    }
  },
}));
```

### Pattern 5: Subscription Management

Handle complex subscriptions and cleanup:

```typescript
const useRealTimeStore = create((set, get) => {
  let socket: WebSocket | null = null;
  let heartbeat: NodeJS.Timer | null = null;

  return {
    messages: [],
    status: "disconnected",

    connect: () => {
      if (socket) return;

      socket = new WebSocket("ws://api.example.com");

      socket.onopen = () => {
        set({ status: "connected" });

        // Set up heartbeat
        heartbeat = setInterval(() => {
          socket?.send("ping");
        }, 30000);
      };

      socket.onmessage = (event) => {
        set((state) => ({
          messages: [...state.messages, JSON.parse(event.data)],
        }));
      };

      socket.onclose = () => {
        set({ status: "disconnected" });
        clearInterval(heartbeat!);
        socket = null;
        heartbeat = null;
      };
    },

    disconnect: () => {
      socket?.close();
      set({ status: "disconnected" });
    },

    // Cleanup method for use in useEffect
    cleanup: () => {
      if (heartbeat) clearInterval(heartbeat);
      if (socket) socket.close();
    },
  };
});

// Usage in component
function Chat() {
  const { connect, disconnect, cleanup } = useRealTimeStore();

  useEffect(() => {
    connect();
    return () => cleanup();
  }, []);

  return <div>/* Chat UI */</div>;
}
```

These patterns demonstrate how Zustand can elegantly handle complex state management scenarios while maintaining its simple and flexible nature. Want to explore any of these patterns in more detail or see more specific examples? Let me know!
