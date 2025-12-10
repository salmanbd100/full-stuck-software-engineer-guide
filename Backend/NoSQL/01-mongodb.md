# MongoDB Fundamentals

## Overview

MongoDB is a document-oriented NoSQL database that stores data in flexible JSON-like documents (BSON format). It's widely used in Node.js applications for its flexibility and scalability.

## 🎯 Core Concepts

**Document**: JSON-like record stored in BSON format
```javascript
{
  _id: ObjectId("507f1f77bcf86cd799439011"),
  name: "John Doe",
  email: "john@example.com",
  age: 30,
  address: {
    street: "123 Main St",
    city: "NYC",
    zip: "10001"
  },
  hobbies: ["reading", "coding"],
  createdAt: ISODate("2024-01-01T00:00:00Z")
}
```

**Collection**: Group of documents (similar to SQL table)
**Database**: Container for collections
**ObjectId**: 12-byte unique identifier for documents

---

## 💻 CRUD Operations with Mongoose

### Setup

```javascript
const mongoose = require('mongoose');

// Connect to MongoDB
mongoose.connect('mongodb://localhost:27017/myapp', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
});

// Define Schema
const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    trim: true,
    minlength: 2,
    maxlength: 50,
  },
  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true,
  },
  age: {
    type: Number,
    min: 0,
    max: 120,
  },
  role: {
    type: String,
    enum: ['user', 'admin', 'moderator'],
    default: 'user',
  },
  isActive: {
    type: Boolean,
    default: true,
  },
  createdAt: {
    type: Date,
    default: Date.now,
  },
}, {
  timestamps: true, // Adds createdAt and updatedAt automatically
});

// Create Model
const User = mongoose.model('User', userSchema);
```

### Create (Insert)

```javascript
// Insert one document
const user = new User({
  name: 'John Doe',
  email: 'john@example.com',
  age: 30,
});

await user.save();

// Or using create()
const user = await User.create({
  name: 'John Doe',
  email: 'john@example.com',
});

// Insert many
await User.insertMany([
  { name: 'Alice', email: 'alice@example.com' },
  { name: 'Bob', email: 'bob@example.com' },
]);
```

### Read (Find)

```javascript
// Find all
const users = await User.find();

// Find with conditions
const adults = await User.find({ age: { $gte: 18 } });

// Find one
const user = await User.findOne({ email: 'john@example.com' });

// Find by ID
const user = await User.findById('507f1f77bcf86cd799439011');

// With projection (select specific fields)
const users = await User.find({}, 'name email'); // Only name and email
const users = await User.find().select('name email -_id'); // Exclude _id

// With sorting
const users = await User.find().sort({ age: -1 }); // Descending by age

// With pagination
const page = 2;
const limit = 10;
const users = await User.find()
  .skip((page - 1) * limit)
  .limit(limit);

// Chaining
const users = await User.find({ isActive: true })
  .select('name email')
  .sort({ createdAt: -1 })
  .limit(10);
```

### Update

```javascript
// Update one
await User.updateOne(
  { email: 'john@example.com' },
  { $set: { age: 31 } }
);

// Update many
await User.updateMany(
  { role: 'user' },
  { $set: { isActive: true } }
);

// Find and update (returns updated document)
const user = await User.findByIdAndUpdate(
  userId,
  { age: 31 },
  { new: true, runValidators: true } // Return updated doc & validate
);

// Find one and update
const user = await User.findOneAndUpdate(
  { email: 'john@example.com' },
  { $inc: { age: 1 } }, // Increment age by 1
  { new: true }
);
```

### Delete

```javascript
// Delete one
await User.deleteOne({ _id: userId });

// Delete many
await User.deleteMany({ isActive: false });

// Find and delete (returns deleted document)
const user = await User.findByIdAndDelete(userId);
const user = await User.findOneAndDelete({ email: 'john@example.com' });
```

---

## 🔍 Query Operators

### Comparison Operators

