# Understanding MongoDB Indexes

Think of indexes in MongoDB like the index at the back of a book - they help you find information quickly without having to scan through every page. Without proper indexing, MongoDB has to perform collection scans, examining every document to find what you're looking for. Let's explore how to implement indexes effectively to optimize your database performance.

### Basic Index Implementation

Let's start with the fundamental index types and their implementation:

```javascript
// src/models/user.model.js
import mongoose from "mongoose";

const UserSchema = new mongoose.Schema({
  email: {
    type: String,
    required: true,
  },
  username: {
    type: String,
    required: true,
  },
  createdAt: {
    type: Date,
    default: Date.now,
  },
  lastLoginAt: Date,
  status: {
    type: String,
    enum: ["active", "inactive", "suspended"],
  },
});

// Single field index
// Why: Quick lookups by email address, commonly used for authentication
UserSchema.index(
  { email: 1 },
  {
    unique: true, // Ensure email uniqueness
    sparse: true, // Don't index documents without email field
    background: true, // Build index in background
  }
);

// Compound index for username and status
// Why: Optimize queries that filter by both username and status
UserSchema.index({
  username: 1,
  status: 1,
});

// Index with partial filter expression
// Why: Index only active users for better performance
UserSchema.index(
  { lastLoginAt: -1 },
  {
    partialFilterExpression: {
      status: "active",
    },
    expireAfterSeconds: 30 * 24 * 60 * 60, // TTL index: Remove after 30 days
  }
);

const User = mongoose.model("User", UserSchema);
```

### Advanced Indexing Strategies

Let's look at more sophisticated indexing patterns for complex queries:

```javascript
// src/models/product.model.js
const ProductSchema = new mongoose.Schema({
  name: String,
  description: String,
  price: Number,
  category: String,
  tags: [String],
  specifications: {
    color: String,
    size: String,
    weight: Number,
  },
  stock: Number,
  isAvailable: Boolean,
});

// Text index for search functionality
// Why: Enable full-text search across multiple fields
ProductSchema.index(
  {
    name: "text",
    description: "text",
    tags: "text",
  },
  {
    weights: {
      name: 10, // Name matches are most important
      description: 5, // Description matches less important
      tags: 2, // Tag matches least important
    },
    default_language: "english",
    language_override: "lang",
  }
);

// Compound index for category and price
// Why: Optimize category browsing with price sorting
ProductSchema.index({
  category: 1,
  price: -1,
});

// Compound index with partial filter
// Why: Optimize queries for available products within price ranges
ProductSchema.index(
  { price: 1 },
  {
    partialFilterExpression: {
      isAvailable: true,
      stock: { $gt: 0 },
    },
  }
);

// Index for geospatial queries
const StoreSchema = new mongoose.Schema({
  name: String,
  location: {
    type: {
      type: String,
      enum: ["Point"],
      required: true,
    },
    coordinates: {
      type: [Number],
      required: true,
    },
  },
});

// Create 2dsphere index for location-based queries
StoreSchema.index({ location: "2dsphere" });
```

### Index Management and Monitoring

Let's create utilities to manage and monitor index performance:

```javascript
// src/utils/index-manager.js
export class IndexManager {
  constructor(model) {
    this.model = model;
  }

  // Analyze index usage
  async analyzeIndexes() {
    const collection = this.model.collection;

    // Get current indexes
    const indexes = await collection.indexes();

    // Get index statistics
    const stats = await collection.aggregate([{ $indexStats: {} }]).toArray();

    // Analyze each index
    const analysis = indexes.map((index) => {
      const indexStats = stats.find((stat) => stat.name === index.name);

      return {
        name: index.name,
        fields: index.key,
        size: indexStats?.size || 0,
        operations: indexStats?.accesses?.ops || 0,
        since: indexStats?.accesses?.since,
        isUnique: index.unique || false,
        isSparse: index.sparse || false,
      };
    });

    return analysis;
  }

  // Find unused indexes
  async findUnusedIndexes(thresholdDays = 30) {
    const analysis = await this.analyzeIndexes();
    const thresholdDate = new Date();
    thresholdDate.setDate(thresholdDate.getDate() - thresholdDays);

    return analysis.filter((index) => {
      // Skip _id index
      if (index.name === "_id_") return false;

      // Check if index is unused
      return index.operations === 0 || new Date(index.since) < thresholdDate;
    });
  }

  // Create indexes with progress monitoring
  async createIndexes(indexSpecs) {
    const total = indexSpecs.length;
    let completed = 0;

    for (const spec of indexSpecs) {
      try {
        await this.model.collection.createIndex(spec.fields, spec.options);
        completed++;

        console.log(
          `Created index ${completed}/${total}: ${JSON.stringify(spec.fields)}`
        );
      } catch (error) {
        console.error(
          `Failed to create index: ${JSON.stringify(spec.fields)}`,
          error
        );
      }
    }
  }
}

// Usage example
const manager = new IndexManager(User);

// Analyze indexes
const analysis = await manager.analyzeIndexes();
console.log("Index Analysis:", analysis);

// Find unused indexes
const unusedIndexes = await manager.findUnusedIndexes();
console.log("Unused Indexes:", unusedIndexes);

// Create new indexes
const indexSpecs = [
  {
    fields: { email: 1 },
    options: { unique: true },
  },
  {
    fields: { username: 1, status: 1 },
    options: { background: true },
  },
];

await manager.createIndexes(indexSpecs);
```

### Best Practices and Performance Tips

1. Index Intersection

```javascript
// Instead of creating a compound index for every combination
UserSchema.index({ age: 1 });
UserSchema.index({ status: 1 });
// MongoDB can intersect these indexes for queries using both fields
// db.users.find({ age: { $gt: 21 }, status: 'active' })
```

2. Covered Queries

```javascript
// Create index to support covered queries
UserSchema.index({ username: 1, email: 1 });

// Query that can be satisfied entirely by the index
await User.find({ username: "john" }, { username: 1, email: 1, _id: 0 });
```

3. Index for Sort Operations

```javascript
// Optimize sort operations with proper indexes
PostSchema.index(
  { createdAt: -1, authorId: 1 },
  {
    // Name index for easier monitoring
    name: "post_author_date_idx",
  }
);

// This query will use the index for both filtering and sorting
await Post.find({ authorId: userId }).sort({ createdAt: -1 }).limit(10);
```

Remember:

- Indexes improve query performance but add overhead to writes
- Monitor index usage to remove unused indexes
- Use compound indexes wisely to support multiple query patterns
- Consider partial indexes to reduce index size
- Use text indexes for full-text search capabilities
- Ensure indexes fit in RAM for optimal performance

Would you like me to explain any of these concepts in more detail?
