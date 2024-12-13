# Express Server Configuration Guide

Setting up an Express server properly is crucial for building secure, maintainable, and scalable applications. Let's explore how to create a robust server configuration that can handle production workloads.

### Environment Configuration

First, we'll set up proper environment management. This helps us keep our application secure and configurable across different environments (development, staging, production):

```javascript
// src/config/environment.js
import dotenv from 'dotenv';
import path from 'path';

// Load appropriate .env file based on environment
dotenv.config({
  path: path.join(__dirname, `../../.env.${process.env.NODE_ENV || 'development'}`)
});

const environment = {
  // Node environment
  NODE_ENV: process.env.NODE_ENV || 'development',
  
  // Server configuration
  server: {
    port: parseInt(process.env.PORT, 10) || 5000,
    host: process.env.HOST || '0.0.0.0',
    // Trust proxy if we're behind a load balancer
    trustProxy: process.env.TRUST_PROXY === 'true',
    
    // API versioning
    apiVersion: process.env.API_VERSION || 'v1',
    
    // Request timeouts
    timeout: {
      server: parseInt(process.env.SERVER_TIMEOUT, 10) || 30000,
      // Separate timeout for file uploads
      upload: parseInt(process.env.UPLOAD_TIMEOUT, 10) || 60000,
    }
  },

  // Security settings
  security: {
    // CORS configuration
    cors: {
      origin: process.env.CORS_ORIGIN?.split(',') || ['http://localhost:3000'],
      credentials: true,
      maxAge: 86400 // 24 hours in seconds
    },
    
    // Rate limiting
    rateLimit: {
      windowMs: 15 * 60 * 1000, // 15 minutes
      max: parseInt(process.env.RATE_LIMIT_MAX, 10) || 100
    },
    
    // Content Security Policy options
    csp: {
      directives: {
        defaultSrc: ["'self'"],
        scriptSrc: ["'self'", "'unsafe-inline'"],
        styleSrc: ["'self'", "'unsafe-inline'"],
        imgSrc: ["'self'", 'data:', 'https:'],
      }
    }
  },

  // Database configuration
  database: {
    url: process.env.MONGODB_URI,
    options: {
      useNewUrlParser: true,
      useUnifiedTopology: true,
      maxPoolSize: parseInt(process.env.DB_POOL_SIZE, 10) || 10,
      serverSelectionTimeoutMS: 5000,
      socketTimeoutMS: 45000,
    }
  },

  // Logging configuration
  logging: {
    level: process.env.LOG_LEVEL || 'info',
    // Disable logging in test environment
    enabled: process.env.NODE_ENV !== 'test',
    // Pretty print logs in development
    pretty: process.env.NODE_ENV === 'development',
    // Additional logging options
    options: {
      // Request logging
      requests: true,
      // Query logging
      queries: process.env.NODE_ENV === 'development',
      // Error logging
      errors: true
    }
  },

  // File upload configuration
  upload: {
    maxFileSize: parseInt(process.env.MAX_FILE_SIZE, 10) || 5 * 1024 * 1024, // 5MB
    allowedMimeTypes: process.env.ALLOWED_MIME_TYPES?.split(',') || [
      'image/jpeg',
      'image/png',
      'image/gif',
      'application/pdf'
    ]
  }
};

export default environment;
```

### Server Bootstrap Configuration

Now let's create the main server configuration that uses these environment settings:

```javascript
// src/config/server.js
import express from 'express';
import helmet from 'helmet';
import cors from 'cors';
import compression from 'compression';
import { rateLimit } from 'express-rate-limit';
import mongoSanitize from 'express-mongo-sanitize';
import xss from 'xss-clean';
import hpp from 'hpp';
import environment from './environment';

export function configureServer(app) {
  // If we're behind a proxy (like nginx)
  if (environment.server.trustProxy) {
    app.enable('trust proxy');
  }

  // Basic security headers with helmet
  app.use(helmet({
    contentSecurityPolicy: environment.security.csp,
    crossOriginEmbedderPolicy: true,
    crossOriginOpenerPolicy: true,
    crossOriginResourcePolicy: { policy: "cross-origin" },
    dnsPrefetchControl: true,
    frameguard: { action: 'deny' },
    hidePoweredBy: true,
    hsts: true,
    ieNoOpen: true,
    noSniff: true,
    originAgentCluster: true,
    permittedCrossDomainPolicies: false,
    referrerPolicy: { policy: 'same-origin' },
    xssFilter: true,
  }));

  // CORS configuration
  app.use(cors(environment.security.cors));

  // Request parsing
  app.use(express.json({ limit: '10kb' })); // Limit JSON payload size
  app.use(express.urlencoded({ extended: true, limit: '10kb' }));

  // Security middleware
  app.use(mongoSanitize()); // Prevent NoSQL injection
  app.use(xss()); // Clean user input
  app.use(hpp()); // Prevent HTTP Parameter Pollution

  // Rate limiting
  const limiter = rateLimit(environment.security.rateLimit);
  app.use('/api/', limiter);

  // Compression
  app.use(compression({
    filter: (req, res) => {
      if (req.headers['x-no-compression']) {
        return false;
      }
      return compression.filter(req, res);
    },
    level: 6 // Balanced compression level
  }));

  // Timeouts
  app.use((req, res, next) => {
    // Set different timeouts for different types of requests
    const timeout = req.url.startsWith('/api/upload')
      ? environment.server.timeout.upload
      : environment.server.timeout.server;

    req.setTimeout(timeout, () => {
      res.status(408).send('Request Timeout');
    });

    next();
  });

  return app;
}

// Error handling configuration
export function configureErrorHandling(app) {
  // 404 handler
  app.use((req, res) => {
    res.status(404).json({
      error: {
        message: 'Resource not found',
        status: 404
      }
    });
  });

  // Global error handler
  app.use((err, req, res, next) => {
    const status = err.status || 500;
    const message = err.message || 'Internal server error';

    // Don't leak error details in production
    const error = {
      message,
      status,
      ...(environment.NODE_ENV === 'development' && {
        stack: err.stack,
        details: err.details
      })
    };

    res.status(status).json({ error });
  });

  return app;
}
```

### Example Usage

Here's how to use this configuration in your main server file:

```javascript
// src/server.js
import express from 'express';
import { configureServer, configureErrorHandling } from './config/server';
import environment from './config/environment';
import logger from './utils/logger';
import routes from './routes';

async function startServer() {
  try {
    const app = express();

    // Apply server configuration
    configureServer(app);

    // Mount routes
    app.use(`/api/${environment.server.apiVersion}`, routes);

    // Apply error handling
    configureErrorHandling(app);

    // Start listening
    const server = app.listen(environment.server.port, environment.server.host, () => {
      logger.info(`Server running on port ${environment.server.port} in ${environment.NODE_ENV} mode`);
    });

    // Graceful shutdown
    const shutdown = async () => {
      logger.info('Shutting down server...');
      
      server.close(async () => {
        try {
          // Close database connections
          await mongoose.connection.close();
          logger.info('Server shut down successfully');
          process.exit(0);
        } catch (error) {
          logger.error('Error during shutdown:', error);
          process.exit(1);
        }
      });

      // Force shutdown if graceful shutdown fails
      setTimeout(() => {
        logger.error('Could not close connections in time, forcefully shutting down');
        process.exit(1);
      }, 10000);
    };

    // Handle shutdown signals
    process.on('SIGTERM', shutdown);
    process.on('SIGINT', shutdown);

    return server;
  } catch (error) {
    logger.error('Error starting server:', error);
    process.exit(1);
  }
}

startServer();
```

This setup provides a robust foundation for your Express server with:
- Comprehensive environment configuration
- Strong security measures
- Error handling
- Request timeouts
- Rate limiting
- Graceful shutdown
- Proper CORS configuration
- Compression
- And more

Would you like me to explain any of these concepts in more detail?