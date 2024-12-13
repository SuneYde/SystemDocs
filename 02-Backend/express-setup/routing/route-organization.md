# Route Organization in Express

Good route organization is crucial for building maintainable Express applications. As your application grows, having a clear and logical structure for your routes helps developers understand the API's layout, makes debugging easier, and simplifies the process of adding new features.

### Basic Route Structure

Let's start with a scalable folder structure that organizes routes by feature:

```javascript
// src/routes/index.js
import express from 'express';
import userRoutes from './user.routes';
import authRoutes from './auth.routes';
import productRoutes from './product.routes';
import orderRoutes from './order.routes';

const router = express.Router();

// We version our API to allow for future changes
router.use('/v1/users', userRoutes);
router.use('/v1/auth', authRoutes);
router.use('/v1/products', productRoutes);
router.use('/v1/orders', orderRoutes);

export default router;

// src/routes/user.routes.js
import express from 'express';
import { authenticate } from '../middleware/auth';
import { validateRequest } from '../middleware/validation';
import { userSchemas } from '../schemas/user.schemas';
import * as userController from '../controllers/user.controller';

const router = express.Router();

// Group related routes together with meaningful comments
// User profile management
router.get(
  '/profile',
  authenticate,
  userController.getProfile
);

router.patch(
  '/profile',
  authenticate,
  validateRequest(userSchemas.updateProfile),
  userController.updateProfile
);

// User preferences
router.get(
  '/preferences',
  authenticate,
  userController.getPreferences
);

router.put(
  '/preferences',
  authenticate,
  validateRequest(userSchemas.updatePreferences),
  userController.updatePreferences
);

// Admin-only routes for user management
router.get(
  '/',
  authenticate,
  authorize(['admin']),
  userController.listUsers
);

router.get(
  '/:userId',
  authenticate,
  authorize(['admin']),
  userController.getUserById
);

export default router;
```

### Feature-Based Organization

For larger features, we can create sub-routers to better organize related endpoints:

```javascript
// src/routes/product.routes.js
import express from 'express';
import reviewRoutes from './product/review.routes';
import categoryRoutes from './product/category.routes';
import inventoryRoutes from './product/inventory.routes';
import * as productController from '../controllers/product.controller';

const router = express.Router();

// Mount sub-routers for product-related features
router.use('/:productId/reviews', reviewRoutes);
router.use('/categories', categoryRoutes);
router.use('/inventory', inventoryRoutes);

// Main product routes
router.get('/', productController.listProducts);
router.post('/', 
  authenticate,
  authorize(['admin', 'vendor']),
  validateRequest(productSchemas.createProduct),
  productController.createProduct
);

export default router;

// src/routes/product/review.routes.js
import express from 'express';
const router = express.Router({ mergeParams: true }); // Important for accessing parent route params

router.get('/', reviewController.getProductReviews);
router.post('/',
  authenticate,
  validateRequest(reviewSchemas.createReview),
  reviewController.createReview
);

export default router;
```

### Route Parameter Handling

When dealing with routes that need access to parameters, we can create parameter middleware:

```javascript
// src/middleware/params.js
import { NotFoundError } from '../utils/errors';

export const resolveUserParam = async (req, res, next, userId) => {
  try {
    const user = await User.findById(userId);
    if (!user) {
      throw new NotFoundError('User');
    }
    // Attach user to request for use in route handlers
    req.resolvedUser = user;
    next();
  } catch (error) {
    next(error);
  }
};

// src/routes/user.routes.js
const router = express.Router();

// Register parameter middleware
router.param('userId', resolveUserParam);

// Now all routes with :userId will have access to req.resolvedUser
router.get('/:userId', userController.getUserById);
router.patch('/:userId', userController.updateUser);
router.delete('/:userId', userController.deleteUser);
```

### Route Configuration and Documentation

We can enhance our routes with configuration and documentation:

```javascript
// src/routes/config/route.config.js
export const RouteConfig = {
  // Define route metadata and configuration
  users: {
    list: {
      method: 'GET',
      path: '/',
      auth: true,
      roles: ['admin'],
      rateLimitOptions: {
        windowMs: 15 * 60 * 1000,
        max: 100
      },
      cacheOptions: {
        ttl: 60 // seconds
      }
    },
    create: {
      method: 'POST',
      path: '/',
      auth: true,
      roles: ['admin'],
      validation: 'createUser'
    }
  }
};

// src/routes/factories/route.factory.js
import { authenticate } from '../../middleware/auth';
import { authorize } from '../../middleware/authorize';
import { rateLimit } from '../../middleware/rateLimit';
import { cache } from '../../middleware/cache';

export const createRoute = (router, config, handler) => {
  const middleware = [];

  // Add authentication if required
  if (config.auth) {
    middleware.push(authenticate);
  }

  // Add role-based authorization if specified
  if (config.roles?.length) {
    middleware.push(authorize(config.roles));
  }

  // Add rate limiting if configured
  if (config.rateLimitOptions) {
    middleware.push(rateLimit(config.rateLimitOptions));
  }

  // Add caching if configured
  if (config.cacheOptions) {
    middleware.push(cache(config.cacheOptions));
  }

  // Add validation if specified
  if (config.validation) {
    middleware.push(validateRequest(schemas[config.validation]));
  }

  // Register route with all middleware
  router[config.method.toLowerCase()](
    config.path,
    ...middleware,
    handler
  );
};

// Using the route factory
import { RouteConfig } from '../config/route.config';
import * as userController from '../controllers/user.controller';

const router = express.Router();

// Create routes from configuration
Object.entries(RouteConfig.users).forEach(([name, config]) => {
  createRoute(router, config, userController[name]);
});
```

### API Documentation Integration

We can integrate route documentation directly into our route definitions:

```javascript
// src/utils/docs.js
export const apiDoc = (config) => (target, key, descriptor) => {
  if (!target.apiDocs) {
    target.apiDocs = {};
  }
  target.apiDocs[key] = config;
  return descriptor;
};

// src/controllers/user.controller.js
class UserController {
  @apiDoc({
    summary: 'Get user profile',
    description: 'Retrieves the profile of the authenticated user',
    responses: {
      200: {
        description: 'User profile retrieved successfully',
        schema: 'UserProfileResponse'
      },
      401: {
        description: 'Unauthorized - User not authenticated'
      }
    }
  })
  async getProfile(req, res) {
    // Controller implementation
  }
}
```

This organization provides:
- Clear separation of concerns
- Scalable structure for growing applications
- Easy-to-understand route grouping
- Maintainable parameter handling
- Configuration-driven route creation
- Integrated API documentation
- Consistent middleware application

Would you like me to explain any of these concepts in more detail?