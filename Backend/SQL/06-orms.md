# ORMs (Object-Relational Mapping)

## 💡 **Overview**

ORMs (Object-Relational Mappers) bridge the gap between object-oriented programming and relational databases.

**What You'll Learn:**
- ORM fundamentals and how they work
- TypeORM (TypeScript) comprehensive guide
- Sequelize (JavaScript) comprehensive guide
- When to use ORMs vs raw SQL
- Common patterns and best practices

**Why This Matters:**
- ORMs abstract database complexity, letting you work with objects instead of SQL
- Provide type safety and validation
- Handle database migrations and schema synchronization
- Tested frequently in backend interviews

---

## 💡 **What is an ORM?**

An ORM maps database tables to classes/objects, allowing you to interact with databases using your programming language instead of SQL.

**How It Works:**

```
JavaScript/TypeScript Code  ←→  ORM  ←→  Database
     (Objects)                         (Tables)
```

**Without ORM (Raw SQL):**

```javascript
// Manual SQL query
const result = await db.query(
  'SELECT * FROM users WHERE email = ? AND age > ?',
  [email, 18]
);
const users = result.rows;  // Manual parsing

// Manual insert
await db.query(
  'INSERT INTO users (name, email, age) VALUES (?, ?, ?)',
  [name, email, age]
);
```

**With ORM:**

```javascript
// Object-oriented query
const users = await User.findAll({
  where: {
    email: email,
    age: { [Op.gt]: 18 }
  }
});

// Object creation
const user = await User.create({ name, email, age });
```

**Key Benefits:**

| Benefit | Description |
|---------|-------------|
| **Type Safety** | Compile-time checks for TypeScript ORMs |
| **SQL Injection Prevention** | Automatic parameter escaping |
| **Database Agnostic** | Switch databases with minimal code changes |
| **Less Boilerplate** | No manual SQL writing for basic operations |
| **Migrations** | Version control for database schema |
| **Relationships** | Easy handling of table relationships |

---

## 💡 **TypeORM** (TypeScript)

TypeORM is a powerful ORM for TypeScript and JavaScript that supports Active Record and Data Mapper patterns.

### **Setup and Configuration**

**Installation:**

```bash
npm install typeorm reflect-metadata @types/node
npm install pg        # PostgreSQL
# OR
npm install mysql2    # MySQL
```

**Configuration (ormconfig.json):**

```json
{
  "type": "postgres",
  "host": "localhost",
  "port": 5432,
  "username": "postgres",
  "password": "password",
  "database": "mydb",
  "synchronize": false,
  "logging": true,
  "entities": ["src/entities/**/*.ts"],
  "migrations": ["src/migrations/**/*.ts"],
  "subscribers": ["src/subscribers/**/*.ts"]
}
```

**Data Source Setup (TypeScript):**

```typescript
import { DataSource } from 'typeorm';

export const AppDataSource = new DataSource({
  type: 'postgres',
  host: 'localhost',
  port: 5432,
  username: 'postgres',
  password: 'password',
  database: 'mydb',
  synchronize: false,  // Never true in production!
  logging: true,
  entities: ['src/entities/**/*.ts'],
  migrations: ['src/migrations/**/*.ts']
});

// Initialize
AppDataSource.initialize()
  .then(() => console.log('Database connected'))
  .catch((error) => console.error('Database error:', error));
```

---

### **Entities (Models)**

Entities represent database tables as TypeScript classes.

**Basic Entity:**

```typescript
import { Entity, PrimaryGeneratedColumn, Column, CreateDateColumn, UpdateDateColumn } from 'typeorm';

@Entity('users')
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ unique: true })
  email: string;

  @Column()
  name: string;

  @Column({ type: 'int', default: 0 })
  age: number;

  @Column({ type: 'varchar', length: 20, default: 'active' })
  status: string;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}
```

**Column Types:**

| TypeORM Type | Database Type | Example |
|--------------|---------------|---------|
| `@Column()` | VARCHAR | `name: string` |
| `@Column('text')` | TEXT | `description: string` |
| `@Column('int')` | INTEGER | `age: number` |
| `@Column('decimal')` | DECIMAL | `price: number` |
| `@Column('boolean')` | BOOLEAN | `isActive: boolean` |
| `@Column('jsonb')` | JSONB | `metadata: object` |

