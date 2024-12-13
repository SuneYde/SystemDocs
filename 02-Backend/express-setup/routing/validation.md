# Request Validation in Express

Validation is a crucial security layer in any web application. It ensures that incoming data meets your application's requirements before it reaches your business logic. Let's explore how to create a robust validation system.

### Setting Up the Validation System

First, let's create a validation middleware that can handle different types of validation rules. We'll use Joi for validation, but this pattern can work with any validation library:

```javascript
// src/middleware/validate.js
import Joi from "joi";
import { ValidationError } from "../utils/errors";

export const validate = (schema) => {
  return (req, res, next) => {
    // Create an object containing all possible inputs
    const dataToValidate = {
      body: req.body,
      query: req.query,
      params: req.params,
    };

    // Validate the data against the schema
    const { error, value } = schema.validate(dataToValidate, {
      // Return all errors, not just the first one
      abortEarly: false,
      // Allow unknown fields to be stripped
      stripUnknown: true,
      // Convert types when possible (string '1' to number 1)
      convert: true,
    });

    if (error) {
      // Transform Joi errors into a more user-friendly format
      const validationErrors = error.details.map((detail) => ({
        field: detail.path.join("."),
        message: detail.message,
        type: detail.type,
      }));

      return next(new ValidationError("Validation failed", validationErrors));
    }

    // Replace request data with validated data
    req.body = value.body;
    req.query = value.query;
    req.params = value.params;

    next();
  };
};
```

### Creating Validation Schemas

Now let's create validation schemas for different parts of our application. We'll organize them by feature:

```javascript
// src/validations/user.validation.js
import Joi from "joi";

export const userValidation = {
  // Schema for creating a new user
  createUser: Joi.object({
    body: Joi.object({
      email: Joi.string().email().required().messages({
        "string.email": "Please provide a valid email address",
        "any.required": "Email is required",
      }),
      password: Joi.string()
        .min(8)
        .pattern(/^(?=.*[A-Za-z])(?=.*\d)/)
        .required()
        .messages({
          "string.min": "Password must be at least 8 characters long",
          "string.pattern.base":
            "Password must contain both letters and numbers",
        }),
      name: Joi.string().min(2).max(50).required().messages({
        "string.min": "Name must be at least 2 characters long",
        "string.max": "Name cannot exceed 50 characters",
      }),
    }),
  }),

  // Schema for updating a user
  updateUser: Joi.object({
    body: Joi.object({
      email: Joi.string().email(),
      name: Joi.string().min(2).max(50),
      // Don't allow password updates through this endpoint
      password: Joi.forbidden(),
    }).min(1), // Require at least one field to update
  }),

  // Schema for user search/filtering
  getUserList: Joi.object({
    query: Joi.object({
      search: Joi.string().max(100),
      role: Joi.string().valid("user", "admin", "moderator"),
      status: Joi.string().valid("active", "inactive", "pending"),
      sort: Joi.string().valid(
        "name",
        "email",
        "createdAt",
        "-name",
        "-email",
        "-createdAt"
      ),
      page: Joi.number().integer().min(1),
      limit: Joi.number().integer().min(1).max(100),
    }),
  }),
};
```

### Advanced Validation Patterns

Let's implement some advanced validation patterns for complex scenarios:

```javascript
// src/validations/order.validation.js
import Joi from "joi";

// Custom validation helpers
const customValidators = {
  isValidProductItems: (value, helpers) => {
    // Check for duplicate product IDs
    const productIds = value.map((item) => item.productId);
    const uniqueIds = new Set(productIds);

    if (uniqueIds.size !== productIds.length) {
      return helpers.error("array.unique", {
        message: "Duplicate products are not allowed",
      });
    }

    // Validate total items don't exceed limit
    const totalItems = value.reduce((sum, item) => sum + item.quantity, 0);
    if (totalItems > 100) {
      return helpers.error("array.max", {
        message: "Total order quantity cannot exceed 100 items",
      });
    }

    return value;
  },
};

export const orderValidation = {
  createOrder: Joi.object({
    body: Joi.object({
      items: Joi.array()
        .items(
          Joi.object({
            productId: Joi.string()
              .pattern(/^[0-9a-fA-F]{24}$/) // MongoDB ObjectId pattern
              .required(),
            quantity: Joi.number().integer().min(1).max(10).required(),
          })
        )
        .min(1)
        .custom(customValidators.isValidProductItems)
        .required(),

      shippingAddress: Joi.object({
        street: Joi.string().required(),
        city: Joi.string().required(),
        state: Joi.string().required(),
        zipCode: Joi.string()
          .pattern(/^\d{5}(-\d{4})?$/)
          .required(),
        country: Joi.string().length(2).uppercase().required(),
      }).required(),

      specialInstructions: Joi.string().max(500).allow("", null),
    }),
  }),
};
```

