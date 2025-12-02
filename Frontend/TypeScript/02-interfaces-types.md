# Interfaces & Types

## Concept

TypeScript provides two main ways to define object shapes: **interfaces** and **type aliases**. While they have significant overlap, each has unique capabilities and use cases.

### Key Points
- Interfaces can be extended and implemented
- Type aliases can represent unions, intersections, primitives
- Interfaces can be merged (declaration merging)
- Type aliases can use utility types and mapped types
- Both can describe object shapes

---

## Example 1: Interface Basics

**Basic Interface Definition** - Defines contracts for object shapes with properties, including optional and readonly modifiers.

```typescript
// Basic interface
interface User {
    id: number;
    name: string;
    email: string;
    age?: number; // Optional property
    readonly createdAt: Date; // Read-only property
}

const user: User = {
    id: 1,
    name: 'Alice',
    email: 'alice@example.com',
    createdAt: new Date()
};

// user.createdAt = new Date(); // Error: readonly property

**Interface with Methods** - Interfaces can define method signatures that implementing objects must provide.

```typescript
// Interface with methods
interface Calculator {
    add(a: number, b: number): number;
    subtract(a: number, b: number): number;
}

const calc: Calculator = {
    add: (a, b) => a + b,
    subtract: (a, b) => a - b
};
```

**Interface with Function Type** - Describes function signatures using call signature syntax.

```typescript
// Interface with function type
interface SearchFunc {
    (query: string, limit: number): string[];
}

const searchUsers: SearchFunc = (query, limit) => {
    // Implementation
    return [];
};
```

**Interface with Index Signature** - Allows objects to have dynamic properties with specified key and value types.

```typescript
// Interface with index signature
interface StringMap {
    [key: string]: string;
}

const colors: StringMap = {
    primary: '#007bff',
    secondary: '#6c757d'
};
```

**Extending Interfaces** - Interfaces can extend other interfaces to inherit their properties and add new ones.

```typescript
// Extending interfaces
interface Person {
    name: string;
    age: number;
}

interface Employee extends Person {
    employeeId: string;
    department: string;
}

const employee: Employee = {
    name: 'Bob',
    age: 30,
    employeeId: 'E123',
    department: 'Engineering'
};
```

**Multiple Inheritance** - Interfaces can extend multiple interfaces simultaneously using comma-separated list.

```typescript
// Multiple inheritance
interface Timestamped {
    createdAt: Date;
    updatedAt: Date;
}

interface Product extends Person, Timestamped {
    productId: string;
}
```

---

## Example 2: Type Aliases

**Basic Type Aliases** - Create reusable type definitions using the type keyword for primitives, unions, or complex types.

```typescript
// Basic type alias
type UserID = string | number;

type User = {
    id: UserID;
    name: string;
    email: string;
};
```

**Union Types** - Allow values to be one of several types using the | operator.

```typescript
// Union types
type Status = 'pending' | 'approved' | 'rejected';

type Result = Success | Error;

type Success = {
    success: true;
    data: any;
};

type Error = {
    success: false;
    error: string;
};
```

**Intersection Types** - Combine multiple types using the & operator to create types with all properties.

```typescript
// Intersection types
type Timestamped = {
    createdAt: Date;
    updatedAt: Date;
};

type Person = {
    name: string;
    age: number;
};

type TimestampedPerson = Person & Timestamped;

const person: TimestampedPerson = {
    name: 'Alice',
    age: 30,
    createdAt: new Date(),
    updatedAt: new Date()
};
```

**Function Types** - Define function signatures using type aliases for reusable function patterns.

```typescript
// Function types
type AddFunction = (a: number, b: number) => number;

const add: AddFunction = (a, b) => a + b;
```

**Type Aliases for Primitives** - Create descriptive names for primitive types and utility type combinations.

```typescript
// Type aliases for primitives
type ID = string;
type Callback = () => void;
type Nullable<T> = T | null;
```

**Tuple Types** - Define fixed-length arrays with specific types for each position.

```typescript
// Tuple types
type Point = [number, number];
type RGB = [number, number, number];

const point: Point = [10, 20];
const color: RGB = [255, 0, 0];
```

**Mapped Types** - Transform properties of existing types programmatically using key remapping.

```typescript
// Mapped types (only with type)
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
};

type ReadonlyUser = Readonly<User>;
```

---

## Example 3: Interface vs Type - When to Use

```typescript
// ✅ Use INTERFACE for:

// 1. Object shapes that might be extended
interface Animal {
    name: string;
    age: number;
}

interface Dog extends Animal {
    breed: string;
}

// 2. Declaration merging (useful for extending third-party types)
interface Window {
    customProperty: string;
}

interface Window {
    anotherProperty: number;
}
// Both declarations merge into single interface

// 3. When implementing in classes
interface Drawable {
    draw(): void;
}

class Circle implements Drawable {
    draw() {
        console.log('Drawing circle');
    }
}

// ✅ Use TYPE for:

// 1. Union types
type Result = Success | Failure;
type ID = string | number;

// 2. Intersection types
type Employee = Person & HasId & Timestamped;

// 3. Tuple types
type Coordinates = [number, number, number];

// 4. Utility types and mapped types
type Partial<T> = {
    [P in keyof T]?: T[P];
};

