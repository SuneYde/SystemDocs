# Environment Variables in Express

Environment variables are a crucial part of any production-ready application. They allow us to keep sensitive information secure and manage different configurations across various environments. Let's explore how to set up and manage environment variables effectively in an Express application.

### Setting Up Environment Variables

First, let's create a robust environment variables setup that can handle multiple environments:

```javascript
// src/config/env.js
import dotenv from 'dotenv';
import path from 'path';
import fs from 'fs';

// Determines which .env file to load based on the environment
function loadEnvFile() {
    const environment = process.env.NODE_ENV || 'development';
    const envPath = path.join(__dirname, `../../.env.${environment}`);
    const defaultEnvPath = path.join(__dirname, '../../.env');

    // Check if environment-specific file exists
    if (fs.existsSync(envPath)) {
        dotenv.config({ path: envPath });
    } else {
        // Fall back to default .env file
        dotenv.config({ path: defaultEnvPath });
    }
}

// Load environment variables
loadEnvFile();

// Validate required environment variables
function validateEnv() {
    const requiredEnvVars = [
        'NODE_ENV',
        'PORT',
        'MONGODB_URI',
        'JWT_SECRET',
    ];

    const missingEnvVars = requiredEnvVars.filter(envVar => !process.env[envVar]);

    if (missingEnvVars.length > 0) {
        throw new Error(
            `Missing required environment variables: ${missingEnvVars.join(', ')}`
        );
    }
}

// Create configuration object with type validation
const env = {
    // Server Configuration
    node_env: process.env.NODE_ENV || 'development',
    port: parseInt(process.env.PORT, 10) || 3000,
    host: process.env.HOST || 'localhost',

    // Database Configuration
    database: {
        uri: process.env.MONGODB_URI,
        options: {
            maxPoolSize: parseInt(process.env.DB_POOL_SIZE, 10) || 10,
            retryWrites: process.env.DB_RETRY_WRITES !== 'false',
        }
    },

    // Authentication Configuration
    auth: {
        jwt_secret: process.env.JWT_SECRET,
        jwt_expires_in: process.env.JWT_EXPIRES_IN || '7d',
        jwt_refresh_expires_in: process.env.JWT_REFRESH_EXPIRES_IN || '30d',
        bcrypt_rounds: parseInt(process.env.BCRYPT_ROUNDS, 10) || 12,
    },

    // Email Configuration
    email: {
        smtp_host: process.env.SMTP_HOST,
        smtp_port: parseInt(process.env.SMTP_PORT, 10) || 587,
        smtp_user: process.env.SMTP_USER,
        smtp_pass: process.env.SMTP_PASS,
        from_email: process.env.FROM_EMAIL || 'noreply@example.com',
    },

    // AWS Configuration (if using AWS services)
    aws: {
        access_key_id: process.env.AWS_ACCESS_KEY_ID,
        secret_access_key: process.env.AWS_SECRET_ACCESS_KEY,
        region: process.env.AWS_REGION || 'us-east-1',
        s3_bucket: process.env.AWS_S3_BUCKET,
    },

    // Redis Configuration (if using Redis)
    redis: {
        url: process.env.REDIS_URL,
        host: process.env.REDIS_HOST || 'localhost',
        port: parseInt(process.env.REDIS_PORT, 10) || 6379,
        password: process.env.REDIS_PASSWORD,
    },

    // Logging Configuration
    logging: {
        level: process.env.LOG_LEVEL || 'info',
        filename: process.env.LOG_FILE || 'app.log',
    },

    // API Rate Limiting
    rate_limit: {
        window_ms: parseInt(process.env.RATE_LIMIT_WINDOW_MS, 10) || 15 * 60 * 1000,
        max_requests: parseInt(process.env.RATE_LIMIT_MAX_REQUESTS, 10) || 100,
    },

    // CORS Configuration
    cors: {
        origin: process.env.CORS_ORIGIN?.split(',') || ['http://localhost:3000'],
        credentials: process.env.CORS_CREDENTIALS === 'true',
    }
};

// Helper function to check if we're in production
env.isProduction = () => env.node_env === 'production';

// Helper function to check if we're in development
env.isDevelopment = () => env.node_env === 'development';

// Helper function to check if we're in test environment
env.isTest = () => env.node_env === 'test';

// Validate environment variables
try {
    validateEnv();
} catch (error) {
    console.error('Environment validation failed:', error.message);
    process.exit(1);
}

export default env;
```

