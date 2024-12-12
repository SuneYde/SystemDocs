# compound-components.md

Welcome to our deep dive into compound components! Think of compound components like a well-designed set of LEGO pieces - while each piece can work independently, they're designed to work together in specific ways to create something greater than the sum of their parts.

### Understanding Compound Components

Compound components are a pattern where several components work together to create a cohesive user interface with shared state and behavior. Imagine building a `Select` component - instead of passing all configuration as props to a single component, we create several specialized components that work together:

```javascript
import React, { createContext, useContext, useState } from "react";

// First, let's create our Select component system
const SelectContext = createContext();

const Select = ({ children, onValueChange }) => {
  const [selectedValue, setSelectedValue] = useState(null);
  const [isOpen, setIsOpen] = useState(false);

  const handleSelect = (value) => {
    setSelectedValue(value);
    setIsOpen(false);
    onValueChange?.(value);
  };

  return (
    <SelectContext.Provider
      value={{
        selectedValue,
        isOpen,
        setIsOpen,
        onSelect: handleSelect,
      }}
    >
      <div className="select-container">{children}</div>
    </SelectContext.Provider>
  );
};

// Create sub-components that work together
const Trigger = ({ children }) => {
  const { selectedValue, isOpen, setIsOpen } = useContext(SelectContext);

  return (
    <button
      className={`select-trigger ${isOpen ? "open" : ""}`}
      onClick={() => setIsOpen(!isOpen)}
    >
      {children(selectedValue)}
    </button>
  );
};

const Options = ({ children }) => {
  const { isOpen } = useContext(SelectContext);

  if (!isOpen) return null;

  return <div className="select-options">{children}</div>;
};

const Option = ({ value, children }) => {
  const { selectedValue, onSelect } = useContext(SelectContext);
  const isSelected = selectedValue === value;

  return (
    <div
      className={`select-option ${isSelected ? "selected" : ""}`}
      onClick={() => onSelect(value)}
    >
      {children}
    </div>
  );
};

// Attach sub-components to main component
Select.Trigger = Trigger;
Select.Options = Options;
Select.Option = Option;

// Now we can use our compound component like this:
const Example = () => {
  return (
    <Select onValueChange={(value) => console.log("Selected:", value)}>
      <Select.Trigger>
        {(value) => (
          <span>{value ? `Selected: ${value}` : "Choose an option"}</span>
        )}
      </Select.Trigger>

      <Select.Options>
        <Select.Option value="1">Option 1</Select.Option>
        <Select.Option value="2">Option 2</Select.Option>
        <Select.Option value="3">Option 3</Select.Option>
      </Select.Options>
    </Select>
  );
};
```

### Building a More Complex Example: Tabs Component

Let's build a more complex compound component system for tabs with animations and keyboard navigation:

```javascript
import React, { createContext, useContext, useState, useRef } from "react";

const TabsContext = createContext();

const Tabs = ({ children, defaultTab }) => {
  const [activeTab, setActiveTab] = useState(defaultTab);
  const tabsRef = useRef(new Map());

  const registerTab = (id, ref) => {
    tabsRef.current.set(id, ref);
  };

  const handleKeyNav = (event, currentId) => {
    const tabs = Array.from(tabsRef.current.keys());
    const currentIndex = tabs.indexOf(currentId);

    switch (event.key) {
      case "ArrowRight":
        const nextIndex = (currentIndex + 1) % tabs.length;
        setActiveTab(tabs[nextIndex]);
        tabsRef.current.get(tabs[nextIndex])?.focus();
        break;
      case "ArrowLeft":
        const prevIndex = (currentIndex - 1 + tabs.length) % tabs.length;
        setActiveTab(tabs[prevIndex]);
        tabsRef.current.get(tabs[prevIndex])?.focus();
        break;
    }
  };

  return (
    <TabsContext.Provider
      value={{
        activeTab,
        setActiveTab,
        registerTab,
        handleKeyNav,
      }}
    >
      <div className="tabs-container">{children}</div>
    </TabsContext.Provider>
  );
};

const TabList = ({ children }) => {
  return (
    <div className="tabs-list" role="tablist">
      {children}
    </div>
  );
};

const Tab = ({ id, children }) => {
  const { activeTab, setActiveTab, registerTab, handleKeyNav } =
    useContext(TabsContext);

  const ref = useRef();

  useEffect(() => {
    registerTab(id, ref.current);
    return () => registerTab(id, null);
  }, [id]);

  return (
    <button
      ref={ref}
      role="tab"
      aria-selected={activeTab === id}
      aria-controls={`panel-${id}`}
      id={`tab-${id}`}
      className={`tab ${activeTab === id ? "active" : ""}`}
      onClick={() => setActiveTab(id)}
      onKeyDown={(e) => handleKeyNav(e, id)}
      tabIndex={activeTab === id ? 0 : -1}
    >
      {children}
    </button>
  );
};

const TabPanels = ({ children }) => {
  return <div className="tab-panels">{children}</div>;
};

const TabPanel = ({ id, children }) => {
  const { activeTab } = useContext(TabsContext);
  const isActive = activeTab === id;

  return (
    <div
      role="tabpanel"
      id={`panel-${id}`}
      aria-labelledby={`tab-${id}`}
      hidden={!isActive}
      className={`tab-panel ${isActive ? "active" : ""}`}
    >
      {children}
    </div>
  );
};

// Attach all sub-components
Tabs.TabList = TabList;
Tabs.Tab = Tab;
Tabs.TabPanels = TabPanels;
Tabs.TabPanel = TabPanel;

// Usage example with animations and styling
const Example = () => {
  return (
    <Tabs defaultTab="1">
      <Tabs.TabList>
        <Tabs.Tab id="1">Profile</Tabs.Tab>
        <Tabs.Tab id="2">Settings</Tabs.Tab>
        <Tabs.Tab id="3">Notifications</Tabs.Tab>
      </Tabs.TabList>

      <Tabs.TabPanels>
        <Tabs.TabPanel id="1">
          <h2>Profile Content</h2>
          <p>User profile information goes here...</p>
        </Tabs.TabPanel>

        <Tabs.TabPanel id="2">
          <h2>Settings Content</h2>
          <p>User settings go here...</p>
        </Tabs.TabPanel>

        <Tabs.TabPanel id="3">
          <h2>Notifications Content</h2>
          <p>User notifications go here...</p>
        </Tabs.TabPanel>
      </Tabs.TabPanels>
    </Tabs>
  );
};
```

### Advanced Patterns and Best Practices

1. **Type Safety with TypeScript**

```typescript
interface TabsContextType {
  activeTab: string;
  setActiveTab: (id: string) => void;
  registerTab: (id: string, ref: HTMLElement | null) => void;
  handleKeyNav: (event: React.KeyboardEvent, currentId: string) => void;
}

interface TabsProps {
  children: React.ReactNode;
  defaultTab: string;
}

interface TabProps {
  id: string;
  children: React.ReactNode;
}

// Add type safety to context
const TabsContext = createContext<TabsContextType | undefined>(undefined);

// Type-safe hook
const useTabsContext = () => {
  const context = useContext(TabsContext);
  if (!context) {
    throw new Error("Tabs components must be used within Tabs provider");
  }
  return context;
};
```

2. **Validation and Error Handling**

```javascript
const Tabs = ({ children, defaultTab }) => {
  // Validate children structure
  const validateChildren = () => {
    const childArray = React.Children.toArray(children);
    const hasTabList = childArray.some((child) => child.type === TabList);
    const hasTabPanels = childArray.some((child) => child.type === TabPanels);

    if (!hasTabList || !hasTabPanels) {
      throw new Error(
        "Tabs must contain both TabList and TabPanels components"
      );
    }
  };

  useEffect(() => {
    validateChildren();
  }, [children]);

  // Rest of the implementation...
};
```

3. **Performance Optimization**

```javascript
const Tab = React.memo(({ id, children }) => {
  const { activeTab, setActiveTab } = useTabsContext();

  const handleClick = useCallback(() => {
    setActiveTab(id);
  }, [id, setActiveTab]);

  return (
    <button
      className={`tab ${activeTab === id ? "active" : ""}`}
      onClick={handleClick}
    >
      {children}
    </button>
  );
});
```

Want to explore any of these concepts further or see more specific examples? Let me know!
