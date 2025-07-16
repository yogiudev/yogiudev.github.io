<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# MongoDB Aggregation Queries: Complete Guidebook

## What is MongoDB Aggregation?

MongoDB aggregation is a powerful framework for processing and transforming documents in a collection. Think of it as a **pipeline** where data flows through various stages, with each stage performing a specific operation on the data. Unlike simple queries that just filter and return documents, aggregation allows you to group, sort, reshape, and compute values across your entire dataset.

## The Aggregation Pipeline Concept

The aggregation pipeline consists of **stages** that process documents sequentially. Each stage:

- Takes documents as input
- Performs an operation (filter, group, sort, etc.)
- Passes the result to the next stage

```javascript
db.collection.aggregate([
  { $stage1: { /* operations */ } },
  { $stage2: { /* operations */ } },
  { $stage3: { /* operations */ } }
])
```


## Core Aggregation Stages

### 1. \$match - Filtering Documents

The `$match` stage filters documents, similar to the `find()` method. It should be placed **early** in the pipeline for better performance.

```javascript
// Basic filtering
db.sales.aggregate([
  { $match: { status: "completed" } }
])

// Complex filtering with multiple conditions
db.sales.aggregate([
  { $match: { 
    status: "completed",
    amount: { $gte: 100 },
    date: { $gte: ISODate("2023-01-01") }
  }}
])
```


### 2. \$project - Reshaping Documents

The `$project` stage selects, excludes, or transforms fields in documents.

```javascript
// Select specific fields
db.customers.aggregate([
  { $project: { name: 1, email: 1, _id: 0 } }
])

// Create computed fields
db.sales.aggregate([
  { $project: { 
    product: 1,
    total: { $multiply: ["$price", "$quantity"] },
    discountedPrice: { $subtract: ["$price", "$discount"] }
  }}
])

// Rename fields
db.users.aggregate([
  { $project: { 
    fullName: "$name",
    userEmail: "$email",
    registrationDate: "$createdAt"
  }}
])
```


### 3. \$group - Grouping and Aggregating

The `$group` stage groups documents by specified fields and performs calculations.

```javascript
// Group by category and count
db.products.aggregate([
  { $group: { 
    _id: "$category",
    count: { $sum: 1 },
    averagePrice: { $avg: "$price" }
  }}
])

// Group by multiple fields
db.sales.aggregate([
  { $group: { 
    _id: { 
      year: { $year: "$date" },
      month: { $month: "$date" }
    },
    totalSales: { $sum: "$amount" },
    orderCount: { $sum: 1 }
  }}
])

// Group all documents (no grouping field)
db.orders.aggregate([
  { $group: { 
    _id: null,
    totalRevenue: { $sum: "$total" },
    averageOrderValue: { $avg: "$total" },
    maxOrder: { $max: "$total" }
  }}
])
```


### 4. \$sort - Ordering Results

The `$sort` stage orders documents by specified fields.

```javascript
// Sort by single field
db.products.aggregate([
  { $sort: { price: -1 } }  // -1 for descending, 1 for ascending
])

// Sort by multiple fields
db.customers.aggregate([
  { $sort: { 
    lastName: 1,
    firstName: 1,
    age: -1
  }}
])
```


### 5. \$limit and \$skip - Pagination

Control the number of documents returned and implement pagination.

```javascript
// Get top 5 products by price
db.products.aggregate([
  { $sort: { price: -1 } },
  { $limit: 5 }
])

// Skip first 10 documents, then take 5 (pagination)
db.products.aggregate([
  { $sort: { price: -1 } },
  { $skip: 10 },
  { $limit: 5 }
])
```


## Advanced Aggregation Stages

### 6. \$unwind - Deconstructing Arrays

The `$unwind` stage creates separate documents for each element in an array field.

```javascript
// Basic unwind
db.orders.aggregate([
  { $unwind: "$items" }
])

// Unwind with additional options
db.orders.aggregate([
  { $unwind: { 
    path: "$items",
    includeArrayIndex: "itemIndex",
    preserveNullAndEmptyArrays: true
  }}
])
```


### 7. \$lookup - Joining Collections

The `$lookup` stage performs left outer joins with other collections.

```javascript
// Basic lookup
db.orders.aggregate([
  { $lookup: {
    from: "customers",
    localField: "customerId",
    foreignField: "_id",
    as: "customerInfo"
  }}
])

// Lookup with pipeline (more complex joins)
db.orders.aggregate([
  { $lookup: {
    from: "products",
    let: { orderItems: "$items" },
    pipeline: [
      { $match: { $expr: { $in: ["$_id", "$$orderItems.productId"] } } },
      { $project: { name: 1, price: 1 } }
    ],
    as: "productDetails"
  }}
])
```


### 8. \$addFields - Adding New Fields

The `$addFields` stage adds new fields to documents without removing existing ones.

```javascript
db.students.aggregate([
  { $addFields: {
    totalScore: { $add: ["$math", "$science", "$english"] },
    fullName: { $concat: ["$firstName", " ", "$lastName"] },
    isHighPerformer: { $gte: ["$averageScore", 85] }
  }}
])
```


