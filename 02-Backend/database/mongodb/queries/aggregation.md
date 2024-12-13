# MongoDB Aggregation Framework Guide

The aggregation framework is MongoDB's powerful tool for data analysis and transformation. Think of it as a pipeline where your data flows through different stages, being shaped and transformed at each step. Let's explore how to use this framework effectively for data processing and analysis.

### Understanding Basic Aggregation

Let's start with a simple example and build up to more complex operations:

```javascript
// src/services/order.service.js
import mongoose from "mongoose";

class OrderAnalytics {
  // Basic aggregation to get sales summary
  async getSalesSummary(startDate, endDate) {
    const pipeline = [
      // Stage 1: Match orders within date range
      {
        $match: {
          createdAt: {
            $gte: new Date(startDate),
            $lte: new Date(endDate),
          },
          status: "completed",
        },
      },

      // Stage 2: Group by date and calculate daily totals
      {
        $group: {
          _id: {
            $dateToString: {
              format: "%Y-%m-%d",
              date: "$createdAt",
            },
          },
          totalSales: { $sum: "$total" },
          numberOfOrders: { $sum: 1 },
          averageOrderValue: { $avg: "$total" },
        },
      },

      // Stage 3: Sort by date
      {
        $sort: { _id: 1 },
      },
    ];

    return await Order.aggregate(pipeline);
  }
}
```

### Advanced Aggregation Patterns

Let's explore more sophisticated aggregation operations:

```javascript
class OrderAnalytics {
  // Complex aggregation for product performance analysis
  async getProductPerformance(period = "30d") {
    const dateRange = this.calculateDateRange(period);

    const pipeline = [
      // Stage 1: Unwind order items
      {
        $unwind: "$items",
      },

      // Stage 2: Match relevant time period
      {
        $match: {
          createdAt: {
            $gte: dateRange.startDate,
            $lte: dateRange.endDate,
          },
        },
      },

      // Stage 3: Group by product
      {
        $group: {
          _id: "$items.productId",
          totalQuantitySold: { $sum: "$items.quantity" },
          totalRevenue: {
            $sum: {
              $multiply: ["$items.price", "$items.quantity"],
            },
          },
          numberOfOrders: { $sum: 1 },
          averageOrderSize: { $avg: "$items.quantity" },
        },
      },

      // Stage 4: Lookup product details
      {
        $lookup: {
          from: "products",
          localField: "_id",
          foreignField: "_id",
          as: "product",
        },
      },

      // Stage 5: Unwind product array from lookup
      {
        $unwind: "$product",
      },

      // Stage 6: Project final shape
      {
        $project: {
          productName: "$product.name",
          category: "$product.category",
          totalQuantitySold: 1,
          totalRevenue: 1,
          numberOfOrders: 1,
          averageOrderSize: 1,
          revenuePerUnit: {
            $divide: ["$totalRevenue", "$totalQuantitySold"],
          },
        },
      },

      // Stage 7: Sort by revenue
      {
        $sort: { totalRevenue: -1 },
      },
    ];

    return await Order.aggregate(pipeline);
  }

  // Aggregation for customer segmentation
  async customerSegmentation() {
    const pipeline = [
      // Stage 1: Group by customer
      {
        $group: {
          _id: "$customerId",
          totalSpent: { $sum: "$total" },
          numberOfOrders: { $sum: 1 },
          averageOrderValue: { $avg: "$total" },
          firstPurchase: { $min: "$createdAt" },
          lastPurchase: { $max: "$createdAt" },
        },
      },

      // Stage 2: Add computed fields
      {
        $addFields: {
          daysSinceLastPurchase: {
            $divide: [
              { $subtract: [new Date(), "$lastPurchase"] },
              1000 * 60 * 60 * 24, // Convert to days
            ],
          },
          customerLifespan: {
            $divide: [
              { $subtract: ["$lastPurchase", "$firstPurchase"] },
              1000 * 60 * 60 * 24, // Convert to days
            ],
          },
        },
      },

      // Stage 3: Calculate customer segment
      {
        $addFields: {
          segment: {
            $switch: {
              branches: [
                {
                  case: {
                    $and: [
                      { $gte: ["$totalSpent", 1000] },
                      { $lte: ["$daysSinceLastPurchase", 30] },
                    ],
                  },
                  then: "VIP",
                },
                {
                  case: {
                    $and: [
                      { $gte: ["$totalSpent", 500] },
                      { $lte: ["$daysSinceLastPurchase", 90] },
                    ],
                  },
                  then: "Regular",
                },
                {
                  case: { $gt: ["$daysSinceLastPurchase", 180] },
                  then: "At Risk",
                },
              ],
              default: "New",
            },
          },
        },
      },

      // Stage 4: Group by segment
      {
        $group: {
          _id: "$segment",
          customerCount: { $sum: 1 },
          totalRevenue: { $sum: "$totalSpent" },
          averageOrderValue: { $avg: "$averageOrderValue" },
        },
      },
    ];

    return await Order.aggregate(pipeline);
  }

  // Time series analysis
  async getRevenueTimeSeries(interval = "day") {
    const groupByDate = {
      day: {
        year: { $year: "$createdAt" },
        month: { $month: "$createdAt" },
        day: { $dayOfMonth: "$createdAt" },
      },
      week: {
        year: { $year: "$createdAt" },
        week: { $week: "$createdAt" },
      },
      month: {
        year: { $year: "$createdAt" },
        month: { $month: "$createdAt" },
      },
    };

    const pipeline = [
      // Stage 1: Group by time interval
      {
        $group: {
          _id: groupByDate[interval],
          revenue: { $sum: "$total" },
          orders: { $sum: 1 },
          uniqueCustomers: {
            $addToSet: "$customerId",
          },
        },
      },

      // Stage 2: Add computed metrics
      {
        $addFields: {
          averageOrderValue: {
            $divide: ["$revenue", "$orders"],
          },
          uniqueCustomerCount: {
            $size: "$uniqueCustomers",
          },
        },
      },

      // Stage 3: Project date format
      {
        $project: {
          date: this.getDateFormat(interval),
          revenue: 1,
          orders: 1,
          averageOrderValue: 1,
          uniqueCustomers: "$uniqueCustomerCount",
        },
      },

      // Stage 4: Sort by date
      {
        $sort: { date: 1 },
      },
    ];

    return await Order.aggregate(pipeline);
  }

  // Helper function for date formatting
  getDateFormat(interval) {
    switch (interval) {
      case "day":
        return {
          $dateFromParts: {
            year: "$_id.year",
            month: "$_id.month",
            day: "$_id.day",
          },
        };
      case "week":
        return {
          $dateFromParts: {
            isoWeekYear: "$_id.year",
            isoWeek: "$_id.week",
          },
        };
      case "month":
        return {
          $dateFromParts: {
            year: "$_id.year",
            month: "$_id.month",
          },
        };
    }
  }
}
```

