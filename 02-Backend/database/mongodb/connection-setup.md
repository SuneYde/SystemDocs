# MongoDB Connection Setup Guide

Setting up a MongoDB connection properly is crucial for your application's reliability and performance. A well-configured connection can handle application crashes, network issues, and high traffic loads gracefully. Let's explore how to create a robust MongoDB connection setup.

### Basic Connection Setup

Let's start with a foundation that includes essential error handling and connection management:

```javascript
// src/config/database.js
import mongoose from "mongoose";
import logger from "../utils/logger";

class Database {
  constructor() {
    // Store the connection promise for reuse
    this.connectionPromise = null;

    // Configure mongoose to use native JavaScript promises
    mongoose.Promise = global.Promise;

    // Listen for mongoose events
    this.setupMongooseEvents();
  }

  async connect() {
    try {
      // Only create a new connection if one doesn't exist
      if (!this.connectionPromise) {
        const connectionOptions = {
          // Use the new URL parser
          // Why: The old parser is deprecated and less reliable
          useNewUrlParser: true,

          // Use the new Server Discovery and Monitoring engine
          // Why: Provides better handling of MongoDB topology changes
          useUnifiedTopology: true,

          // Maximum number of connections in the pool
          // Why: Prevents resource exhaustion
          maxPoolSize: 10,

          // Minimum number of connections in the pool
          // Why: Keeps connections ready for sudden traffic spikes
          minPoolSize: 2,

          // How long to wait for a connection from the pool
          // Why: Prevents requests from hanging indefinitely
          connectTimeoutMS: 10000,

          // How long to wait for a response from MongoDB
          // Why: Prevents slow queries from blocking other operations
          socketTimeoutMS: 45000,

          // Keep connections alive
          // Why: Prevents firewalls from closing idle connections
          keepAlive: true,
          keepAliveInitialDelay: 300000, // 5 minutes
        };

        // Create the connection
        this.connectionPromise = mongoose.connect(
          process.env.MONGODB_URI,
          connectionOptions
        );

        // Wait for the connection to be established
        await this.connectionPromise;
        logger.info("Successfully connected to MongoDB");
      }

      return this.connectionPromise;
    } catch (error) {
      logger.error("MongoDB connection error:", error);
      throw error;
    }
  }

  setupMongooseEvents() {
    // Listen for connection errors
    mongoose.connection.on("error", (error) => {
      logger.error("MongoDB connection error:", error);
    });

    // Listen for disconnection events
    mongoose.connection.on("disconnected", () => {
      logger.warn("MongoDB disconnected. Attempting to reconnect...");
    });

    // Listen for successful reconnection
    mongoose.connection.on("reconnected", () => {
      logger.info("MongoDB reconnected successfully");
    });

    // Handle process termination
    process.on("SIGINT", this.gracefulShutdown.bind(this));
    process.on("SIGTERM", this.gracefulShutdown.bind(this));
  }

  async gracefulShutdown() {
    try {
      logger.info("Closing MongoDB connection...");
      await mongoose.connection.close();
      logger.info("MongoDB connection closed successfully");
      process.exit(0);
    } catch (error) {
      logger.error("Error closing MongoDB connection:", error);
      process.exit(1);
    }
  }

  // Get the current connection status
  getConnectionState() {
    return mongoose.connection.readyState;
  }
}

// Create a singleton instance
const database = new Database();
export default database;
```

### Advanced Connection Options

For production applications, we might want to add more sophisticated configuration options:

```javascript
// src/config/database-advanced.js
import mongoose from "mongoose";
import logger from "../utils/logger";

class AdvancedDatabase extends Database {
  constructor() {
    super();
    this.retryAttempts = 0;
    this.maxRetryAttempts = 5;
  }

  async connect() {
    try {
      if (!this.connectionPromise) {
        const advancedOptions = {
          // All previous options plus:

          // Write Concern settings
          // Why: Ensures data durability
          w: "majority", // Wait for writes to be acknowledged by majority of replicas
          wtimeout: 5000, // Maximum time to wait for write concern

          // Read Preference settings
          // Why: Controls how reads are distributed across replicas
          readPreference: "secondaryPreferred",

          // Replica Set settings
          // Why: Ensures high availability
          replicaSet: process.env.MONGODB_REPLICA_SET,

          // Server Selection settings
          // Why: Controls how long to wait when finding a server
          serverSelectionTimeoutMS: 5000,

          // Heartbeat settings
          // Why: Controls how often to check server status
          heartbeatFrequencyMS: 10000,

          // Compression settings
          // Why: Reduces network bandwidth usage
          compressors: ["zlib"],

          // Auto-Index creation
          // Why: Prevents automatic index creation in production
          autoIndex: process.env.NODE_ENV !== "production",

          // Buffer commands until connection is established
          // Why: Prevents losing operations during connection
          bufferCommands: true,

          // Connection pool monitoring
          // Why: Helps track connection pool usage
          monitorCommands: process.env.NODE_ENV !== "production",
        };

        this.connectionPromise = this.connectWithRetry(advancedOptions);
        await this.connectionPromise;
      }

      return this.connectionPromise;
    } catch (error) {
      logger.error("Failed to establish MongoDB connection:", error);
      throw error;
    }
  }

  async connectWithRetry(options) {
    while (this.retryAttempts < this.maxRetryAttempts) {
      try {
        return await mongoose.connect(process.env.MONGODB_URI, options);
      } catch (error) {
        this.retryAttempts++;
        const delay = Math.min(1000 * Math.pow(2, this.retryAttempts), 10000);

        logger.warn(
          `MongoDB connection attempt ${this.retryAttempts} failed. ` +
            `Retrying in ${delay}ms...`
        );

        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    throw new Error("Maximum MongoDB connection retry attempts reached");
  }

  // Add connection pool monitoring
  setupConnectionMonitoring() {
    mongoose.connection.on("connected", () => {
      const poolStats = mongoose.connection.db.admin().serverStatus();
      logger.info("MongoDB pool stats:", {
        activeConnections: poolStats.connections.current,
        availableConnections: poolStats.connections.available,
        totalConnections: poolStats.connections.totalCreated,
      });
    });
  }
}
```

### Connection Health Checks

Adding health checks helps monitor your database connection:

```javascript
// src/utils/database-health.js
class DatabaseHealthCheck {
  constructor(database) {
    this.database = database;
    this.healthStatus = {
      isConnected: false,
      lastCheck: null,
      error: null,
    };
  }

  async checkHealth() {
    try {
      // Check if we can execute a simple command
      await mongoose.connection.db.admin().ping();

      this.healthStatus = {
        isConnected: true,
        lastCheck: new Date(),
        error: null,
      };

      return this.healthStatus;
    } catch (error) {
      this.healthStatus = {
        isConnected: false,
        lastCheck: new Date(),
        error: error.message,
      };

      logger.error("Database health check failed:", error);
      return this.healthStatus;
    }
  }

  // Start periodic health checks
  startHealthChecks(interval = 30000) {
    this.healthCheckInterval = setInterval(() => this.checkHealth(), interval);
  }

  stopHealthChecks() {
    if (this.healthCheckInterval) {
      clearInterval(this.healthCheckInterval);
    }
  }
}
```

### Using the Database Setup

Here's how to use this setup in your application:

```javascript
// src/app.js
import express from "express";
import database from "./config/database";
import { DatabaseHealthCheck } from "./utils/database-health";

const app = express();

// Initialize database connection
const initializeDatabase = async () => {
  try {
    await database.connect();

    // Set up health checking
    const healthChecker = new DatabaseHealthCheck(database);
    healthChecker.startHealthChecks();

    // Add health check endpoint
    app.get("/health/db", async (req, res) => {
      const health = await healthChecker.checkHealth();
      const statusCode = health.isConnected ? 200 : 503;
      res.status(statusCode).json(health);
    });
  } catch (error) {
    logger.error("Failed to initialize database:", error);
    process.exit(1);
  }
};

// Start the application
const startApp = async () => {
  await initializeDatabase();

  const port = process.env.PORT || 3000;
  app.listen(port, () => {
    logger.info(`Server is running on port ${port}`);
  });
};

startApp();
```

This setup provides:

- Robust error handling
- Connection pooling
- Automatic reconnection
- Health monitoring
- Graceful shutdown
- Production-ready configuration
- Performance optimizations

Would you like me to explain any part of this in more detail?
