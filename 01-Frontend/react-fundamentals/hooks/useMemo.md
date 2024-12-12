# useMemo.md

Hey there! Let's explore React's useMemo hook - your tool for optimizing expensive calculations and preventing unnecessary re-computations. Think of useMemo as a way to "save" the result of a complex calculation and reuse it unless the inputs change.

### Understanding useMemo: The Basics

useMemo comes into play when you have computationally expensive operations that you don't want to repeat on every render. Here's a simple example:

```javascript
import React, { useMemo, useState } from "react";

const ExpensiveCalculation = ({ numbers }) => {
  // Memoize the result of the expensive calculation
  const sum = useMemo(() => {
    console.log("Calculating sum..."); // See when it runs
    return numbers.reduce((acc, num) => acc + num, 0);
  }, [numbers]); // Only recalculate if numbers change

  return <div>Sum: {sum}</div>;
};
```

### When to Use useMemo

Let's look at some common scenarios where useMemo is useful:

#### 1. Expensive Calculations

```javascript
const DataAnalysis = ({ data }) => {
  const [filter, setFilter] = useState("");

  // Memoize expensive data processing
  const processedData = useMemo(() => {
    console.log("Processing data...");

    return data
      .filter((item) => item.value > 1000)
      .map((item) => ({
        ...item,
        normalized: item.value / 1000,
        category: determineCategory(item),
      }))
      .sort((a, b) => b.normalized - a.normalized);
  }, [data]); // Only recompute when data changes

  // Memoize filtered results
  const filteredResults = useMemo(() => {
    console.log("Filtering results...");

    return processedData.filter((item) =>
      item.name.toLowerCase().includes(filter.toLowerCase())
    );
  }, [processedData, filter]);

  return (
    <div>
      <input
        value={filter}
        onChange={(e) => setFilter(e.target.value)}
        placeholder="Filter results..."
      />
      <DataTable data={filteredResults} />
    </div>
  );
};
```

#### 2. Reference Equality

When passing objects or arrays to child components:

```javascript
const ParentComponent = ({ items }) => {
  const [sort, setSort] = useState("asc");

  // Memoize config object to maintain reference equality
  const tableConfig = useMemo(
    () => ({
      sortDirection: sort,
      columns: [
        { key: "name", label: "Name" },
        { key: "age", label: "Age" },
      ],
      rowHeight: 40,
      headerHeight: 50,
    }),
    [sort]
  ); // Only recreate when sort changes

  return <DataTable data={items} config={tableConfig} />;
};
```

### Advanced Patterns

#### 1. Memoizing Complex Calculations with Multiple Dependencies

```javascript
const DataVisualization = ({ data, filters, view }) => {
  // Multiple-step data processing
  const processedData = useMemo(() => {
    // Step 1: Filter data
    const filtered = data.filter((item) =>
      filters.every((filter) => filter(item))
    );

    // Step 2: Transform data
    const transformed = filtered.map((item) => ({
      ...item,
      value: calculateValue(item),
      color: determineColor(item),
    }));

    // Step 3: Group data
    const grouped = transformed.reduce((acc, item) => {
      const key = item.category;
      return {
        ...acc,
        [key]: [...(acc[key] || []), item],
      };
    }, {});

    // Step 4: Final calculations
    return Object.entries(grouped).map(([key, items]) => ({
      category: key,
      total: items.reduce((sum, item) => sum + item.value, 0),
      items: items.sort((a, b) => b.value - a.value),
    }));
  }, [data, filters]); // Recalculate when data or filters change

  return <VisualizationComponent data={processedData} view={view} />;
};
```

#### 2. Conditional Memoization

```javascript
const SmartList = ({ items, isVirtualized }) => {
  // Only apply expensive virtualization calculations if needed
  const listProps = useMemo(() => {
    if (!isVirtualized) return { items };

    return {
      items,
      rowHeight: 50,
      overscanCount: 3,
      visibleRows: Math.ceil(window.innerHeight / 50),
      totalHeight: items.length * 50,
      getItemOffset: (index) => index * 50,
    };
  }, [items, isVirtualized]);

  return isVirtualized ? (
    <VirtualList {...listProps} />
  ) : (
    <SimpleList {...listProps} />
  );
};
```

#### 3. Memoizing Selector Functions

```javascript
const UserDashboard = ({ users, departments }) => {
  const [selectedDept, setSelectedDept] = useState(null);

  // Memoize complex data selectors
  const departmentStats = useMemo(() => {
    return departments.map((dept) => {
      const deptUsers = users.filter((u) => u.deptId === dept.id);
      return {
        ...dept,
        userCount: deptUsers.length,
        averageSalary:
          deptUsers.reduce((sum, u) => sum + u.salary, 0) / deptUsers.length,
        topPerformers: deptUsers
          .filter((u) => u.performance > 90)
          .sort((a, b) => b.performance - a.performance),
      };
    });
  }, [users, departments]);

  // Memoize selected department details
  const selectedDepartmentDetails = useMemo(() => {
    if (!selectedDept) return null;

    return {
      ...departmentStats.find((d) => d.id === selectedDept),
      users: users
        .filter((u) => u.deptId === selectedDept)
        .map((u) => ({
          ...u,
          performanceRank: calculateRank(u.performance),
        })),
    };
  }, [selectedDept, departmentStats, users]);

  return (
    <div>
      <DepartmentList
        departments={departmentStats}
        onSelect={setSelectedDept}
      />
      {selectedDepartmentDetails && (
        <DepartmentDetails data={selectedDepartmentDetails} />
      )}
    </div>
  );
};
```

### Best Practices and Optimization Tips

1. **Don't Over-Optimize**

```javascript
// ❌ Don't memoize simple calculations
const SimpleComponent = ({ name }) => {
  // Unnecessary memoization
  const upperName = useMemo(() => name.toUpperCase(), [name]);

  // ✅ Do this instead
  const upperName = name.toUpperCase();
};
```

2. **Be Careful with Dependencies**

```javascript
// ❌ Missing dependencies
const BadExample = ({ data, threshold }) => {
  const filtered = useMemo(
    () => data.filter((item) => item.value > threshold),
    [data] // Missing threshold dependency
  );
};

// ✅ Include all dependencies
const GoodExample = ({ data, threshold }) => {
  const filtered = useMemo(
    () => data.filter((item) => item.value > threshold),
    [data, threshold]
  );
};
```

3. **Combine with useCallback**

```javascript
const TableComponent = ({ data }) => {
  const sortedData = useMemo(
    () => [...data].sort((a, b) => b.value - a.value),
    [data]
  );

  const handleRowClick = useCallback((id) => {
    console.log("Clicked row:", id);
  }, []);

  return <Table data={sortedData} onRowClick={handleRowClick} />;
};
```

Want to explore any of these patterns in more detail or see more specific examples? Let me know!