**Column Options:**

```typescript
@Column({
  type: 'varchar',
  length: 100,
  unique: true,
  nullable: false,
  default: 'default value',
  select: true  // Include in SELECT by default
})
email: string;
```

---

### **CRUD Operations with TypeORM**

**Create (Insert):**

```typescript
import { AppDataSource } from './data-source';
import { User } from './entities/User';

const userRepository = AppDataSource.getRepository(User);

// Method 1: Using create + save
const user = userRepository.create({
  email: 'john@example.com',
  name: 'John Doe',
  age: 30
});
await userRepository.save(user);

// Method 2: Using save directly
const user = await userRepository.save({
  email: 'john@example.com',
  name: 'John Doe',
  age: 30
});

// Method 3: Using insert (returns ID only)
const result = await userRepository.insert({
  email: 'john@example.com',
  name: 'John Doe',
  age: 30
});
console.log(result.identifiers[0].id);
```

**Read (Select):**

```typescript
// Find all
const users = await userRepository.find();

// Find one by ID
const user = await userRepository.findOne({
  where: { id: 1 }
});

// Find one by email
const user = await userRepository.findOne({
  where: { email: 'john@example.com' }
});

// Find with conditions
const users = await userRepository.find({
  where: {
    age: 30,
    status: 'active'
  }
});

// Find with operators (MoreThan, LessThan, Like, etc.)
import { MoreThan, Like } from 'typeorm';

const users = await userRepository.find({
  where: {
    age: MoreThan(18),
    name: Like('%John%')
  }
});

// Find with select specific columns
const users = await userRepository.find({
  select: ['id', 'name', 'email']
});

// Find with order
const users = await userRepository.find({
  order: {
    createdAt: 'DESC',
    name: 'ASC'
  }
});

// Find with pagination
const users = await userRepository.find({
  skip: 20,  // OFFSET
  take: 10   // LIMIT
});

// Count
const count = await userRepository.count();
const activeCount = await userRepository.count({
  where: { status: 'active' }
});
```

**Update:**

```typescript
// Method 1: Load, modify, save
const user = await userRepository.findOne({ where: { id: 1 } });
user.name = 'Jane Doe';
user.age = 31;
await userRepository.save(user);

// Method 2: Using update (more efficient)
await userRepository.update(
  { id: 1 },
  { name: 'Jane Doe', age: 31 }
);

// Update multiple rows
await userRepository.update(
  { status: 'inactive' },
  { status: 'active' }
);

// Increment/Decrement
await userRepository.increment({ id: 1 }, 'age', 1);  // age = age + 1
await userRepository.decrement({ id: 1 }, 'age', 1);  // age = age - 1
```

**Delete:**

```typescript
// Method 1: Load and remove
const user = await userRepository.findOne({ where: { id: 1 } });
await userRepository.remove(user);

// Method 2: Using delete (more efficient)
await userRepository.delete({ id: 1 });

// Delete multiple
await userRepository.delete({ status: 'inactive' });

// Soft delete (if @DeleteDateColumn is defined)
await userRepository.softDelete({ id: 1 });
```

---

### **Relations in TypeORM**

**One-to-One:**

```typescript
// User entity
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  email: string;

  @OneToOne(() => Profile, profile => profile.user, { cascade: true })
  @JoinColumn()
  profile: Profile;
}

// Profile entity
@Entity()
export class Profile {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  bio: string;

  @Column()
  avatar: string;

  @OneToOne(() => User, user => user.profile)
  user: User;
}

// Usage
const user = await userRepository.findOne({
  where: { id: 1 },
  relations: ['profile']
});
```

**One-to-Many / Many-to-One:**

```typescript
// Author entity (one author, many posts)
@Entity()
export class Author {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => Post, post => post.author)
  posts: Post[];
}

// Post entity
@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @ManyToOne(() => Author, author => author.posts)
  @JoinColumn({ name: 'author_id' })
  author: Author;
}

// Usage
const author = await authorRepository.findOne({
  where: { id: 1 },
  relations: ['posts']
});

console.log(author.posts);  // All posts by this author
```