### Conditional Validation

Sometimes validation rules depend on other fields. Here's how to handle that:

```javascript
// src/validations/payment.validation.js
export const paymentValidation = {
  processPayment: Joi.object({
    body: Joi.object({
      amount: Joi.number().positive().precision(2).required(),

      paymentMethod: Joi.string()
        .valid("credit_card", "bank_transfer", "paypal")
        .required(),

      // Conditional validation based on payment method
      creditCard: Joi.when("paymentMethod", {
        is: "credit_card",
        then: Joi.object({
          number: Joi.string().creditCard().required(),
          expiryMonth: Joi.number().integer().min(1).max(12).required(),
          expiryYear: Joi.number()
            .integer()
            .min(new Date().getFullYear())
            .max(new Date().getFullYear() + 10)
            .required(),
          cvv: Joi.string()
            .pattern(/^\d{3,4}$/)
            .required(),
        }).required(),
        otherwise: Joi.forbidden(),
      }),

      bankAccount: Joi.when("paymentMethod", {
        is: "bank_transfer",
        then: Joi.object({
          accountNumber: Joi.string()
            .pattern(/^\d{10,12}$/)
            .required(),
          routingNumber: Joi.string()
            .pattern(/^\d{9}$/)
            .required(),
        }).required(),
        otherwise: Joi.forbidden(),
      }),

      paypalEmail: Joi.when("paymentMethod", {
        is: "paypal",
        then: Joi.string().email().required(),
        otherwise: Joi.forbidden(),
      }),
    }),
  }),
};
```

### Type-Safe Validation with TypeScript

For TypeScript users, let's add type safety to our validation system:

```typescript
// src/types/validation.ts
import { Request, Response, NextFunction } from "express";
import { ObjectSchema } from "joi";

export interface ValidationSchema extends ObjectSchema {
  body?: ObjectSchema;
  query?: ObjectSchema;
  params?: ObjectSchema;
}

export interface ValidateOptions {
  stripUnknown?: boolean;
  abortEarly?: boolean;
}

// Enhanced validate middleware with types
export const validate = (
  schema: ValidationSchema,
  options: ValidateOptions = {}
) => {
  return (req: Request, res: Response, next: NextFunction): void => {
    const validationOptions = {
      abortEarly: false,
      stripUnknown: true,
      ...options,
    };

    const { error, value } = schema.validate(
      {
        body: req.body,
        query: req.query,
        params: req.params,
      },
      validationOptions
    );

    if (error) {
      const validationErrors = error.details.map((detail) => ({
        field: detail.path.join("."),
        message: detail.message,
        type: detail.type,
      }));

      return next(new ValidationError("Validation failed", validationErrors));
    }

    // Type assertion for validated data
    req.body = value.body;
    req.query = value.query as Record<string, string>;
    req.params = value.params;

    next();
  };
};
```

### Using Validation in Routes

Finally, let's see how to use our validation system in routes:

```javascript
// src/routes/user.routes.js
import express from "express";
import { validate } from "../middleware/validate";
import { userValidation } from "../validations/user.validation";
import userController from "../controllers/user.controller";

const router = express.Router();

router.post(
  "/",
  validate(userValidation.createUser),
  userController.createUser
);

router.patch(
  "/:id",
  validate(userValidation.updateUser),
  userController.updateUser
);

router.get("/", validate(userValidation.getUserList), userController.getUsers);

export default router;
```

This validation system provides:

- Consistent error handling
- Type safety with TypeScript
- Complex validation rules
- Conditional validation
- Custom validation functions
- Clear, readable schemas
- Reusable validation components

Would you like me to explain any part in more detail?