### Optimizing Aggregation Performance

```javascript
class AggregationOptimizer {
  constructor(model) {
    this.model = model;
  }

  // Add index hints to pipeline
  addIndexHints(pipeline, indexes) {
    if (pipeline[0]?.$match) {
      return [
        {
          $hint: indexes,
        },
        ...pipeline,
      ];
    }
    return pipeline;
  }

  // Use allowDiskUse for large datasets
  async safeAggregate(pipeline, options = {}) {
    const defaultOptions = {
      allowDiskUse: true,
      maxTimeMS: 60000, // 1 minute timeout
    };

    try {
      return await this.model.aggregate(pipeline, {
        ...defaultOptions,
        ...options,
      });
    } catch (error) {
      if (error.code === 16389) {
        // timeoutError
        throw new Error("Aggregation timeout exceeded");
      }
      throw error;
    }
  }

  // Analyze aggregation performance
  async explainAggregation(pipeline) {
    const explanation = await this.model
      .aggregate(pipeline)
      .explain("executionStats");

    return {
      executionTimeMillis: explanation.executionStats.executionTimeMillis,
      totalDocsExamined: explanation.executionStats.totalDocsExamined,
      totalKeysExamined: explanation.executionStats.totalKeysExamined,
      inMemorySort: explanation.executionStats.totalDocsExamined > 100000,
      indexesUsed:
        explanation.queryPlanner.winningPlan.inputStage.inputStage?.indexName,
    };
  }
}
```

This aggregation framework provides:

- Complex data analysis
- Performance optimization
- Flexible data transformation
- Time series analysis
- Customer segmentation
- Product analytics

Would you like me to explain any of these concepts in more detail?
