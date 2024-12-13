# Creating Custom Middleware in Express

Custom middleware allows you to add your own processing steps to Express's request handling pipeline. Think of middleware as checkpoints that each request must pass through - you can inspect, modify, or even stop requests at each checkpoint. Let's explore how to create effective custom middleware.

### Understanding Middleware Basics

A middleware function in Express has access to three important objects: the request object (req), the response object (res), and the next middleware function (next). Here's how to create a basic middleware:

```javascript
// Basic middleware structure
const simpleLogger = (req, res, next) => {
  // 1. You can read the request
  console.log(`${req.method} ${req.path}`);
  
  // 2. You can modify the request or response objects
  req.requestTime = new Date();
  
  // 3. You must either:
  //    - Call next() to pass control to the next middleware
  //    - Send a response using res.send(), res.json(), etc.
  next();
};

// Using the middleware
app.use(simpleLogger);
```

### Creating Authentication Middleware

Authentication is one of the most common use cases for custom middleware. Here's how to create a robust authentication system:

```javascript
// Authentication middleware with error handling
const authenticate = async (req, res, next) => {
  try {
    // Get token from authorization header
    const authHeader = req.headers.authorization;
    if (!authHeader?.startsWith('Bearer ')) {
      throw new Error('No token provided');
    }

    const token = authHeader.split(' ')[1];
    
    // Verify the token
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    
    // Add user information to request object
    // This makes it available to all subsequent middleware and routes
    req.user = decoded;
    
    // Important: If everything is okay, call next()
    next();
  } catch (error) {
    // If anything goes wrong, create an error response
    res.status(401).json({
      error: {
        message: 'Authentication failed',
        details: error.message
      }
    });
  }
};

// Role-based authorization middleware
const authorize = (...allowedRoles) => {
  return (req, res, next) => {
    // This middleware depends on authenticate running first
    if (!req.user) {
      return res.status(401).json({
        error: 'Authentication required'
      });
    }

    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({
        error: 'Insufficient permissions'
      });
    }

    next();
  };
};

// Using authentication and authorization together
app.get('/admin/users', 
  authenticate,
  authorize('admin', 'superadmin'),
  (req, res) => {
    // Handle route
  }
);
```

### Request Validation Middleware

Validation middleware helps ensure that incoming requests meet your requirements:

```javascript
// General validation middleware creator
const validateRequest = (schema) => {
  return (req, res, next) => {
    try {
      // Schema can validate different parts of the request
      const validationResult = schema.validate({
        body: req.body,
        query: req.query,
        params: req.params
      }, { abortEarly: false });

      if (validationResult.error) {
        // Transform validation errors into a cleaner format
        const errors = validationResult.error.details.map(error => ({
          field: error.path.join('.'),
          message: error.message
        }));

        return res.status(400).json({
          error: {
            message: 'Validation failed',
            details: errors
          }
        });
      }

      // Add validated data to request object
      req.validated = validationResult.value;
      next();
    } catch (error) {
      next(error);
    }
  };
};

// Using the validation middleware with a schema
const userSchema = Joi.object({
  body: Joi.object({
    email: Joi.string().email().required(),
    password: Joi.string().min(8).required(),
    name: Joi.string().required()
  })
});

app.post('/users', validateRequest(userSchema), (req, res) => {
  // Handle validated request
  const { email, password, name } = req.validated.body;
});
```

### Performance Monitoring Middleware

Monitor your application's performance with custom middleware:

```javascript
// Performance monitoring middleware
const performanceMonitor = (req, res, next) => {
  // Store start time
  const start = process.hrtime();

  // Store original end function
  const originalEnd = res.end;

  // Override end function to calculate duration
  res.end = function(...args) {
    // Calculate duration
    const duration = process.hrtime(start);
    const durationMs = (duration[0] * 1000) + (duration[1] / 1e6);

    // Log performance metrics
    console.log({
      method: req.method,
      path: req.path,
      duration: `${durationMs.toFixed(2)}ms`,
      status: res.statusCode,
      contentLength: res.get('Content-Length'),
      userAgent: req.get('User-Agent')
    });

    // Call original end function
    originalEnd.apply(res, args);
  };

  next();
};

// Add request tracking
app.use(performanceMonitor);
```

### Error Handling Middleware

Custom error handling middleware can provide consistent error responses:

```javascript
// Custom error class
class AppError extends Error {
  constructor(message, statusCode = 500, code = 'INTERNAL_ERROR') {
    super(message);
    this.statusCode = statusCode;
    this.code = code;
  }
}

// Error handling middleware
const errorHandler = (err, req, res, next) => {
  // Log the error
  console.error('Error:', {
    message: err.message,
    stack: err.stack,
    code: err.code,
    url: req.originalUrl,
    method: req.method
  });

  // Different handling for different types of errors
  if (err instanceof AppError) {
    return res.status(err.statusCode).json({
      error: {
        message: err.message,
        code: err.code
      }
    });
  }

  if (err.name === 'ValidationError') {
    return res.status(400).json({
      error: {
        message: 'Validation Error',
        details: Object.values(err.errors).map(e => e.message)
      }
    });
  }

  // Default error response
  res.status(500).json({
    error: {
      message: 'An unexpected error occurred',
      code: 'INTERNAL_ERROR'
    }
  });
};

// Using error handling
app.use(errorHandler);
```

### Request Rate Limiting Middleware

Create custom rate limiting for specific routes or actions:

```javascript
// Rate limiting middleware creator
const createRateLimiter = (options = {}) => {
  const {
    windowMs = 60 * 1000, // 1 minute default
    max = 100, // 100 requests per windowMs
    message = 'Too many requests'
  } = options;

  // Store for tracking requests
  const requests = new Map();

  // Cleanup old entries periodically
  setInterval(() => {
    const now = Date.now();
    for (const [key, data] of requests.entries()) {
      if (now - data.startTime > windowMs) {
        requests.delete(key);
      }
    }
  }, windowMs);

  return (req, res, next) => {
    // Get identifier (e.g., IP address or user ID)
    const key = req.user?.id || req.ip;
    const now = Date.now();

    if (!requests.has(key)) {
      requests.set(key, {
        count: 1,
        startTime: now
      });
      return next();
    }

    const data = requests.get(key);
    
    // Reset if window has passed
    if (now - data.startTime > windowMs) {
      data.count = 1;
      data.startTime = now;
      return next();
    }

    // Check limit
    if (data.count >= max) {
      return res.status(429).json({
        error: {
          message,
          retryAfter: Math.ceil((data.startTime + windowMs - now) / 1000)
        }
      });
    }

    // Increment count
    data.count++;
    next();
  };
};

// Using the rate limiter
app.post('/api/comments',
  createRateLimiter({
    windowMs: 60 * 1000, // 1 minute
    max: 5 // 5 comments per minute
  }),
  (req, res) => {
    // Handle comment creation
  }
);
```

Would you like me to explain any of these patterns in more detail?