```javascript
// $eq - Equal
{ age: { $eq: 25 } }
{ age: 25 } // Shorthand

// $ne - Not equal
{ age: { $ne: 25 } }

// $gt, $gte - Greater than, Greater than or equal
{ age: { $gt: 18 } }
{ age: { $gte: 18 } }

// $lt, $lte - Less than, Less than or equal
{ age: { $lt: 65 } }

// $in - Match any value in array
{ role: { $in: ['admin', 'moderator'] } }

// $nin - Not in array
{ status: { $nin: ['deleted', 'banned'] } }
```

### Logical Operators

```javascript
// $and - All conditions must match
await User.find({
  $and: [
    { age: { $gte: 18 } },
    { role: 'admin' }
  ]
});

// Implicit $and (more common)
await User.find({ age: { $gte: 18 }, role: 'admin' });

// $or - At least one condition must match
await User.find({
  $or: [
    { role: 'admin' },
    { role: 'moderator' }
  ]
});

// $nor - None of the conditions match
await User.find({
  $nor: [
    { isActive: false },
    { isDeleted: true }
  ]
});

// $not - Negation
await User.find({ age: { $not: { $lt: 18 } } });
```

### Element Operators

```javascript
// $exists - Field exists
await User.find({ phone: { $exists: true } });

// $type - Field type
await User.find({ age: { $type: 'number' } });
```

### Array Operators

```javascript
// Contains element
await User.find({ hobbies: 'reading' });

// $all - Contains all elements
await User.find({ hobbies: { $all: ['reading', 'coding'] } });

// $size - Array size
await User.find({ hobbies: { $size: 3 } });

// $elemMatch - At least one element matches all conditions
await User.find({
  orders: {
    $elemMatch: {
      status: 'completed',
      total: { $gt: 100 }
    }
  }
});
```

### String Operators

```javascript
// $regex - Regular expression
await User.find({ name: { $regex: /^John/i } }); // Case-insensitive starts with "John"

// Text search (requires text index)
await User.find({ $text: { $search: 'john doe' } });
```

---

## 📊 Aggregation Pipeline

Aggregation is used for complex data processing and analysis.

```javascript
// Basic aggregation
const result = await User.aggregate([
  // Stage 1: Match (filter)
  { $match: { isActive: true } },

  // Stage 2: Group
  { $group: {
      _id: '$role',
      count: { $sum: 1 },
      averageAge: { $avg: '$age' }
    }
  },

  // Stage 3: Sort
  { $sort: { count: -1 } }
]);

// Example result:
// [
//   { _id: 'user', count: 150, averageAge: 28.5 },
//   { _id: 'admin', count: 10, averageAge: 35.2 }
// ]
```

### Common Aggregation Stages

```javascript
// $match - Filter documents
{ $match: { age: { $gte: 18 } } }

// $group - Group by field and aggregate
{ $group: {
    _id: '$department',
    totalSalary: { $sum: '$salary' },
    avgSalary: { $avg: '$salary' },
    minSalary: { $min: '$salary' },
    maxSalary: { $max: '$salary' },
    count: { $sum: 1 }
  }
}

// $project - Select/transform fields
{ $project: {
    name: 1,
    email: 1,
    fullName: { $concat: ['$firstName', ' ', '$lastName'] },
    _id: 0
  }
}

// $sort - Sort documents
{ $sort: { age: -1, name: 1 } }

// $limit - Limit number of documents
{ $limit: 10 }

// $skip - Skip documents (pagination)
{ $skip: 20 }

// $lookup - Join with another collection (like SQL JOIN)
{ $lookup: {
    from: 'posts',
    localField: '_id',
    foreignField: 'authorId',
    as: 'posts'
  }
}

// $unwind - Deconstruct array field
{ $unwind: '$hobbies' }

// $addFields - Add new fields
{ $addFields: {
    fullName: { $concat: ['$firstName', ' ', '$lastName'] }
  }
}
```

### Real-World Example