## Powerful Aggregation Operators

### Arithmetic Operators

```javascript
// Mathematical operations
db.sales.aggregate([
  { $project: {
    total: { $multiply: ["$price", "$quantity"] },
    discountAmount: { $multiply: ["$total", "$discountPercent"] },
    finalPrice: { $subtract: ["$total", "$discountAmount"] },
    pricePerUnit: { $divide: ["$total", "$quantity"] }
  }}
])
```


### String Operators

```javascript
// String manipulation
db.users.aggregate([
  { $project: {
    fullName: { $concat: ["$firstName", " ", "$lastName"] },
    emailDomain: { $substr: ["$email", { $add: [{ $indexOfCP: ["$email", "@"] }, 1] }, -1] },
    upperCaseName: { $toUpper: "$name" },
    nameLength: { $strLenCP: "$name" }
  }}
])
```


### Date Operators

```javascript
// Date operations
db.orders.aggregate([
  { $project: {
    year: { $year: "$orderDate" },
    month: { $month: "$orderDate" },
    dayOfWeek: { $dayOfWeek: "$orderDate" },
    formattedDate: { $dateToString: { 
      format: "%Y-%m-%d", 
      date: "$orderDate" 
    }}
  }}
])
```


### Conditional Operators

```javascript
// Conditional logic
db.products.aggregate([
  { $project: {
    name: 1,
    price: 1,
    priceCategory: {
      $switch: {
        branches: [
          { case: { $lt: ["$price", 50] }, then: "Budget" },
          { case: { $lt: ["$price", 200] }, then: "Mid-range" },
          { case: { $gte: ["$price", 200] }, then: "Premium" }
        ],
        default: "Unknown"
      }
    },
    isOnSale: { $cond: { if: { $gt: ["$discount", 0] }, then: true, else: false } }
  }}
])
```


## Complex Aggregation Examples

### Example 1: Sales Analytics Dashboard

```javascript
// Monthly sales report with trends
db.sales.aggregate([
  // Filter last 12 months
  { $match: { 
    date: { $gte: new Date(new Date().setMonth(new Date().getMonth() - 12)) }
  }},
  
  // Group by month and region
  { $group: {
    _id: {
      year: { $year: "$date" },
      month: { $month: "$date" },
      region: "$region"
    },
    totalSales: { $sum: "$amount" },
    orderCount: { $sum: 1 },
    averageOrderValue: { $avg: "$amount" }
  }},
  
  // Add computed fields
  { $addFields: {
    monthName: {
      $switch: {
        branches: [
          { case: { $eq: ["$_id.month", 1] }, then: "January" },
          { case: { $eq: ["$_id.month", 2] }, then: "February" },
          // ... continue for all months
        ]
      }
    }
  }},
  
  // Sort by year and month
  { $sort: { "_id.year": 1, "_id.month": 1 } }
])
```


### Example 2: Customer Segmentation

```javascript
// Customer lifetime value and segmentation
db.customers.aggregate([
  // Join with orders
  { $lookup: {
    from: "orders",
    localField: "_id",
    foreignField: "customerId",
    as: "orders"
  }},
  
  // Add customer metrics
  { $addFields: {
    totalSpent: { $sum: "$orders.total" },
    orderCount: { $size: "$orders" },
    firstOrderDate: { $min: "$orders.date" },
    lastOrderDate: { $max: "$orders.date" }
  }},
  
  // Calculate customer lifetime value
  { $addFields: {
    averageOrderValue: { $divide: ["$totalSpent", "$orderCount"] },
    customerSegment: {
      $switch: {
        branches: [
          { case: { $and: [{ $gte: ["$totalSpent", 1000] }, { $gte: ["$orderCount", 5] }] }, then: "VIP" },
          { case: { $and: [{ $gte: ["$totalSpent", 500] }, { $gte: ["$orderCount", 3] }] }, then: "Premium" },
          { case: { $gte: ["$totalSpent", 100] }, then: "Regular" }
        ],
        default: "New"
      }
    }
  }},
  
  // Group by segment for analysis
  { $group: {
    _id: "$customerSegment",
    customerCount: { $sum: 1 },
    totalRevenue: { $sum: "$totalSpent" },
    averageLifetimeValue: { $avg: "$totalSpent" }
  }}
])
```


## Advanced Tips and Tricks

### 1. Performance Optimization

**Use indexes effectively:**

```javascript
// Create compound index for common aggregation patterns
db.sales.createIndex({ "date": 1, "region": 1, "status": 1 })

// Place $match early in pipeline
db.sales.aggregate([
  { $match: { status: "completed" } },  // Filter first
  { $group: { _id: "$region", total: { $sum: "$amount" } } }
])
```

**Limit data early:**

```javascript
// Good: Filter before processing
db.products.aggregate([
  { $match: { category: "electronics" } },
  { $sort: { price: -1 } },
  { $limit: 10 }
])
```


### 2. Working with Arrays

**Array operations:**