type PartialUser = Partial<User>;

// 5. Primitive aliases
type Email = string;
type Count = number;

// 6. Function types
type AsyncCallback = (error: Error | null, data: any) => Promise<void>;
```

---

## Common Pitfalls

### Pitfall 1: Extending vs Intersection

```typescript
// With interfaces - error if properties conflict
interface A {
    prop: string;
}

interface B extends A {
    // prop: number; // Error: Types conflict
}

// With types - creates never type if properties conflict
type C = {
    prop: string;
};

type D = C & {
    prop: number; // No error, but prop becomes never type
};

const d: D = {
    prop: 'test' as never // prop is effectively unusable
};
```

### Pitfall 2: Index Signatures

```typescript
// PROBLEM - Can't add specific properties with different types
interface Config {
    [key: string]: string; // All properties must be strings
    // timeout: number; // Error: number not assignable to string
}

// SOLUTION 1 - Make index signature more permissive
interface ConfigFixed {
    [key: string]: string | number;
    timeout: number; // OK now
}

// SOLUTION 2 - Use type with Record
type ConfigType = Record<string, string> & {
    timeout: number;
};
```

### Pitfall 3: Optional vs Undefined

```typescript
interface User {
    name: string;
    age?: number; // Can be omitted
}

type UserType = {
    name: string;
    age: number | undefined; // Must be present, can be undefined
};

const user1: User = {
    name: 'Alice'
    // age can be omitted
};

const user2: UserType = {
    name: 'Bob',
    age: undefined // Must explicitly set to undefined
};
```

---

## Best Practices

### 1. Prefer Interface for Object Shapes

```typescript
// Good - using interface for objects
interface Product {
    id: string;
    name: string;
    price: number;
}

// Good - using type for unions/primitives
type ProductStatus = 'inStock' | 'outOfStock' | 'discontinued';
type ProductID = string;
```

### 2. Use Descriptive Names

```typescript
// Bad
interface Data {
    x: number;
    y: string;
}

// Good
interface UserProfile {
    userId: number;
    username: string;
}
```

### 3. Composition Over Complex Types

```typescript
// Less maintainable
type ComplexUser = {
    id: string;
    name: string;
    email: string;
    address: {
        street: string;
        city: string;
        country: string;
    };
    preferences: {
        theme: 'light' | 'dark';
        notifications: boolean;
    };
};

// Better - compose from smaller types
interface Address {
    street: string;
    city: string;
    country: string;
}

interface UserPreferences {
    theme: 'light' | 'dark';
    notifications: boolean;
}

interface User {
    id: string;
    name: string;
    email: string;
    address: Address;
    preferences: UserPreferences;
}
```

---

## Real-world Scenarios

### Scenario 1: API Response Types

```typescript
// Base response interface
interface ApiResponse<T> {
    success: boolean;
    data?: T;
    error?: string;
    timestamp: Date;
}

// Specific response types
interface User {
    id: string;
    name: string;
    email: string;
}

interface Post {
    id: string;
    title: string;
    content: string;
    authorId: string;
}

// Usage
type UserResponse = ApiResponse<User>;
type PostsResponse = ApiResponse<Post[]>;

async function getUser(id: string): Promise<UserResponse> {
    const response = await fetch(`/api/users/${id}`);
    return response.json();
}
```

### Scenario 2: Form State Management

```typescript
interface FormField<T> {
    value: T;
    error?: string;
    touched: boolean;
    dirty: boolean;
}

interface LoginForm {
    email: FormField<string>;
    password: FormField<string>;
    rememberMe: FormField<boolean>;
}

const initialForm: LoginForm = {
    email: { value: '', touched: false, dirty: false },
    password: { value: '', touched: false, dirty: false },
    rememberMe: { value: false, touched: false, dirty: false }
};
```

### Scenario 3: Event Handlers

```typescript
// React event handler types
interface ButtonProps {
    onClick: (event: React.MouseEvent<HTMLButtonElement>) => void;
    onHover?: (event: React.MouseEvent<HTMLButtonElement>) => void;
}

interface InputProps {
    value: string;
    onChange: (event: React.ChangeEvent<HTMLInputElement>) => void;
    onBlur?: (event: React.FocusEvent<HTMLInputElement>) => void;
}

// Custom event types
type UserClickedEvent = {
    type: 'USER_CLICKED';
    payload: {
        userId: string;
        timestamp: Date;
    };
};

type UserLoggedInEvent = {
    type: 'USER_LOGGED_IN';
    payload: {
        userId: string;
        sessionId: string;
    };
};

type AppEvent = UserClickedEvent | UserLoggedInEvent;

function handleEvent(event: AppEvent) {
    switch (event.type) {
        case 'USER_CLICKED':
            console.log(event.payload.userId);
            break;
        case 'USER_LOGGED_IN':
            console.log(event.payload.sessionId);
            break;
    }
}
```

---

## External Resources

- [TypeScript Handbook: Interfaces](https://www.typescriptlang.org/docs/handbook/interfaces.html)
- [TypeScript Handbook: Type Aliases](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-aliases)
- [TypeScript Deep Dive: Interfaces](https://basarat.gitbook.io/typescript/type-system/interfaces)

---

[← Back to TypeScript](./README.md) | [Next: Generics →](./03-generics.md)