**Many-to-Many:**

```typescript
// Student entity
@Entity()
export class Student {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @ManyToMany(() => Course, course => course.students)
  @JoinTable({
    name: 'student_courses',  // Junction table name
    joinColumn: { name: 'student_id', referencedColumnName: 'id' },
    inverseJoinColumn: { name: 'course_id', referencedColumnName: 'id' }
  })
  courses: Course[];
}

// Course entity
@Entity()
export class Course {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @ManyToMany(() => Student, student => student.courses)
  students: Student[];
}

// Usage
const student = await studentRepository.findOne({
  where: { id: 1 },
  relations: ['courses']
});

console.log(student.courses);  // All courses for this student
```

---

### **Query Builder (Advanced Queries)**

For complex queries, use QueryBuilder:

```typescript
// Basic query builder
const users = await userRepository
  .createQueryBuilder('user')
  .where('user.age > :age', { age: 18 })
  .andWhere('user.status = :status', { status: 'active' })
  .orderBy('user.createdAt', 'DESC')
  .getMany();

// JOIN query
const users = await userRepository
  .createQueryBuilder('user')
  .leftJoinAndSelect('user.posts', 'post')
  .where('user.id = :id', { id: 1 })
  .getOne();

// Complex JOIN with conditions
const users = await userRepository
  .createQueryBuilder('user')
  .leftJoinAndSelect('user.posts', 'post', 'post.published = :published', { published: true })
  .where('user.age > :age', { age: 18 })
  .orderBy('user.name', 'ASC')
  .addOrderBy('post.createdAt', 'DESC')
  .getMany();

// Aggregations
const result = await userRepository
  .createQueryBuilder('user')
  .select('COUNT(user.id)', 'count')
  .addSelect('AVG(user.age)', 'avgAge')
  .where('user.status = :status', { status: 'active' })
  .getRawOne();

// Pagination with count
const [users, total] = await userRepository
  .createQueryBuilder('user')
  .where('user.status = :status', { status: 'active' })
  .skip(20)
  .take(10)
  .getManyAndCount();
```

---

## 💡 **Sequelize** (JavaScript)

Sequelize is a promise-based ORM for Node.js supporting PostgreSQL, MySQL, SQLite, and more.

### **Setup and Configuration**

**Installation:**

```bash
npm install sequelize
npm install pg pg-hstore    # PostgreSQL
# OR
npm install mysql2          # MySQL
```

**Configuration:**

```javascript
const { Sequelize } = require('sequelize');

const sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: 'postgres',  // or 'mysql', 'sqlite', 'mssql'
  logging: console.log,
  pool: {
    max: 5,
    min: 0,
    acquire: 30000,
    idle: 10000
  }
});

// Test connection
async function testConnection() {
  try {
    await sequelize.authenticate();
    console.log('Database connected');
  } catch (error) {
    console.error('Database error:', error);
  }
}

testConnection();
```

---

### **Models (Entities)**

**Define Model:**

```javascript
const { DataTypes } = require('sequelize');

const User = sequelize.define('User', {
  id: {
    type: DataTypes.INTEGER,
    primaryKey: true,
    autoIncrement: true
  },
  email: {
    type: DataTypes.STRING(100),
    unique: true,
    allowNull: false,
    validate: {
      isEmail: true
    }
  },
  name: {
    type: DataTypes.STRING,
    allowNull: false
  },
  age: {
    type: DataTypes.INTEGER,
    validate: {
      min: 0,
      max: 120
    }
  },
  status: {
    type: DataTypes.ENUM('active', 'inactive', 'banned'),
    defaultValue: 'active'
  }
}, {
  tableName: 'users',
  timestamps: true,  // Adds createdAt and updatedAt
  underscored: true  // Use snake_case for column names
});

module.exports = User;
```

**Data Types:**