```javascript
// Get top 5 users by post count
const topUsers = await User.aggregate([
  // Join with posts collection
  {
    $lookup: {
      from: 'posts',
      localField: '_id',
      foreignField: 'authorId',
      as: 'posts'
    }
  },
  // Add post count field
  {
    $addFields: {
      postCount: { $size: '$posts' }
    }
  },
  // Filter active users
  {
    $match: { isActive: true }
  },
  // Sort by post count descending
  {
    $sort: { postCount: -1 }
  },
  // Limit to top 5
  {
    $limit: 5
  },
  // Select only needed fields
  {
    $project: {
      name: 1,
      email: 1,
      postCount: 1
    }
  }
]);
```

---

## 🔑 Indexing for Performance

Indexes improve query performance significantly.

```javascript
// Create index on single field
userSchema.index({ email: 1 }); // 1 for ascending, -1 for descending

// Compound index (multiple fields)
userSchema.index({ role: 1, createdAt: -1 });

// Unique index (enforces uniqueness)
userSchema.index({ email: 1 }, { unique: true });

// Text index (for full-text search)
userSchema.index({ name: 'text', bio: 'text' });

// TTL index (auto-delete after expiry)
userSchema.index({ createdAt: 1 }, { expireAfterSeconds: 3600 });

// Sparse index (only index documents that have the field)
userSchema.index({ phone: 1 }, { sparse: true });

// Check indexes
await User.collection.getIndexes();

// Explain query (check if index is used)
await User.find({ email: 'john@example.com' }).explain('executionStats');
```

**Best Practices:**
- Index fields used in `find()`, `sort()`, `group by`
- Limit number of indexes (each index slows down writes)
- Use compound indexes for multi-field queries
- Monitor query performance with `.explain()`

---

## 🔄 Relationships in MongoDB

### 1. Embedded Documents (Denormalization)

```javascript
// One-to-Few: User with addresses
const userSchema = new mongoose.Schema({
  name: String,
  addresses: [{
    street: String,
    city: String,
    zip: String
  }]
});

// Pros: Fast reads, single query
// Cons: Document size limit (16MB), data duplication
```

### 2. References (Normalization)

```javascript
// One-to-Many: User and Posts
const userSchema = new mongoose.Schema({
  name: String,
  email: String,
});

const postSchema = new mongoose.Schema({
  title: String,
  content: String,
  authorId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  }
});

// Query with populate
const posts = await Post.find()
  .populate('authorId', 'name email');
// Returns posts with full author details

// Pros: No duplication, flexible
// Cons: Requires multiple queries (or lookup)
```

### 3. Hybrid Approach

```javascript
// Store frequently accessed data embedded, reference for full details
const postSchema = new mongoose.Schema({
  title: String,
  author: {
    id: mongoose.Schema.Types.ObjectId,
    name: String, // Embedded for quick access
    email: String
  }
});

// Update both when author changes (trade-off for read performance)
```

---

## 🔒 Transactions (ACID)

MongoDB supports multi-document ACID transactions (v4.0+).

```javascript
const session = await mongoose.startSession();
session.startTransaction();

try {
  // Transfer money between accounts
  await Account.updateOne(
    { _id: fromAccountId },
    { $inc: { balance: -amount } },
    { session }
  );

  await Account.updateOne(
    { _id: toAccountId },
    { $inc: { balance: amount } },
    { session }
  );

  // Commit transaction
  await session.commitTransaction();
} catch (error) {
  // Rollback on error
  await session.abortTransaction();
  throw error;
} finally {
  session.endSession();
}
```

---

## 📚 Interview Questions

### Q1: SQL vs MongoDB - When to use each?

**Answer:**

**Use SQL (PostgreSQL, MySQL) when:**
- Complex joins and relationships
- ACID transactions critical (banking, finance)
- Structured, predictable schema
- Mature ecosystem and tools needed

**Use MongoDB when:**
- Flexible/evolving schema
- Hierarchical/nested data (JSON-like)
- Horizontal scaling needed
- Rapid prototyping
- Document-based data model fits naturally

