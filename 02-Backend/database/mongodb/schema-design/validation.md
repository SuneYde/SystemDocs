# MongoDB Schema Validation Guide

Data validation is crucial for maintaining data integrity in your MongoDB database. While MongoDB is schema-flexible, adding validation rules helps ensure your data remains consistent and reliable. Let's explore how to implement effective validation at both the schema and application levels.

### Basic Schema Validation

Let's start with a simple example that demonstrates the fundamental concepts of MongoDB validation:

```javascript
// src/models/user.model.js
import mongoose from "mongoose";

const UserSchema = new mongoose.Schema({
  email: {
    type: String,
    required: [true, "Email is required"],
    unique: true,
    // Custom validator for email format
    validate: {
      validator: function (email) {
        return /^[\w-]+(\.[\w-]+)*@([\w-]+\.)+[a-zA-Z]{2,7}$/.test(email);
      },
      message: (props) => `${props.value} is not a valid email address`,
    },
    // Ensure consistent format
    lowercase: true,
    trim: true,
  },

  password: {
    type: String,
    required: [true, "Password is required"],
    minlength: [8, "Password must be at least 8 characters long"],
    // Don't include password in query results by default
    select: false,
    validate: {
      validator: function (password) {
        // Require at least one number and one letter
        return /^(?=.*[A-Za-z])(?=.*\d)[A-Za-z\d]{8,}$/.test(password);
      },
      message: "Password must contain at least one letter and one number",
    },
  },

  age: {
    type: Number,
    min: [0, "Age cannot be negative"],
    max: [150, "Age cannot exceed 150"],
    // Optional field with custom validation
    validate: {
      validator: function (age) {
        // If age is provided, it must be a whole number
        return age === undefined || Number.isInteger(age);
      },
      message: "Age must be a whole number",
    },
  },

  status: {
    type: String,
    enum: {
      values: ["active", "inactive", "pending"],
      message: "{VALUE} is not a valid status",
    },
    default: "pending",
  },
});
```

### Advanced Validation Patterns

Now let's explore more complex validation scenarios:

```javascript
// src/models/order.model.js
const OrderSchema = new mongoose.Schema({
  customer: {
    type: mongoose.Schema.Types.ObjectId,
    ref: "User",
    required: true,
  },

  items: {
    type: [
      {
        product: {
          type: mongoose.Schema.Types.ObjectId,
          ref: "Product",
          required: true,
        },
        quantity: {
          type: Number,
          required: true,
          min: [1, "Quantity must be at least 1"],
        },
        price: {
          type: Number,
          required: true,
          min: [0, "Price cannot be negative"],
        },
      },
    ],
    validate: {
      // Ensure order has at least one item
      validator: function (items) {
        return items.length > 0;
      },
      message: "Order must contain at least one item",
    },
  },

  total: {
    type: Number,
    required: true,
    // Validate total matches sum of item prices
    validate: {
      validator: function (total) {
        const calculatedTotal = this.items.reduce(
          (sum, item) => sum + item.price * item.quantity,
          0
        );
        return Math.abs(total - calculatedTotal) < 0.01; // Account for floating point
      },
      message: "Total price does not match sum of items",
    },
  },

  shippingAddress: {
    street: {
      type: String,
      required: [true, "Street address is required"],
    },
    city: {
      type: String,
      required: [true, "City is required"],
    },
    zipCode: {
      type: String,
      required: [true, "ZIP code is required"],
      validate: {
        validator: function (zip) {
          return /^\d{5}(-\d{4})?$/.test(zip);
        },
        message: "Invalid ZIP code format",
      },
    },
  },

  // Conditional validation example
  couponCode: {
    type: String,
    validate: {
      validator: async function (code) {
        if (!code) return true; // Optional field

        // Check if coupon exists and is valid
        const coupon = await mongoose.model("Coupon").findOne({ code });
        if (!coupon) return false;

        // Check if coupon is expired
        if (coupon.expiresAt < new Date()) return false;

        // Check if coupon is valid for this order total
        if (coupon.minimumPurchase > this.total) return false;

        return true;
      },
      message: "Invalid or expired coupon code",
    },
  },
});

// Add pre-save hooks for additional validation
OrderSchema.pre("save", async function (next) {
  // Check if products are in stock
  for (const item of this.items) {
    const product = await mongoose.model("Product").findById(item.product);
    if (!product) {
      throw new Error(`Product ${item.product} not found`);
    }
    if (product.stock < item.quantity) {
      throw new Error(`Insufficient stock for product ${product.name}`);
    }
  }
  next();
});
```