| Sequelize Type | Database Type | Example |
|----------------|---------------|---------|
| `DataTypes.STRING` | VARCHAR(255) | `name: DataTypes.STRING` |
| `DataTypes.STRING(100)` | VARCHAR(100) | `email: DataTypes.STRING(100)` |
| `DataTypes.TEXT` | TEXT | `description: DataTypes.TEXT` |
| `DataTypes.INTEGER` | INTEGER | `age: DataTypes.INTEGER` |
| `DataTypes.FLOAT` | FLOAT | `rating: DataTypes.FLOAT` |
| `DataTypes.DECIMAL(10,2)` | DECIMAL(10,2) | `price: DataTypes.DECIMAL(10,2)` |
| `DataTypes.BOOLEAN` | BOOLEAN | `isActive: DataTypes.BOOLEAN` |
| `DataTypes.DATE` | TIMESTAMP | `createdAt: DataTypes.DATE` |
| `DataTypes.JSONB` | JSONB | `metadata: DataTypes.JSONB` |

---

### **CRUD Operations with Sequelize**

**Create (Insert):**

```javascript
// Create single record
const user = await User.create({
  email: 'john@example.com',
  name: 'John Doe',
  age: 30
});

// Bulk create
const users = await User.bulkCreate([
  { email: 'john@example.com', name: 'John', age: 30 },
  { email: 'jane@example.com', name: 'Jane', age: 25 }
]);

// Create with associated data
const user = await User.create({
  email: 'john@example.com',
  name: 'John',
  profile: {
    bio: 'Software engineer',
    avatar: 'avatar.jpg'
  }
}, {
  include: [Profile]  // Save associated profile
});
```

**Read (Select):**

```javascript
// Find all
const users = await User.findAll();

// Find by primary key
const user = await User.findByPk(1);

// Find one
const user = await User.findOne({
  where: { email: 'john@example.com' }
});

// Find with conditions
const users = await User.findAll({
  where: {
    age: 30,
    status: 'active'
  }
});

// Find with operators
const { Op } = require('sequelize');

const users = await User.findAll({
  where: {
    age: { [Op.gt]: 18 },
    name: { [Op.like]: '%John%' },
    status: { [Op.in]: ['active', 'pending'] }
  }
});

// Select specific columns
const users = await User.findAll({
  attributes: ['id', 'name', 'email']
});

// Exclude columns
const users = await User.findAll({
  attributes: { exclude: ['password'] }
});

// Order
const users = await User.findAll({
  order: [
    ['createdAt', 'DESC'],
    ['name', 'ASC']
  ]
});

// Pagination
const users = await User.findAll({
  limit: 10,
  offset: 20
});

// Find and count
const { count, rows } = await User.findAndCountAll({
  where: { status: 'active' },
  limit: 10,
  offset: 20
});
```

**Update:**

```javascript
// Update single record
const user = await User.findByPk(1);
user.name = 'Jane Doe';
user.age = 31;
await user.save();

// Update with update method
await User.update(
  { name: 'Jane Doe', age: 31 },
  { where: { id: 1 } }
);

// Update multiple records
await User.update(
  { status: 'active' },
  { where: { status: 'inactive' } }
);

// Increment/Decrement
await User.increment('age', { where: { id: 1 } });
await User.increment({ age: 1, loginCount: 1 }, { where: { id: 1 } });
await User.decrement('age', { where: { id: 1 } });
```

**Delete:**

```javascript
// Delete by instance
const user = await User.findByPk(1);
await user.destroy();

// Delete with destroy method
await User.destroy({
  where: { id: 1 }
});

// Delete multiple
await User.destroy({
  where: { status: 'inactive' }
});

// Delete all (dangerous!)
await User.destroy({ truncate: true });
```

---

### **Relations in Sequelize**

**One-to-One:**

```javascript
// Define models
const User = sequelize.define('User', { /* ... */ });
const Profile = sequelize.define('Profile', { /* ... */ });

// Define relationship
User.hasOne(Profile, { foreignKey: 'userId', as: 'profile' });
Profile.belongsTo(User, { foreignKey: 'userId', as: 'user' });

// Usage
const user = await User.findByPk(1, {
  include: [{ model: Profile, as: 'profile' }]
});
console.log(user.profile);
```

**One-to-Many / Many-to-One:**