**Example:** E-commerce product catalog → MongoDB (flexible attributes per category). Financial transactions → SQL (strict ACID compliance).

---

### Q2: What is BSON and why does MongoDB use it?

**Answer:**

**BSON (Binary JSON)** is MongoDB's data format.

**Advantages over JSON:**
1. **More data types:** Date, Binary, ObjectId, Decimal128, Int64
2. **Efficient:** Binary format, faster to parse and traverse
3. **Size-friendly:** Stores length prefix for quick skipping

```javascript
// JSON - Limited types
{ "date": "2024-12-08T10:00:00Z" } // String, not Date

// BSON - Native types
{ "date": ISODate("2024-12-08T10:00:00Z") } // Actual Date object
{ "_id": ObjectId("507f1f77bcf86cd799439011") } // 12-byte ObjectId
```

---

### Q3: Explain MongoDB indexing and its importance.

**Answer:**

**Indexes** are data structures that improve query performance.

**Without Index:**
```javascript
// Scans all 1,000,000 documents (COLLSCAN)
db.users.find({ email: 'john@example.com' })
// Time: ~500ms
```

**With Index:**
```javascript
db.users.createIndex({ email: 1 })
db.users.find({ email: 'john@example.com' })
// Time: ~5ms (100x faster!)
```

**Types:**
- **Single Field:** `{ email: 1 }`
- **Compound:** `{ role: 1, createdAt: -1 }`
- **Text:** Full-text search
- **Geospatial:** Location queries
- **TTL:** Auto-delete old documents

**Trade-offs:**
- ✅ Faster reads
- ❌ Slower writes (must update index)
- ❌ Uses disk space

---

### Q4: How do you handle N+1 query problem in MongoDB?

**Answer:**

**Problem:**
```javascript
// Get users and their posts (N+1 queries)
const users = await User.find(); // 1 query

for (const user of users) {
  user.posts = await Post.find({ authorId: user._id }); // N queries
}
// Total: 1 + N queries (slow for large N)
```

**Solution 1: populate() with Mongoose**
```javascript
const users = await User.find().populate('posts');
// 2 queries total (efficient)
```

**Solution 2: Aggregation $lookup**
```javascript
const users = await User.aggregate([
  {
    $lookup: {
      from: 'posts',
      localField: '_id',
      foreignField: 'authorId',
      as: 'posts'
    }
  }
]);
// Single aggregation query
```

**Solution 3: Embed data (denormalization)**
```javascript
// Store post count in user document
userSchema.add({ postCount: Number });
// No additional query needed
```

---

### Q5: Embedded vs Referenced documents - which to choose?

**Answer:**

**Embedded (Denormalized):**
```javascript
// User with embedded addresses
{
  name: "John",
  addresses: [
    { street: "123 Main", city: "NYC" },
    { street: "456 Oak", city: "LA" }
  ]
}
```

**When to use:**
- One-to-few relationships
- Data accessed together
- Read performance critical
- Data doesn't change often

**Referenced (Normalized):**
```javascript
// User
{ _id: 1, name: "John" }

// Posts (separate collection)
{ title: "Post 1", authorId: 1 }
{ title: "Post 2", authorId: 1 }
```

**When to use:**
- One-to-many/many-to-many
- Data changes frequently
- Need to query independently
- Document size concerns

**Rule of thumb:**
- Embed if "part of" relationship (address is part of user)
- Reference if "has a" relationship (user has posts)

---

## ✅ Best Practices

1. **Use indexes** on frequently queried fields
2. **Limit document size** to 16MB max (usually < 1MB)
3. **Avoid deep nesting** (max 100 levels, but stay < 5)
4. **Use projections** to return only needed fields
5. **Implement pagination** for large result sets
6. **Use aggregation** for complex queries
7. **Enable schema validation** for data integrity
8. **Monitor query performance** with `.explain()`
9. **Use connection pooling** for better performance
10. **Handle errors** properly (duplicate keys, validation, etc.)

---

[← Back to Backend](../README.md) | [Next: Design Patterns →](./02-design-patterns.md)
