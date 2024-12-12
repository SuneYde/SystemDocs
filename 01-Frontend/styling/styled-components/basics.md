# Styled Components: The Fundamentals

Welcome to the world of styled-components! This guide will help you understand how to write and use styled-components effectively in your React applications. Think of styled-components as a way to write CSS directly within your JavaScript code, but with all the power of dynamic values and component-based thinking.

### Understanding the Core Concept

Styled-components combines traditional CSS with JavaScript template literals, allowing you to write actual CSS code to style your components. Let's start with a simple example:

```javascript
import styled from "styled-components";

// Creating a styled button component
const Button = styled.button`
  /* This is regular CSS, but it's in your JavaScript! */
  padding: 8px 16px;
  background-color: #0070f3;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 16px;
  cursor: pointer;

  /* You can use pseudo-classes just like in regular CSS */
  &:hover {
    background-color: #0051a2;
  }

  /* And pseudo-elements too */
  &::after {
    content: " →";
  }
`;

// Using the styled component
function App() {
  return <Button>Click Me</Button>;
}
```

### Working with Props

One of the most powerful features of styled-components is the ability to use props to change styles dynamically. Here's how we can make our button more flexible:

```javascript
const Button = styled.button`
  /* Base styles remain the same */
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  font-size: 16px;
  cursor: pointer;

  /* Use props to determine the button's variant */
  background-color: ${(props) =>
    props.variant === "secondary" ? "#e2e8f0" : "#0070f3"};

  color: ${(props) => (props.variant === "secondary" ? "#1a202c" : "white")};

  /* Size variations based on props */
  padding: ${(props) => {
    switch (props.size) {
      case "small":
        return "4px 8px";
      case "large":
        return "12px 24px";
      default:
        return "8px 16px";
    }
  }};

  /* Disabled state */
  opacity: ${(props) => (props.disabled ? 0.5 : 1)};
  cursor: ${(props) => (props.disabled ? "not-allowed" : "pointer")};
`;

// Usage with props
function App() {
  return (
    <div>
      <Button>Default Button</Button>
      <Button variant="secondary" size="small">
        Small Secondary Button
      </Button>
      <Button size="large" disabled>
        Disabled Large Button
      </Button>
    </div>
  );
}
```

### Extending Styles

Sometimes you want to create a new component based on an existing one. Styled-components makes this easy:

```javascript
// Base button component
const BaseButton = styled.button`
  padding: 8px 16px;
  border-radius: 4px;
  font-size: 16px;
  cursor: pointer;
`;

// Extended primary button
const PrimaryButton = styled(BaseButton)`
  background-color: #0070f3;
  color: white;
  border: none;

  &:hover {
    background-color: #0051a2;
  }
`;

// Extended outline button
const OutlineButton = styled(BaseButton)`
  background-color: transparent;
  color: #0070f3;
  border: 2px solid #0070f3;

  &:hover {
    background-color: #0070f3;
    color: white;
  }
`;
```

### Working with Global Styles

While styled-components focuses on component-level styles, you sometimes need global styles. Here's how to handle them:

```javascript
import { createGlobalStyle } from "styled-components";

const GlobalStyle = createGlobalStyle`
  /* Reset default styles */
  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }
  
  /* Set base styles */
  body {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen,
      Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
    line-height: 1.5;
    color: #333;
  }
  
  /* Add any other global styles */
  a {
    color: #0070f3;
    text-decoration: none;
    
    &:hover {
      text-decoration: underline;
    }
  }
`;

// Use in your app
function App() {
  return (
    <>
      <GlobalStyle />
      {/* Rest of your app */}
    </>
  );
}
```

### Using Helper Functions

As your styles get more complex, it's helpful to create reusable style functions:

```javascript
// Helper functions for common styles
const flexCenter = `
  display: flex;
  justify-content: center;
  align-items: center;
`;

const textEllipsis = `
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
`;

// Using the helpers
const Card = styled.div`
  ${flexCenter}
  background: white;
  border-radius: 8px;
  padding: 24px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
`;

const CardTitle = styled.h2`
  ${textEllipsis}
  max-width: 200px;
  font-size: 18px;
  margin-bottom: 8px;
`;
```

### Using Media Queries

Styled-components makes responsive design straightforward:

```javascript
const Container = styled.div`
  width: 100%;
  padding: 0 16px;
  margin: 0 auto;

  /* Media queries work just like in regular CSS */
  @media (min-width: 640px) {
    max-width: 640px;
    padding: 0 24px;
  }

  @media (min-width: 768px) {
    max-width: 768px;
  }

  @media (min-width: 1024px) {
    max-width: 1024px;
  }
`;

// You can also create reusable media query helpers
const media = {
  sm: (styles) => `
    @media (min-width: 640px) {
      ${styles}
    }
  `,
  md: (styles) => `
    @media (min-width: 768px) {
      ${styles}
    }
  `,
  lg: (styles) => `
    @media (min-width: 1024px) {
      ${styles}
    }
  `,
};

// Using media helpers
const ResponsiveText = styled.p`
  font-size: 16px;

  ${media.sm(`
    font-size: 18px;
  `)}

  ${media.lg(`
    font-size: 20px;
  `)}
`;
```

Remember that styled-components integrates seamlessly with React's component model, allowing you to create reusable, maintainable style solutions that feel natural in a React application. Would you like me to explain any of these concepts in more detail?