```javascript
// Author has many Posts
const Author = sequelize.define('Author', { /* ... */ });
const Post = sequelize.define('Post', { /* ... */ });

Author.hasMany(Post, { foreignKey: 'authorId', as: 'posts' });
Post.belongsTo(Author, { foreignKey: 'authorId', as: 'author' });

// Usage
const author = await Author.findByPk(1, {
  include: [{ model: Post, as: 'posts' }]
});
console.log(author.posts);  // All posts by author
```

**Many-to-Many:**

```javascript
// Students and Courses
const Student = sequelize.define('Student', { /* ... */ });
const Course = sequelize.define('Course', { /* ... */ });

Student.belongsToMany(Course, { through: 'StudentCourses', as: 'courses' });
Course.belongsToMany(Student, { through: 'StudentCourses', as: 'students' });

// Usage
const student = await Student.findByPk(1, {
  include: [{ model: Course, as: 'courses' }]
});
console.log(student.courses);
```

---

## 💡 **ORM vs Raw SQL**

### **When to Use ORM**

**✅ Use ORM For:**

| Scenario | Why ORM is Better |
|----------|-------------------|
| **CRUD Operations** | Less boilerplate, type safety |
| **Simple Queries** | Cleaner syntax, easier to maintain |
| **Multiple Databases** | Database agnostic code |
| **Rapid Development** | Faster development with less code |
| **Type Safety** | Compile-time checks (TypeScript) |
| **Migrations** | Built-in schema versioning |

**Examples:**

```typescript
// ✅ ORM: Clean and simple
const users = await User.find({ where: { age: { [Op.gt]: 18 } } });

// ❌ Raw SQL: More verbose
const result = await db.query('SELECT * FROM users WHERE age > $1', [18]);
const users = result.rows;
```

### **When to Use Raw SQL**

**✅ Use Raw SQL For:**

| Scenario | Why Raw SQL is Better |
|----------|----------------------|
| **Complex Queries** | Complex JOINs, subqueries, CTEs |
| **Performance Critical** | Optimized queries without ORM overhead |
| **Bulk Operations** | Large inserts, updates, deletes |
| **Database-Specific Features** | Window functions, full-text search |
| **Legacy Systems** | Existing stored procedures |

**Examples:**

```sql
-- ✅ Raw SQL: Complex query with CTE
WITH top_authors AS (
  SELECT author_id, COUNT(*) as post_count
  FROM posts
  WHERE published_at > NOW() - INTERVAL '30 days'
  GROUP BY author_id
  HAVING COUNT(*) > 10
)
SELECT u.*, ta.post_count
FROM users u
INNER JOIN top_authors ta ON u.id = ta.author_id
ORDER BY ta.post_count DESC;

-- ❌ ORM: Very complex to express with ORM
```

### **Hybrid Approach**

Use both ORM and raw SQL strategically:

```typescript
// ORM for simple operations
const user = await User.findByPk(1);

// Raw SQL for complex analytics
const stats = await sequelize.query(`
  WITH monthly_revenue AS (
    SELECT
      DATE_TRUNC('month', order_date) as month,
      SUM(total) as revenue
    FROM orders
    GROUP BY month
  )
  SELECT
    month,
    revenue,
    LAG(revenue) OVER (ORDER BY month) as prev_revenue,
    (revenue - LAG(revenue) OVER (ORDER BY month)) / LAG(revenue) OVER (ORDER BY month) * 100 as growth_pct
  FROM monthly_revenue
  ORDER BY month DESC
`, { type: QueryTypes.SELECT });
```

---

## 📚 **Common Interview Questions**

### Q1: What is an ORM and why use it?

**Answer:**

An ORM (Object-Relational Mapping) maps database tables to objects, letting you interact with databases using programming language constructs instead of SQL.

**Benefits:**
- **Type Safety**: Compile-time error checking (TypeScript)
- **SQL Injection Prevention**: Automatic parameter escaping
- **Less Boilerplate**: No manual SQL for basic operations
- **Database Agnostic**: Switch databases easily
- **Migrations**: Schema version control
- **Productivity**: Faster development

**Trade-offs:**
- **Performance Overhead**: Slight performance cost vs raw SQL
- **Complex Queries**: Harder to express complex queries
- **Learning Curve**: Need to learn ORM syntax
- **Less Control**: Limited access to database-specific features

