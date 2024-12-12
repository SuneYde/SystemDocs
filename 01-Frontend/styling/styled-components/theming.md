# Theming in Styled Components

When building modern applications, having a consistent and flexible theming system is crucial. Styled-components provides a powerful theming solution that lets you manage your application's visual identity effectively. Let's explore how to implement theming from the ground up.

### Setting Up the Theme Provider

At its core, theming in styled-components works through React's context API. We'll start by creating and implementing a basic theme:

```javascript
import { ThemeProvider } from "styled-components";

// First, let's define our theme object
const theme = {
  colors: {
    primary: {
      main: "#0070f3",
      light: "#3291ff",
      dark: "#0051a2",
    },
    secondary: {
      main: "#6b7280",
      light: "#9ca3af",
      dark: "#4b5563",
    },
    success: "#10b981",
    error: "#ef4444",
    warning: "#f59e0b",
    text: {
      primary: "#1f2937",
      secondary: "#6b7280",
      disabled: "#9ca3af",
    },
  },
  spacing: {
    xs: "4px",
    sm: "8px",
    md: "16px",
    lg: "24px",
    xl: "32px",
  },
  typography: {
    fontFamily: {
      sans: '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif',
      serif: 'Georgia, Cambria, "Times New Roman", Times, serif',
      mono: 'Menlo, Monaco, Consolas, "Liberation Mono", monospace',
    },
    fontSize: {
      xs: "12px",
      sm: "14px",
      md: "16px",
      lg: "18px",
      xl: "20px",
      "2xl": "24px",
    },
    fontWeight: {
      normal: 400,
      medium: 500,
      semibold: 600,
      bold: 700,
    },
  },
  breakpoints: {
    sm: "640px",
    md: "768px",
    lg: "1024px",
    xl: "1280px",
  },
  shadows: {
    sm: "0 1px 2px rgba(0, 0, 0, 0.05)",
    md: "0 4px 6px rgba(0, 0, 0, 0.1)",
    lg: "0 10px 15px rgba(0, 0, 0, 0.1)",
  },
};

// Now, let's wrap our application with ThemeProvider
function App() {
  return (
    <ThemeProvider theme={theme}>
      {/* Your app components go here */}
    </ThemeProvider>
  );
}
```

### Creating Theme-Aware Components

Now that we've set up our theme, let's create components that use these theme values:

```javascript
import styled from "styled-components";

// A button that uses theme values
const Button = styled.button`
  /* Base styles using theme values */
  padding: ${({ theme }) => `${theme.spacing.sm} ${theme.spacing.md}`};
  font-family: ${({ theme }) => theme.typography.fontFamily.sans};
  font-size: ${({ theme }) => theme.typography.fontSize.md};
  font-weight: ${({ theme }) => theme.typography.fontWeight.medium};
  border-radius: 4px;
  border: none;
  cursor: pointer;
  transition: all 0.2s ease;

  /* Different variants using theme colors */
  ${({ variant = "primary", theme }) => {
    const colors = theme.colors[variant] || theme.colors.primary;

    return `
      background-color: ${colors.main};
      color: white;

      &:hover {
        background-color: ${colors.dark};
      }

      &:focus {
        outline: none;
        box-shadow: 0 0 0 3px ${colors.light}40;
      }
    `;
  }}

  /* Size variations using theme spacing */
  ${({ size = "medium", theme }) => {
    const sizes = {
      small: {
        padding: `${theme.spacing.xs} ${theme.spacing.sm}`,
        fontSize: theme.typography.fontSize.sm,
      },
      medium: {
        padding: `${theme.spacing.sm} ${theme.spacing.md}`,
        fontSize: theme.typography.fontSize.md,
      },
      large: {
        padding: `${theme.spacing.md} ${theme.spacing.lg}`,
        fontSize: theme.typography.fontSize.lg,
      },
    };

    return sizes[size];
  }}
`;
```

### Building a Dark Theme System

Let's implement a complete light/dark theme system:

```javascript
// Define base colors that both themes will use
const baseColors = {
  success: "#10b981",
  error: "#ef4444",
  warning: "#f59e0b",
};

// Create separate light and dark themes
const lightTheme = {
  name: "light",
  colors: {
    ...baseColors,
    background: {
      primary: "#ffffff",
      secondary: "#f3f4f6",
      tertiary: "#e5e7eb",
    },
    text: {
      primary: "#1f2937",
      secondary: "#4b5563",
      tertiary: "#6b7280",
    },
    border: {
      light: "#e5e7eb",
      medium: "#d1d5db",
      heavy: "#9ca3af",
    },
  },
};

const darkTheme = {
  name: "dark",
  colors: {
    ...baseColors,
    background: {
      primary: "#1f2937",
      secondary: "#111827",
      tertiary: "#374151",
    },
    text: {
      primary: "#f9fafb",
      secondary: "#e5e7eb",
      tertiary: "#d1d5db",
    },
    border: {
      light: "#374151",
      medium: "#4b5563",
      heavy: "#6b7280",
    },
  },
};

// Create a theme context and hook
export const ThemeContext = React.createContext({
  themeName: "light",
  toggleTheme: () => {},
});

export const ThemeWrapper = ({ children }) => {
  const [themeName, setThemeName] = useState("light");
  const theme = useMemo(
    () => (themeName === "light" ? lightTheme : darkTheme),
    [themeName]
  );

  const toggleTheme = () => {
    setThemeName((prev) => (prev === "light" ? "dark" : "light"));
  };

  return (
    <ThemeContext.Provider value={{ themeName, toggleTheme }}>
      <ThemeProvider theme={theme}>
        <GlobalStyles />
        {children}
      </ThemeProvider>
    </ThemeContext.Provider>
  );
};

// Global styles that react to theme changes
const GlobalStyles = createGlobalStyle`
  body {
    background-color: ${({ theme }) => theme.colors.background.primary};
    color: ${({ theme }) => theme.colors.text.primary};
    transition: all 0.2s ease-in-out;
  }
`;
```

### Creating Theme-Aware Utility Functions

To make working with theme values easier, we can create utility functions:

```javascript
// Theme utility functions
export const themeUtils = {
  // Get responsive value based on breakpoint
  responsive:
    (property, values) =>
    ({ theme }) => {
      const breakpoints = Object.keys(theme.breakpoints);
      return breakpoints.reduce((acc, breakpoint, index) => {
        if (values[index] !== undefined) {
          acc += `
          @media (min-width: ${theme.breakpoints[breakpoint]}) {
            ${property}: ${values[index]};
          }
        `;
        }
        return acc;
      }, `${property}: ${values[0]};`);
    },

  // Get color with optional opacity
  color:
    (path, opacity) =>
    ({ theme }) => {
      const color = path
        .split(".")
        .reduce((obj, key) => obj[key], theme.colors);
      return opacity !== undefined
        ? `${color}${Math.floor(opacity * 255).toString(16)}`
        : color;
    },

  // Get spacing value or combination
  spacing:
    (...args) =>
    ({ theme }) => {
      return args.map((arg) => theme.spacing[arg] || arg).join(" ");
    },
};

// Using the utilities in components
const Card = styled.div`
  ${themeUtils.responsive("padding", ["sm", "md", "lg"])}
  background-color: ${themeUtils.color("background.primary")};
  border: 1px solid ${themeUtils.color("border.light")};
  box-shadow: ${({ theme }) => theme.shadows.sm};

  &:hover {
    box-shadow: ${({ theme }) => theme.shadows.md};
  }
`;

const Text = styled.p`
  ${themeUtils.responsive("font-size", ["sm", "md", "lg"])}
  color: ${themeUtils.color("text.primary", 0.9)};
  margin: ${themeUtils.spacing("md", "lg")};
`;
```

### Theme Type Safety with TypeScript

For TypeScript users, here's how to add type safety to your theme:

```typescript
// Define theme type
type Theme = {
  colors: {
    primary: {
      main: string;
      light: string;
      dark: string;
    };
    // ... other color definitions
  };
  spacing: Record<"xs" | "sm" | "md" | "lg" | "xl", string>;
  typography: {
    fontFamily: Record<"sans" | "serif" | "mono", string>;
    fontSize: Record<"xs" | "sm" | "md" | "lg" | "xl" | "2xl", string>;
    fontWeight: Record<"normal" | "medium" | "semibold" | "bold", number>;
  };
  breakpoints: Record<"sm" | "md" | "lg" | "xl", string>;
  shadows: Record<"sm" | "md" | "lg", string>;
};

// Extend styled-components' DefaultTheme
declare module "styled-components" {
  export interface DefaultTheme extends Theme {}
}

// Use in your components with full type safety
const Button = styled.button<{ variant?: "primary" | "secondary" }>`
  background-color: ${({ theme, variant = "primary" }) =>
    theme.colors[variant].main};
`;
```

Would you like me to explain any of these concepts in more detail?
