# MongoDB Basic Operations Guide

In MongoDB, basic operations form the foundation of how we interact with our data. Understanding these operations thoroughly is crucial for building efficient and maintainable applications. Let's explore each type of operation and learn when and how to use them effectively.

### Reading Documents

Let's start with retrieving data, as this is the most common operation you'll perform:

```javascript
// src/services/user.service.js
import User from "../models/user.model";

class UserService {
  // Find a single document by ID
  async findById(userId) {
    try {
      // Using findById for direct _id lookups
      // This is the most efficient way to find a document by ID
      const user = await User.findById(userId)
        .select("name email profile") // Only fetch needed fields
        .lean(); // Convert to plain JavaScript object for better performance

      return user;
    } catch (error) {
      // Handle potential casting errors for invalid ObjectIds
      if (error.name === "CastError") {
        throw new Error("Invalid user ID format");
      }
      throw error;
    }
  }

  // Find documents with specific criteria
  async findUsers(criteria) {
    const {
      searchTerm,
      status,
      role,
      ageRange,
      page = 1,
      limit = 10,
    } = criteria;

    // Build query conditions
    const query = {};

    // Add search functionality
    if (searchTerm) {
      query.$or = [
        { name: new RegExp(searchTerm, "i") }, // Case-insensitive name search
        { email: new RegExp(searchTerm, "i") }, // Case-insensitive email search
      ];
    }

    // Add status filter if provided
    if (status) {
      query.status = status;
    }

    // Add role filter if provided
    if (role) {
      query.role = role;
    }

    // Add age range filter if provided
    if (ageRange) {
      query.age = {
        $gte: ageRange.min,
        $lte: ageRange.max,
      };
    }

    // Execute the query with pagination
    const users = await User.find(query)
      .sort({ createdAt: -1 }) // Sort by creation date, newest first
      .skip((page - 1) * limit) // Skip previous pages
      .limit(limit) // Limit results per page
      .select("-password"); // Exclude password field

    // Get total count for pagination
    const total = await User.countDocuments(query);

    return {
      users,
      pagination: {
        total,
        page,
        pages: Math.ceil(total / limit),
      },
    };
  }
}
```

### Creating Documents

Creating documents requires careful consideration of validation and error handling:

```javascript
// src/services/product.service.js
class ProductService {
  // Create a single document
  async createProduct(productData) {
    try {
      // Create the product
      const product = new Product(productData);

      // Validate before saving
      await product.validate();

      // Save to database
      await product.save();

      // If we need to perform additional operations after creation
      await this.handlePostCreation(product);

      return product;
    } catch (error) {
      if (error.code === 11000) {
        // Handle duplicate key errors
        const field = Object.keys(error.keyPattern)[0];
        throw new Error(`A product with this ${field} already exists`);
      }
      throw error;
    }
  }

  // Create multiple documents efficiently
  async createManyProducts(productsData) {
    try {
      // Validate all products before insertion
      const products = productsData.map((data) => new Product(data));
      await Promise.all(products.map((product) => product.validate()));

      // Use ordered: false for better performance when inserting many documents
      // This continues insertion even if some documents fail
      const result = await Product.insertMany(products, {
        ordered: false,
        // Return documents after insertion
        rawResult: true,
      });

      return {
        inserted: result.insertedCount,
        failed: productsData.length - result.insertedCount,
        documents: result.ops,
      };
    } catch (error) {
      if (Array.isArray(error.writeErrors)) {
        // Handle bulk write errors
        const failures = error.writeErrors.map((err) => ({
          index: err.index,
          error: err.errmsg,
        }));

        throw new Error("Some products failed to insert", { failures });
      }
      throw error;
    }
  }
}
```

### Updating Documents

Updates require careful consideration to maintain data integrity:

```javascript
// src/services/order.service.js
class OrderService {
  // Update a single document
  async updateOrder(orderId, updates) {
    // Use findOneAndUpdate to get the updated document
    const order = await Order.findOneAndUpdate(
      { _id: orderId },
      // Use $set to only update specified fields
      { $set: updates },
      {
        new: true, // Return updated document
        runValidators: true, // Run schema validators on update
        context: "query", // Provide context for custom validators
      }
    );

    if (!order) {
      throw new Error("Order not found");
    }

    return order;
  }

  // Update multiple documents
  async updateOrderStatus(status, criteria) {
    const result = await Order.updateMany(
      criteria,
      {
        $set: { status },
        $push: {
          statusHistory: {
            status,
            timestamp: new Date(),
            comment: criteria.comment,
          },
        },
      },
      { runValidators: true }
    );

    return {
      matched: result.matchedCount,
      modified: result.modifiedCount,
    };
  }

  // Atomic updates for critical operations
  async updateInventory(orderId) {
    const session = await mongoose.startSession();
    session.startTransaction();

    try {
      const order = await Order.findById(orderId).session(session);

      // Update inventory for each product
      for (const item of order.items) {
        const result = await Product.findOneAndUpdate(
          {
            _id: item.productId,
            stock: { $gte: item.quantity },
          },
          {
            $inc: { stock: -item.quantity },
          },
          {
            session,
            new: true,
          }
        );

        if (!result) {
          throw new Error(`Insufficient stock for product ${item.productId}`);
        }
      }

      // Commit transaction
      await session.commitTransaction();
    } catch (error) {
      // Rollback on error
      await session.abortTransaction();
      throw error;
    } finally {
      session.endSession();
    }
  }
}
```

### Deleting Documents

Deletion operations need careful handling to prevent accidental data loss:

```javascript
// src/services/post.service.js
class PostService {
  // Soft delete a single document
  async softDeletePost(postId, userId) {
    const post = await Post.findOneAndUpdate(
      {
        _id: postId,
        author: userId, // Ensure user owns the post
      },
      {
        $set: {
          deleted: true,
          deletedAt: new Date(),
          deletedBy: userId,
        },
      },
      { new: true }
    );

    if (!post) {
      throw new Error("Post not found or unauthorized");
    }

    return post;
  }

  // Hard delete with cascading
  async hardDeletePost(postId) {
    const session = await mongoose.startSession();
    session.startTransaction();

    try {
      // Delete the post
      const post = await Post.findByIdAndDelete(postId).session(session);

      if (!post) {
        throw new Error("Post not found");
      }

      // Delete related comments
      await Comment.deleteMany({ postId }).session(session);

      // Delete related likes
      await Like.deleteMany({ postId }).session(session);

      await session.commitTransaction();

      return {
        success: true,
        deletedPost: post,
      };
    } catch (error) {
      await session.abortTransaction();
      throw error;
    } finally {
      session.endSession();
    }
  }

  // Bulk delete with conditions
  async deleteInactivePosts(days) {
    const cutoffDate = new Date();
    cutoffDate.setDate(cutoffDate.getDate() - days);

    const result = await Post.deleteMany({
      lastActivity: { $lt: cutoffDate },
      status: "inactive",
    });

    return {
      deleted: result.deletedCount,
    };
  }
}
```

These operations provide the foundation for interacting with MongoDB while maintaining:

- Data integrity
- Error handling
- Transaction support
- Performance optimization
- Security considerations

Would you like me to explain any of these concepts in more detail?
