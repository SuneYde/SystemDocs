# Express Controllers: A Comprehensive Guide

A well-structured controller layer is essential for building maintainable Express applications. Controllers handle the business logic between your routes and data layer, managing how your application responds to client requests. Let's explore how to build robust controllers using a class-based approach.

### Foundation: The Base Controller

We'll start by creating a base controller that implements common functionality all controllers will need:

```javascript
// src/controllers/base.controller.js
import { AppError } from "../utils/errors";

class BaseController {
  constructor(service) {
    this.service = service;
    this.bindMethods();
  }

  // Automatically bind all methods and wrap with error handling
  bindMethods() {
    const methods = Object.getOwnPropertyNames(
      Object.getPrototypeOf(this)
    ).filter(
      (method) => method !== "constructor" && typeof this[method] === "function"
    );

    methods.forEach((method) => {
      this[method] = this.catchAsync(this[method].bind(this));
    });
  }

  // Error handling wrapper
  catchAsync(fn) {
    return (req, res, next) => {
      Promise.resolve(fn(req, res, next)).catch(next);
    };
  }

  // Standard response methods
  sendSuccess(res, data, statusCode = 200) {
    res.status(statusCode).json({
      status: "success",
      data,
    });
  }

  sendError(res, message, statusCode = 400) {
    res.status(statusCode).json({
      status: "error",
      error: { message },
    });
  }

  // Common CRUD operations
  async getAll(req, res) {
    const filter = this.buildFilter(req.query);
    const options = this.buildPaginationOptions(req.query);

    const items = await this.service.findAll(filter, options);
    const total = await this.service.count(filter);

    this.sendSuccess(res, {
      items,
      pagination: {
        total,
        ...options,
      },
    });
  }

  async getOne(req, res) {
    const item = await this.service.findById(req.params.id);
    if (!item) {
      throw new AppError(`${this.modelName} not found`, 404);
    }
    this.sendSuccess(res, { [this.modelName.toLowerCase()]: item });
  }

  async create(req, res) {
    const item = await this.service.create(req.body);
    this.sendSuccess(res, { [this.modelName.toLowerCase()]: item }, 201);
  }

  async update(req, res) {
    const item = await this.service.update(req.params.id, req.body);
    if (!item) {
      throw new AppError(`${this.modelName} not found`, 404);
    }
    this.sendSuccess(res, { [this.modelName.toLowerCase()]: item });
  }

  async delete(req, res) {
    await this.service.delete(req.params.id);
    res.status(204).send();
  }

  // Helper methods
  buildFilter(query) {
    const filter = { ...query };
    const excludedFields = ["page", "limit", "sort", "fields"];
    excludedFields.forEach((field) => delete filter[field]);
    return filter;
  }

  buildPaginationOptions(query) {
    return {
      page: parseInt(query.page, 10) || 1,
      limit: parseInt(query.limit, 10) || 10,
      sort: query.sort?.split(",").join(" ") || "-createdAt",
      select: query.fields?.split(",").join(" "),
    };
  }
}

export default BaseController;
```

### Implementing Specific Controllers

Now we can create controllers for specific features that inherit from our base controller:

```javascript
// src/controllers/user.controller.js
import BaseController from "./base.controller";
import UserService from "../services/user.service";

class UserController extends BaseController {
  constructor() {
    super(new UserService());
    this.modelName = "User";
  }

  // Custom method for user profile
  async getProfile(req, res) {
    const user = await this.service.findById(req.user.id);
    this.sendSuccess(res, { user });
  }

  // Override create to handle password hashing
  async create(req, res) {
    const { password, ...userData } = req.body;
    const hashedPassword = await this.service.hashPassword(password);

    const user = await this.service.create({
      ...userData,
      password: hashedPassword,
    });

    this.sendSuccess(res, { user: user.toJSON() }, 201);
  }

  // Method for updating user's own profile
  async updateProfile(req, res) {
    const user = await this.service.update(req.user.id, req.body);
    this.sendSuccess(res, { user });
  }

  // Admin-only method to get user statistics
  async getUserStats(req, res) {
    const stats = await this.service.calculateUserStats();
    this.sendSuccess(res, { stats });
  }
}

// Create a single instance
const userController = new UserController();
export default userController;
```

### Setting Up Routes with Controller Methods

Let's connect our controller to routes in a clean way:

```javascript
// src/routes/user.routes.js
import express from "express";
import userController from "../controllers/user.controller";
import { authenticate } from "../middleware/auth";
import { authorize } from "../middleware/authorize";
import { validate } from "../middleware/validate";
import { userSchemas } from "../schemas/user.schemas";

const router = express.Router();

// Define route groupings with middleware
const routes = {
  public: [
    {
      path: "/register",
      method: "post",
      handler: "create",
      middleware: [validate(userSchemas.register)],
    },
  ],
  authenticated: [
    {
      path: "/profile",
      method: "get",
      handler: "getProfile",
    },
    {
      path: "/profile",
      method: "patch",
      handler: "updateProfile",
      middleware: [validate(userSchemas.updateProfile)],
    },
  ],
  admin: [
    {
      path: "/",
      method: "get",
      handler: "getAll",
      middleware: [authorize(["admin"])],
    },
    {
      path: "/stats",
      method: "get",
      handler: "getUserStats",
      middleware: [authorize(["admin"])],
    },
  ],
};

// Register public routes
routes.public.forEach((route) => {
  const middleware = route.middleware || [];
  router[route.method](
    route.path,
    ...middleware,
    userController[route.handler]
  );
});

// Register authenticated routes
routes.authenticated.forEach((route) => {
  const middleware = [authenticate, ...(route.middleware || [])];
  router[route.method](
    route.path,
    ...middleware,
    userController[route.handler]
  );
});

// Register admin routes
routes.admin.forEach((route) => {
  const middleware = [authenticate, ...(route.middleware || [])];
  router[route.method](
    route.path,
    ...middleware,
    userController[route.handler]
  );
});

export default router;
```

[Continue? Let me know if you'd like to see more examples or explore specific aspects in greater detail.]