```javascript
db.orders.aggregate([
  { $project: {
    itemCount: { $size: "$items" },
    hasMultipleItems: { $gt: [{ $size: "$items" }, 1] },
    firstItem: { $arrayElemAt: ["$items", 0] },
    expensiveItems: { $filter: {
      input: "$items",
      as: "item",
      cond: { $gt: ["$$item.price", 100] }
    }}
  }}
])
```


### 3. Data Type Conversions

```javascript
// Convert string to number, handle errors
db.products.aggregate([
  { $addFields: {
    priceAsNumber: {
      $convert: {
        input: "$price",
        to: "double",
        onError: 0,
        onNull: 0
      }
    }
  }}
])
```


### 4. Handling Missing Data

```javascript
// Deal with null/missing values
db.users.aggregate([
  { $project: {
    name: 1,
    age: { $ifNull: ["$age", 0] },
    isActive: { $ifNull: ["$isActive", false] },
    status: { $cond: { 
      if: { $ne: ["$lastLogin", null] }, 
      then: "active", 
      else: "inactive" 
    }}
  }}
])
```


## Advanced Transformation Techniques

### 1. Document Reshaping

```javascript
// Transform document structure
db.orders.aggregate([
  { $project: {
    orderInfo: {
      id: "$_id",
      date: "$orderDate",
      status: "$status"
    },
    customer: {
      name: "$customerName",
      email: "$customerEmail"
    },
    summary: {
      itemCount: { $size: "$items" },
      total: "$total"
    }
  }}
])
```


### 2. Creating Hierarchical Data

```javascript
// Build category hierarchy
db.products.aggregate([
  { $group: {
    _id: "$category",
    products: { $push: {
      name: "$name",
      price: "$price",
      inStock: "$inStock"
    }},
    totalProducts: { $sum: 1 },
    averagePrice: { $avg: "$price" }
  }},
  { $project: {
    category: "$_id",
    _id: 0,
    products: 1,
    totalProducts: 1,
    averagePrice: { $round: ["$averagePrice", 2] }
  }}
])
```


### 3. Time Series Analysis

```javascript
// Daily sales trend with moving average
db.sales.aggregate([
  { $match: { date: { $gte: new Date("2023-01-01") } } },
  { $group: {
    _id: { $dateToString: { format: "%Y-%m-%d", date: "$date" } },
    dailyTotal: { $sum: "$amount" },
    orderCount: { $sum: 1 }
  }},
  { $sort: { "_id": 1 } },
  { $setWindowFields: {
    sortBy: { "_id": 1 },
    output: {
      movingAverage: {
        $avg: "$dailyTotal",
        window: { documents: [-6, 0] }  // 7-day moving average
      }
    }
  }}
])
```


## Common Patterns and Use Cases

### 1. Top N Analysis

```javascript
// Top 5 customers by revenue
db.customers.aggregate([
  { $lookup: {
    from: "orders",
    localField: "_id",
    foreignField: "customerId",
    as: "orders"
  }},
  { $project: {
    name: 1,
    totalSpent: { $sum: "$orders.total" }
  }},
  { $sort: { totalSpent: -1 } },
  { $limit: 5 }
])
```


### 2. Data Validation and Cleaning

```javascript
// Find and fix data inconsistencies
db.products.aggregate([
  { $project: {
    name: 1,
    price: 1,
    issues: {
      $concatArrays: [
        { $cond: [{ $lte: ["$price", 0] }, ["Invalid price"], []] },
        { $cond: [{ $eq: ["$name", ""] }, ["Missing name"], []] },
        { $cond: [{ $eq: ["$category", null] }, ["Missing category"], []] }
      ]
    }
  }},
  { $match: { issues: { $ne: [] } } }
])
```


### 3. Faceted Search

```javascript
// Build faceted search results
db.products.aggregate([
  { $match: { $text: { $search: "smartphone" } } },
  { $facet: {
    products: [
      { $limit: 20 },
      { $project: { name: 1, price: 1, rating: 1 } }
    ],
    priceRanges: [
      { $bucket: {
        groupBy: "$price",
        boundaries: [0, 100, 300, 500, 1000],
        default: "1000+",
        output: { count: { $sum: 1 } }
      }}
    ],
    brands: [
      { $group: { _id: "$brand", count: { $sum: 1 } } },
      { $sort: { count: -1 } }
    ]
  }}
])
```


## Best Practices

### 1. Pipeline Design

- **Filter early**: Use `$match` as early as possible
- **Reduce data size**: Use `$project` to include only necessary fields
- **Index support**: Ensure your `$match` and `$sort` stages can use indexes


### 2. Memory Management

- Each stage has a 100MB memory limit
- Use `allowDiskUse: true` for large datasets
- Consider breaking complex pipelines into smaller ones


### 3. Error Handling

- Use `$ifNull` and `$cond` for null checks
- Implement data validation in your pipeline
- Test with various data scenarios


### 4. Documentation

- Comment complex aggregations
- Use meaningful variable names in `$let` expressions
- Document expected input/output formats

This comprehensive guide covers the essential aspects of MongoDB aggregation queries. Practice these patterns with your own data, and gradually build more complex pipelines as you become comfortable with the basics. Remember that aggregation is powerful but can impact performance, so always test and optimize your queries for your specific use case.

