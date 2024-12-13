# Error Handling in Express

Error handling is one of the most crucial aspects of building robust Express applications. When things go wrong (and they will), proper error handling ensures your application fails gracefully, maintains security, and provides meaningful feedback to both users and developers.

### Understanding Express Error Handling

Express has a special kind of middleware specifically for handling errors. What makes it unique is that it takes four parameters instead of the usual three:

```javascript
// src/middleware/errorHandler.js
import { logger } from '../utils/logger';

// Central error handling middleware
const errorHandler = (err, req, res, next) => {
  // Log error for debugging and monitoring
  logger.error('Error occurred:', {
    error: {
      message: err.message,
      stack: err.stack,
      name: err.name
    },
    request: {
      method: req.method,
      url: req.originalUrl,
      body: req.body,
      params: req.params,
      query: req.query,
      user: req.user?.id // If you have authentication
    }
  });

  // Set safe defaults
  const status = err.status || 500;
  const message = err.message || 'Internal Server Error';

  // Send appropriate response based on environment
  res.status(status).json({
    error: {
      message: process.env.NODE_ENV === 'production' && status === 500
        ? 'An unexpected error occurred'
        : message,
      code: err.code || 'INTERNAL_ERROR',
      // Include additional details only in development
      ...(process.env.NODE_ENV === 'development' && {
        stack: err.stack,
        details: err.details
      })
    }
  });
};

export default errorHandler;
```

### Creating Custom Error Classes

Having specific error types helps you handle different errors appropriately:

```javascript
// src/utils/errors.js
export class AppError extends Error {
  constructor(message, status, code) {
    super(message);
    this.name = this.constructor.name;
    this.status = status;
    this.code = code;
    Error.captureStackTrace(this, this.constructor);
  }
}

// Specific error types for different scenarios
export class ValidationError extends AppError {
  constructor(message, details = []) {
    super(message, 400, 'VALIDATION_ERROR');
    this.details = details;
  }
}

export class AuthenticationError extends AppError {
  constructor(message = 'Authentication required') {
    super(message, 401, 'AUTHENTICATION_ERROR');
  }
}

export class AuthorizationError extends AppError {
  constructor(message = 'Permission denied') {
    super(message, 403, 'AUTHORIZATION_ERROR');
  }
}

export class NotFoundError extends AppError {
  constructor(resource = 'Resource') {
    super(`${resource} not found`, 404, 'NOT_FOUND_ERROR');
  }
}

export class ConflictError extends AppError {
  constructor(message) {
    super(message, 409, 'CONFLICT_ERROR');
  }
}
```

### Handling Async Errors

Asynchronous error handling requires special attention to ensure errors don't slip through:

```javascript
// src/utils/asyncHandler.js
export const asyncHandler = (fn) => {
  return (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
};

// Usage in routes
import { asyncHandler } from '../utils/asyncHandler';
import { NotFoundError } from '../utils/errors';

router.get('/users/:id', asyncHandler(async (req, res) => {
  const user = await User.findById(req.params.id);
  
  if (!user) {
    throw new NotFoundError('User');
  }
  
  res.json(user);
}));
```

### Database Error Handling

Different databases might throw different types of errors. Here's how to handle MongoDB errors consistently:

```javascript
// src/middleware/databaseErrorHandler.js
import mongoose from 'mongoose';
import { ValidationError, ConflictError } from '../utils/errors';

export const handleDatabaseError = (err, req, res, next) => {
  // Handle Mongoose validation errors
  if (err instanceof mongoose.Error.ValidationError) {
    const details = Object.values(err.errors).map(error => ({
      field: error.path,
      message: error.message,
      value: error.value
    }));

    return next(new ValidationError('Validation failed', details));
  }

  // Handle duplicate key errors
  if (err.code === 11000) {
    const field = Object.keys(err.keyPattern)[0];
    return next(
      new ConflictError(`A record with this ${field} already exists`)
    );
  }

  // Handle other database errors
  if (err instanceof mongoose.Error) {
    return next(new AppError('Database error occurred', 500));
  }

  next(err);
};
```

### Request Validation Error Handling

When using validation libraries like Joi or Yup, standardize their error handling:

```javascript
// src/middleware/validationErrorHandler.js
import { ValidationError } from '../utils/errors';

export const handleValidationError = (schema) => {
  return async (req, res, next) => {
    try {
      // Validate request data against schema
      const validated = await schema.validate({
        body: req.body,
        query: req.query,
        params: req.params
      }, { abortEarly: false });

      // Attach validated data to request
      req.validated = validated;
      next();
    } catch (err) {
      // Transform validation library errors to our format
      const details = err.inner?.map(error => ({
        field: error.path,
        message: error.message,
        value: error.value
      })) || [];

      next(new ValidationError('Validation failed', details));
    }
  };
};
```

### Error Response Formatting

Standardize your error responses for consistency:

```javascript
// src/middleware/errorFormatter.js
export const formatErrorResponse = (err) => {
  const baseError = {
    message: err.message,
    code: err.code,
    status: err.status,
    timestamp: new Date().toISOString()
  };

  // Add request ID if available
  if (err.requestId) {
    baseError.requestId = err.requestId;
  }

  // Add validation details if available
  if (err.details) {
    baseError.details = err.details;
  }

  // Add retry information for rate limiting errors
  if (err.code === 'RATE_LIMIT_ERROR') {
    baseError.retryAfter = err.retryAfter;
  }

  return baseError;
};
```

### Putting It All Together

Here's how to use all these components together:

```javascript
// src/app.js
import express from 'express';
import { errorHandler } from './middleware/errorHandler';
import { handleDatabaseError } from './middleware/databaseErrorHandler';
import { requestIdMiddleware } from './middleware/requestId';
import routes from './routes';

const app = express();

// Add request ID to all requests
app.use(requestIdMiddleware);

// API routes
app.use('/api', routes);

// 404 handler
app.use((req, res, next) => {
  next(new NotFoundError('Route'));
});

// Error handling middleware (order matters)
app.use(handleDatabaseError);
app.use(errorHandler);

export default app;
```

This setup provides:
- Consistent error handling across your application
- Proper error logging for debugging
- Security through appropriate error message filtering
- Type-specific error handling
- Standardized error responses
- Request tracking for better debugging
- Validation error handling
- Database error normalization

Would you like me to explain any of these concepts in more detail?