# Understanding Relationships in MongoDB Schema Design

Unlike traditional SQL databases with rigid relationships, MongoDB offers flexible ways to model relationships between data. Let's explore how to design schemas with relationships that are both efficient and maintainable.

### Understanding Data Relationship Patterns

In MongoDB, we have three main ways to model relationships between data:

1. Embedded Documents (Denormalization)
2. References (Normalization)
3. Hybrid Approaches

Let's explore each approach with practical examples.

### Embedded Documents Pattern

Embedding is ideal when you have "belongs to" relationships where the child data is always accessed with the parent. Here's an example:

```javascript
// src/models/order.model.js
import mongoose from "mongoose";

// First, define schemas for embedded documents
const AddressSchema = new mongoose.Schema({
  street: String,
  city: String,
  state: String,
  zipCode: String,
  country: String,
  isDefault: Boolean,
});

const OrderItemSchema = new mongoose.Schema({
  product: {
    name: String,
    price: Number,
    sku: String,
  },
  quantity: Number,
  price: Number,
});

// Main Order schema with embedded documents
const OrderSchema = new mongoose.Schema(
  {
    orderNumber: {
      type: String,
      required: true,
      unique: true,
    },
    customer: {
      name: String,
      email: String,
      // Embed the shipping address
      shippingAddress: AddressSchema,
      // Embed the billing address
      billingAddress: AddressSchema,
    },
    // Embed the order items
    items: [OrderItemSchema],
    total: Number,
    status: {
      type: String,
      enum: ["pending", "processing", "shipped", "delivered"],
      default: "pending",
    },
  },
  {
    timestamps: true,
  }
);

// Add methods to handle embedded documents
OrderSchema.methods.addItem = function (item) {
  this.items.push(item);
  this.total = this.items.reduce(
    (sum, item) => sum + item.price * item.quantity,
    0
  );
};

const Order = mongoose.model("Order", OrderSchema);
export default Order;
```

Using embedded documents makes sense here because:

- Order items are always accessed with the order
- The data forms a clear unit
- The embedded data is relatively stable (won't change often)

### Referenced Documents Pattern

When data needs to be accessed independently or updated frequently, use references:

```javascript
// src/models/user.model.js
const UserSchema = new mongoose.Schema({
  email: {
    type: String,
    required: true,
    unique: true,
  },
  name: String,
  // Reference to orders
  orders: [
    {
      type: mongoose.Schema.Types.ObjectId,
      ref: "Order",
    },
  ],
  // Reference to preferred addresses
  addresses: [
    {
      type: mongoose.Schema.Types.ObjectId,
      ref: "Address",
    },
  ],
});

// Add population helpers
UserSchema.methods.getOrderHistory = async function () {
  // Populate orders with specific fields
  await this.populate({
    path: "orders",
    select: "orderNumber total status createdAt",
    options: { sort: { createdAt: -1 } },
  });
  return this.orders;
};

// src/models/product.model.js
const ProductSchema = new mongoose.Schema({
  name: String,
  price: Number,
  sku: String,
  // Reference to category
  category: {
    type: mongoose.Schema.Types.ObjectId,
    ref: "Category",
    required: true,
  },
  // Reference to reviews
  reviews: [
    {
      type: mongoose.Schema.Types.ObjectId,
      ref: "Review",
    },
  ],
});

// Add methods to handle references
ProductSchema.methods.getAverageRating = async function () {
  await this.populate("reviews");
  if (!this.reviews.length) return null;

  const sum = this.reviews.reduce((acc, review) => acc + review.rating, 0);
  return sum / this.reviews.length;
};
```

### Hybrid Approach: Combining References and Embedding

Sometimes the best solution is a combination of both approaches:

```javascript
// src/models/blog.model.js
const CommentSchema = new mongoose.Schema({
  content: String,
  // Reference the user but embed essential info
  author: {
    _id: {
      type: mongoose.Schema.Types.ObjectId,
      ref: "User",
    },
    name: String,
    avatar: String,
  },
  createdAt: {
    type: Date,
    default: Date.now,
  },
});

const PostSchema = new mongoose.Schema({
  title: String,
  content: String,
  // Reference the author but embed essential info
  author: {
    _id: {
      type: mongoose.Schema.Types.ObjectId,
      ref: "User",
    },
    name: String,
    avatar: String,
  },
  // Embed recent comments, reference the rest
  comments: {
    recent: [CommentSchema],
    count: { type: Number, default: 0 },
    allComments: [
      {
        type: mongoose.Schema.Types.ObjectId,
        ref: "Comment",
      },
    ],
  },
  // Reference categories
  categories: [
    {
      type: mongoose.Schema.Types.ObjectId,
      ref: "Category",
    },
  ],
});

// Add methods to handle hybrid data
PostSchema.methods.addComment = async function (comment) {
  // Keep only recent comments embedded
  const MAX_RECENT_COMMENTS = 5;

  // Add to all comments
  this.comments.allComments.push(comment._id);
  this.comments.count += 1;

  // Manage recent comments
  this.comments.recent.unshift({
    content: comment.content,
    author: {
      _id: comment.author._id,
      name: comment.author.name,
      avatar: comment.author.avatar,
    },
  });

  // Keep only recent comments
  if (this.comments.recent.length > MAX_RECENT_COMMENTS) {
    this.comments.recent.pop();
  }

  await this.save();
};
```

### Managing Many-to-Many Relationships

Many-to-many relationships require special consideration:

```javascript
// src/models/course.model.js
const CourseSchema = new mongoose.Schema({
  title: String,
  description: String,
  // Reference to students enrolled
  students: [
    {
      type: mongoose.Schema.Types.ObjectId,
      ref: "User",
    },
  ],
  // Reference to instructors
  instructors: [
    {
      type: mongoose.Schema.Types.ObjectId,
      ref: "User",
    },
  ],
});

// src/models/user.model.js
const UserSchema = new mongoose.Schema({
  // ... other fields

  // Reference courses based on role
  enrolledCourses: [
    {
      type: mongoose.Schema.Types.ObjectId,
      ref: "Course",
    },
  ],
  teachingCourses: [
    {
      type: mongoose.Schema.Types.ObjectId,
      ref: "Course",
    },
  ],
});

// Add methods to manage enrollment
CourseSchema.methods.enrollStudent = async function (studentId) {
  // Add student to course
  if (!this.students.includes(studentId)) {
    this.students.push(studentId);
  }

  // Add course to student's enrolled courses
  await mongoose
    .model("User")
    .findByIdAndUpdate(studentId, { $addToSet: { enrolledCourses: this._id } });

  await this.save();
};

CourseSchema.methods.unenrollStudent = async function (studentId) {
  // Remove student from course
  this.students = this.students.filter((id) => !id.equals(studentId));

  // Remove course from student's enrolled courses
  await mongoose
    .model("User")
    .findByIdAndUpdate(studentId, { $pull: { enrolledCourses: this._id } });

  await this.save();
};
```

### Handling Tree Structures

For hierarchical data like categories or comments, we can use a special pattern:

```javascript
// src/models/category.model.js
const CategorySchema = new mongoose.Schema({
  name: String,
  slug: String,
  parent: {
    type: mongoose.Schema.Types.ObjectId,
    ref: "Category",
    default: null,
  },
  // Store the full path to root
  ancestors: [
    {
      type: mongoose.Schema.Types.ObjectId,
      ref: "Category",
    },
  ],
  // Optional: Store level for easy querying
  level: {
    type: Number,
    default: 0,
  },
});

// Add methods to handle tree operations
CategorySchema.methods.addChild = async function (childData) {
  // Create child category
  const child = new mongoose.model("Category")({
    ...childData,
    parent: this._id,
    ancestors: [...this.ancestors, this._id],
    level: this.level + 1,
  });

  await child.save();
  return child;
};

CategorySchema.statics.getTree = async function () {
  const categories = await this.find({}).populate("parent").sort("level");

  const tree = {};
  categories.forEach((cat) => {
    if (!cat.parent) {
      tree[cat._id] = { ...cat.toObject(), children: {} };
    } else {
      let parent = tree;
      cat.ancestors.forEach((ancestorId) => {
        parent = parent[ancestorId].children;
      });
      parent[cat._id] = { ...cat.toObject(), children: {} };
    }
  });

  return tree;
};
```

This schema design approach provides:

- Clear separation of concerns
- Efficient querying capabilities
- Scalable data relationships
- Flexible data access patterns
- Maintainable codebase

Would you like me to explain any of these patterns in more detail?