---

### Q2: What are the advantages and disadvantages of ORMs?

**Answer:**

**Advantages:**

| Advantage | Description |
|-----------|-------------|
| **Productivity** | Write less code for common operations |
| **Type Safety** | Catch errors at compile time (TypeScript) |
| **Security** | Prevents SQL injection automatically |
| **Maintainability** | Easier to refactor and maintain |
| **Portability** | Switch databases with minimal changes |
| **Relationships** | Easy handling of foreign keys and joins |

**Disadvantages:**

| Disadvantage | Description |
|--------------|-------------|
| **Performance** | Overhead compared to raw SQL |
| **Complexity** | Complex queries are harder to express |
| **Learning Curve** | Need to learn ORM API and patterns |
| **Control** | Less control over exact SQL generated |
| **Debugging** | Harder to debug generated SQL |
| **Features** | Limited access to database-specific features |

---

### Q3: When should you use raw SQL instead of an ORM?

**Answer:**

**Use Raw SQL When:**

1. **Complex Queries**: Multiple JOINs, CTEs, subqueries
   ```sql
   -- Hard to express with ORM
   WITH RECURSIVE org_chart AS (...)
   SELECT * FROM org_chart;
   ```

2. **Performance Critical**: Bulk operations, large datasets
   ```sql
   -- Faster with raw SQL
   INSERT INTO logs SELECT * FROM temp_logs WHERE date > '2024-01-01';
   ```

3. **Database-Specific Features**: Window functions, full-text search
   ```sql
   -- PostgreSQL specific
   SELECT *, ts_rank(search_vector, query) FROM articles
   WHERE search_vector @@ to_tsquery('postgres & performance');
   ```

4. **Legacy Systems**: Existing stored procedures
5. **Reporting/Analytics**: Complex aggregations

**Use ORM When:**
- ✅ CRUD operations
- ✅ Simple queries
- ✅ Rapid development
- ✅ Type safety needed

---

### Q4: How do you handle the N+1 query problem in ORMs?

**Answer:**

The N+1 problem occurs when you fetch N records, then make N additional queries for related data.

**❌ Problem:**

```typescript
// Bad: N+1 queries (1 to fetch users, N to fetch each user's posts)
const users = await User.findAll();  // 1 query
for (const user of users) {
  const posts = await Post.findAll({ where: { userId: user.id } });  // N queries
}
```

**✅ Solution 1: Eager Loading**

```typescript
// Good: 1 query with JOIN
const users = await User.findAll({
  include: [{ model: Post, as: 'posts' }]  // Single JOIN query
});
```

**✅ Solution 2: Batch Loading**

```typescript
// Good: 2 queries total (1 for users, 1 for all posts)
const users = await User.findAll();
const userIds = users.map(u => u.id);
const posts = await Post.findAll({
  where: { userId: { [Op.in]: userIds } }
});

// Group posts by userId
const postsByUser = posts.reduce((acc, post) => {
  acc[post.userId] = acc[post.userId] || [];
  acc[post.userId].push(post);
  return acc;
}, {});
```

**✅ Solution 3: DataLoader** (for GraphQL)

```javascript
const DataLoader = require('dataloader');

const postLoader = new DataLoader(async (userIds) => {
  const posts = await Post.findAll({
    where: { userId: { [Op.in]: userIds } }
  });

  // Group by userId
  const postsByUser = userIds.map(id =>
    posts.filter(p => p.userId === id)
  );

  return postsByUser;
});
```

---

### Q5: How do migrations work in ORMs?

**Answer:**

Migrations are version-controlled schema changes that can be applied and rolled back.

**TypeORM Migration:**

```typescript
import { MigrationInterface, QueryRunner, Table } from 'typeorm';

export class CreateUsersTable1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.createTable(
      new Table({
        name: 'users',
        columns: [
          { name: 'id', type: 'int', isPrimary: true, isGenerated: true },
          { name: 'email', type: 'varchar', isUnique: true },
          { name: 'name', type: 'varchar' }
        ]
      })
    );
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropTable('users');
  }
}
```