### Custom Validation Rules

For complex business rules that span multiple fields or require async operations:

```javascript
// src/validators/order.validator.js
export class OrderValidator {
  static async validateOrder(order) {
    const errors = [];

    // Check business hours
    if (!this.isWithinBusinessHours()) {
      errors.push("Orders can only be placed during business hours");
    }

    // Check customer order limit
    const customerOrderCount = await mongoose.model("Order").countDocuments({
      customer: order.customer,
      createdAt: { $gt: new Date(Date.now() - 24 * 60 * 60 * 1000) },
    });

    if (customerOrderCount >= 5) {
      errors.push("Maximum 5 orders per customer per day");
    }

    // Check delivery area
    const validZipCodes = await mongoose.model("DeliveryArea").find({
      zipCode: order.shippingAddress.zipCode,
    });

    if (validZipCodes.length === 0) {
      errors.push("Delivery not available in this area");
    }

    // Check item combinations
    const invalidCombos = this.checkInvalidCombinations(order.items);
    if (invalidCombos.length > 0) {
      errors.push(`Invalid item combinations: ${invalidCombos.join(", ")}`);
    }

    return {
      isValid: errors.length === 0,
      errors,
    };
  }

  static isWithinBusinessHours() {
    const now = new Date();
    const hour = now.getHours();
    const day = now.getDay();

    // Check if weekend
    if (day === 0 || day === 6) return false;

    // Check if within 9 AM to 5 PM
    return hour >= 9 && hour < 17;
  }

  static checkInvalidCombinations(items) {
    // Example business rule: Can't order competing products together
    const incompatiblePairs = [
      ["PROD1", "PROD2"],
      ["PROD3", "PROD4"],
    ];

    const productIds = items.map((item) => item.product.toString());

    return incompatiblePairs.filter(
      (pair) => productIds.includes(pair[0]) && productIds.includes(pair[1])
    );
  }
}

// Using the validator in the schema
OrderSchema.pre("save", async function (next) {
  const validation = await OrderValidator.validateOrder(this);

  if (!validation.isValid) {
    throw new Error(validation.errors.join(". "));
  }

  next();
});
```

### Error Handling and Messages

To provide clear feedback when validation fails:

```javascript
// src/utils/validation-error.js
export class ValidationError extends Error {
  constructor(errors) {
    super("Validation failed");
    this.name = "ValidationError";
    this.errors = this.formatErrors(errors);
  }

  formatErrors(errors) {
    if (errors.errors) {
      // Mongoose validation errors
      return Object.values(errors.errors).map((err) => ({
        field: err.path,
        message: err.message,
        value: err.value,
      }));
    }

    // Custom validation errors
    if (Array.isArray(errors)) {
      return errors.map((error) => ({
        message: error,
      }));
    }

    return [{ message: errors.message || "Validation failed" }];
  }
}

// Using the error handler
OrderSchema.pre("save", async function (next) {
  try {
    const validation = await OrderValidator.validateOrder(this);

    if (!validation.isValid) {
      throw new ValidationError(validation.errors);
    }

    next();
  } catch (error) {
    next(error);
  }
});
```

This validation system provides:

- Field-level validation
- Complex business rule validation
- Async validation support
- Clear error messages
- Custom validation rules
- Conditional validation
- Cross-field validation

Would you like me to explain any of these concepts in more detail?