### Example Environment Files

Here's how to structure your environment files:

```bash
# .env.development
NODE_ENV=development
PORT=3000
MONGODB_URI=mongodb://localhost:27017/myapp_dev
JWT_SECRET=dev_secret_key
LOG_LEVEL=debug

# .env.production
NODE_ENV=production
PORT=80
MONGODB_URI=mongodb+srv://user:pass@cluster.mongodb.net/myapp_prod
JWT_SECRET=long_random_production_secret
LOG_LEVEL=info

# .env.test
NODE_ENV=test
PORT=3001
MONGODB_URI=mongodb://localhost:27017/myapp_test
JWT_SECRET=test_secret_key
LOG_LEVEL=error
```

### Usage in Application Code

Here's how to use the environment configuration throughout your application:

```javascript
// src/server.js
import env from './config/env';
import express from 'express';

const app = express();

// Use environment variables in server setup
app.listen(env.port, () => {
    console.log(`Server running on port ${env.port} in ${env.node_env} mode`);
});

// src/database/connection.js
import mongoose from 'mongoose';
import env from '../config/env';

async function connectDatabase() {
    try {
        await mongoose.connect(env.database.uri, env.database.options);
        console.log('Database connected successfully');
    } catch (error) {
        console.error('Database connection failed:', error);
        process.exit(1);
    }
}

// src/middleware/auth.js
import jwt from 'jsonwebtoken';
import env from '../config/env';

export const verifyToken = (token) => {
    try {
        return jwt.verify(token, env.auth.jwt_secret);
    } catch (error) {
        throw new Error('Invalid token');
    }
};

// src/services/email.js
import nodemailer from 'nodemailer';
import env from '../config/env';

const transporter = nodemailer.createTransport({
    host: env.email.smtp_host,
    port: env.email.smtp_port,
    auth: {
        user: env.email.smtp_user,
        pass: env.email.smtp_pass
    }
});
```

### Environment Variable Security

To maintain security when working with environment variables:

```javascript
// src/utils/secureConfig.js
import env from '../config/env';

// Sanitize configuration for logging
export const sanitizeConfig = (config) => {
    const sensitiveKeys = [
        'jwt_secret',
        'smtp_pass',
        'aws_secret_access_key',
        'redis_password'
    ];

    return Object.entries(config).reduce((acc, [key, value]) => {
        if (typeof value === 'object' && value !== null) {
            acc[key] = sanitizeConfig(value);
        } else if (sensitiveKeys.some(k => key.toLowerCase().includes(k))) {
            acc[key] = '[REDACTED]';
        } else {
            acc[key] = value;
        }
        return acc;
    }, {});
};

// Use when logging configuration
console.log('App config:', sanitizeConfig(env));
```

### Environment-Specific Features

You can use environment variables to enable/disable features based on the environment:

```javascript
// src/middleware/logger.js
import env from '../config/env';

export const requestLogger = (req, res, next) => {
    // Only log requests in development
    if (env.isDevelopment()) {
        console.log(`${req.method} ${req.path}`);
    }
    next();
};

// src/middleware/errorHandler.js
export const errorHandler = (err, req, res, next) => {
    // Include stack traces only in development
    const error = {
        message: err.message,
        status: err.status || 500,
        ...(env.isDevelopment() && { stack: err.stack })
    };
    
    res.status(error.status).json({ error });
};
```

This setup provides a robust foundation for managing environment variables in your Express application. Remember to:
- Never commit environment files to version control
- Use different variables for different environments
- Validate required variables on startup
- Sanitize sensitive information when logging
- Use type conversion for numeric values
- Provide sensible defaults where appropriate

Would you like me to explain any of these concepts in more detail?