**Commands:**

```bash
# Generate migration
npm run typeorm migration:generate -- -n CreateUsersTable

# Run migrations
npm run typeorm migration:run

# Revert last migration
npm run typeorm migration:revert
```

**Why Migrations:**
- ✅ Version control for database schema
- ✅ Reproducible across environments
- ✅ Team collaboration
- ✅ Rollback capability

---

## ✅ **Best Practices**

### **Do's**

1. **✅ Use Eager Loading to Prevent N+1**
   ```typescript
   // Load related data in single query
   const users = await User.findAll({ include: ['posts'] });
   ```

2. **✅ Use Indexes on Foreign Keys**
   ```typescript
   @Index()
   @Column()
   userId: number;
   ```

3. **✅ Use Transactions for Multi-Step Operations**
   ```typescript
   await sequelize.transaction(async (t) => {
     await User.create({ name: 'John' }, { transaction: t });
     await Post.create({ userId: 1, title: 'Hello' }, { transaction: t });
   });
   ```

4. **✅ Validate Data at Model Level**
   ```typescript
   @Column()
   @Length(3, 100)
   @IsEmail()
   email: string;
   ```

5. **✅ Use Query Builders for Complex Queries**
   ```typescript
   const users = await userRepository
     .createQueryBuilder('user')
     .leftJoinAndSelect('user.posts', 'post')
     .where('user.age > :age', { age: 18 })
     .getMany();
   ```

### **Don'ts**

1. **❌ Don't Use `synchronize: true` in Production**
   ```typescript
   // Bad: Auto-syncs schema (can delete data!)
   synchronize: true  // Only for development!

   // Good: Use migrations
   synchronize: false
   ```

2. **❌ Don't Ignore N+1 Problems**
   ```typescript
   // Bad: N+1 queries
   const users = await User.findAll();
   for (const user of users) {
     const posts = await Post.findAll({ where: { userId: user.id } });
   }

   // Good: Eager load
   const users = await User.findAll({ include: ['posts'] });
   ```

3. **❌ Don't Load Unnecessary Relations**
   ```typescript
   // Bad: Loads all relations
   const user = await User.findOne({ relations: ['posts', 'comments', 'likes'] });

   // Good: Load only what you need
   const user = await User.findOne({ relations: ['posts'] });
   ```

4. **❌ Don't Bypass Validation**
   ```typescript
   // Bad: Skips validation
   await userRepository.insert({ email: 'invalid' });

   // Good: Uses validation
   const user = userRepository.create({ email: 'invalid' });
   await userRepository.save(user);  // Validates first
   ```

5. **❌ Don't Use ORM for Complex Analytics**
   ```typescript
   // Bad: Very complex with ORM
   const result = await userRepository.createQueryBuilder()...

   // Good: Use raw SQL for complex queries
   const result = await dataSource.query(`WITH...`);
   ```

---

## 🎯 **Summary**

**Key Concepts:**
- **ORMs** map database tables to objects, abstracting SQL complexity
- **TypeORM** provides TypeScript-first ORM with decorators and type safety
- **Sequelize** provides JavaScript-friendly ORM with promise-based API
- **Use ORM** for CRUD and simple queries; use raw SQL for complex operations
- **Eager loading** prevents N+1 query problems

**TypeORM vs Sequelize:**

| Feature | TypeORM | Sequelize |
|---------|---------|-----------|
| **Language** | TypeScript-first | JavaScript-friendly |
| **Syntax** | Decorators | Model definition |
| **Type Safety** | Excellent (TypeScript) | Good (with TypeScript) |
| **Learning Curve** | Moderate | Easy |
| **Community** | Growing | Large, mature |

**When to Use:**
- ✅ **ORM**: CRUD, simple queries, rapid development, type safety
- ✅ **Raw SQL**: Complex queries, performance-critical, bulk operations, analytics

**Best Practices:**
- Use eager loading to prevent N+1
- Never use `synchronize: true` in production
- Use migrations for schema changes
- Use transactions for multi-step operations
- Validate data at model level

---

[← Previous: PostgreSQL](./05-postgresql.md) | [Next: Migrations →](./07-migrations.md)
