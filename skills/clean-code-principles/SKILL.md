---
name: clean-code-principles
description: SOLID principles, design patterns, DRY, KISS, and clean code fundamentals. Use when reviewing architecture, checking code quality, refactoring, or discussing design decisions. Triggers on "review architecture", "check code quality", "SOLID principles", "design patterns", or "clean code".
license: MIT
metadata:
  author: agent-skills
  version: "1.0.0"
---

# Clean Code Principles

Fundamental software design principles, SOLID, design patterns, and clean code practices. Language-agnostic guidelines for writing maintainable, scalable software.

## When to Apply

Reference these guidelines when:
- Designing new features or systems
- Reviewing code architecture
- Refactoring existing code
- Discussing design decisions
- Improving code quality

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | SOLID Principles | CRITICAL | `solid-` |
| 2 | Core Principles | CRITICAL | `core-` |
| 3 | Design Patterns | HIGH | `pattern-` |
| 4 | Code Organization | HIGH | `org-` |
| 5 | Naming & Readability | MEDIUM | `name-` |
| 6 | Functions & Methods | MEDIUM | `func-` |
| 7 | Comments & Documentation | LOW | `doc-` |

## Quick Reference

### 1. SOLID Principles (CRITICAL)

- `solid-srp` - Single Responsibility Principle
- `solid-ocp` - Open/Closed Principle
- `solid-lsp` - Liskov Substitution Principle
- `solid-isp` - Interface Segregation Principle
- `solid-dip` - Dependency Inversion Principle

### 2. Core Principles (CRITICAL)

- `core-dry` - Don't Repeat Yourself
- `core-kiss` - Keep It Simple, Stupid
- `core-yagni` - You Aren't Gonna Need It
- `core-separation-of-concerns` - Separate different responsibilities
- `core-composition-over-inheritance` - Favor composition
- `core-law-of-demeter` - Principle of least knowledge
- `core-fail-fast` - Detect and report errors early
- `core-encapsulation` - Hide implementation details

### 3. Design Patterns (HIGH)

- `pattern-factory` - Factory pattern for object creation
- `pattern-strategy` - Strategy pattern for algorithms
- `pattern-repository` - Repository pattern for data access
- `pattern-decorator` - Decorator pattern for behavior extension
- `pattern-observer` - Observer pattern for event handling
- `pattern-adapter` - Adapter pattern for interface conversion
- `pattern-facade` - Facade pattern for simplified interfaces
- `pattern-dependency-injection` - DI for loose coupling

### 4. Code Organization (HIGH)

- `org-feature-folders` - Organize by feature, not layer
- `org-module-boundaries` - Clear module boundaries
- `org-layered-architecture` - Proper layer separation
- `org-package-cohesion` - Related code together
- `org-circular-dependencies` - Avoid circular imports

### 5. Naming & Readability (MEDIUM)

- `name-meaningful` - Use intention-revealing names
- `name-consistent` - Consistent naming conventions
- `name-searchable` - Avoid magic numbers/strings
- `name-avoid-encodings` - No Hungarian notation
- `name-domain-language` - Use domain terminology

### 6. Functions & Methods (MEDIUM)

- `func-small` - Keep functions small
- `func-single-purpose` - Do one thing
- `func-few-arguments` - Limit parameters
- `func-no-side-effects` - Minimize side effects
- `func-command-query` - Separate commands and queries

### 7. Comments & Documentation (LOW)

- `doc-self-documenting` - Code should explain itself
- `doc-why-not-what` - Explain why, not what
- `doc-avoid-noise` - No redundant comments
- `doc-api-docs` - Document public APIs

## Essential Guidelines

### SOLID Principles

#### Single Responsibility Principle (SRP)

```
A class should have only one reason to change.
```

```typescript
// ❌ Multiple responsibilities
class UserService {
  createUser(data) { /* validation, creation, email, logging */ }
  generateReport() { /* reporting logic */ }
  exportToCsv() { /* export logic */ }
}

// ✅ Single responsibility each
class UserService {
  constructor(
    private validator: UserValidator,
    private repository: UserRepository,
    private notifier: NotificationService,
  ) {}

  createUser(data) {
    this.validator.validate(data);
    const user = this.repository.create(data);
    this.notifier.sendWelcome(user);
    return user;
  }
}
```

#### Open/Closed Principle (OCP)

```
Software entities should be open for extension, closed for modification.
```

```typescript
// ❌ Requires modification to add new types
function calculateArea(shape) {
  if (shape.type === 'circle') {
    return Math.PI * shape.radius ** 2;
  } else if (shape.type === 'rectangle') {
    return shape.width * shape.height;
  }
  // Adding triangle requires modifying this function
}

// ✅ Open for extension via new classes
interface Shape {
  area(): number;
}

class Circle implements Shape {
  constructor(private radius: number) {}
  area() { return Math.PI * this.radius ** 2; }
}

class Rectangle implements Shape {
  constructor(private width: number, private height: number) {}
  area() { return this.width * this.height; }
}

// Adding Triangle doesn't modify existing code
class Triangle implements Shape {
  constructor(private base: number, private height: number) {}
  area() { return 0.5 * this.base * this.height; }
}
```

#### Liskov Substitution Principle (LSP)

```
Subtypes must be substitutable for their base types.
```

```typescript
// ❌ Violates LSP - Square changes Rectangle behavior
class Rectangle {
  constructor(protected width: number, protected height: number) {}
  setWidth(w: number) { this.width = w; }
  setHeight(h: number) { this.height = h; }
  area() { return this.width * this.height; }
}

class Square extends Rectangle {
  setWidth(w: number) { this.width = this.height = w; } // Breaks expectation
  setHeight(h: number) { this.width = this.height = h; }
}

// ✅ Proper abstraction
interface Shape {
  area(): number;
}

class Rectangle implements Shape {
  constructor(private width: number, private height: number) {}
  area() { return this.width * this.height; }
}

class Square implements Shape {
  constructor(private side: number) {}
  area() { return this.side ** 2; }
}
```

#### Interface Segregation Principle (ISP)

```
Clients should not be forced to depend on interfaces they don't use.
```

```typescript
// ❌ Fat interface
interface Worker {
  work(): void;
  eat(): void;
  sleep(): void;
}

class Robot implements Worker {
  work() { /* ... */ }
  eat() { throw new Error('Robots dont eat'); } // Forced to implement
  sleep() { throw new Error('Robots dont sleep'); }
}

// ✅ Segregated interfaces
interface Workable {
  work(): void;
}

interface Feedable {
  eat(): void;
}

interface Restable {
  sleep(): void;
}

class Human implements Workable, Feedable, Restable {
  work() { /* ... */ }
  eat() { /* ... */ }
  sleep() { /* ... */ }
}

class Robot implements Workable {
  work() { /* ... */ }
}
```

#### Dependency Inversion Principle (DIP)

```
Depend on abstractions, not concretions.
```

```typescript
// ❌ Depends on concrete implementation
class OrderService {
  private repository = new MySQLOrderRepository(); // Hard dependency

  createOrder(data) {
    return this.repository.save(data);
  }
}

// ✅ Depends on abstraction
interface OrderRepository {
  save(order: Order): Order;
  find(id: string): Order | null;
}

class OrderService {
  constructor(private repository: OrderRepository) {} // Injected abstraction

  createOrder(data) {
    return this.repository.save(data);
  }
}

// Can swap implementations
new OrderService(new MySQLOrderRepository());
new OrderService(new MongoOrderRepository());
new OrderService(new InMemoryOrderRepository()); // For testing
```

### Core Principles

#### DRY - Don't Repeat Yourself

```typescript
// ❌ Duplicated logic
function validateEmail(email: string) {
  const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return regex.test(email);
}

function isValidUserEmail(user: User) {
  const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/; // Duplicated
  return regex.test(user.email);
}

// ✅ Single source of truth
const EMAIL_REGEX = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

function isValidEmail(email: string): boolean {
  return EMAIL_REGEX.test(email);
}

function validateUser(user: User) {
  return isValidEmail(user.email);
}
```

#### KISS - Keep It Simple, Stupid

```typescript
// ❌ Over-engineered
class ConfigurationManagerFactoryBuilderProvider {
  private static instance: ConfigurationManagerFactoryBuilderProvider;
  private configFactoryBuilder: ConfigFactoryBuilder;

  static getInstance() {
    if (!this.instance) {
      this.instance = new ConfigurationManagerFactoryBuilderProvider();
    }
    return this.instance;
  }

  createConfigManager() {
    return this.configFactoryBuilder.build().create();
  }
}

// ✅ Simple and direct
const config = {
  apiUrl: process.env.API_URL,
  timeout: parseInt(process.env.TIMEOUT || '5000'),
};

export default config;
```

#### YAGNI - You Aren't Gonna Need It

```typescript
// ❌ Premature abstraction
interface DataExporter {
  exportToCsv(): string;
  exportToJson(): string;
  exportToXml(): string;      // Not needed yet
  exportToYaml(): string;     // Not needed yet
  exportToPdf(): string;      // Not needed yet
  exportToExcel(): string;    // Not needed yet
}

// ✅ Build only what's needed now
interface DataExporter {
  exportToCsv(): string;
  exportToJson(): string;
}
// Add more methods when actually required
```

#### Composition Over Inheritance

```typescript
// ❌ Deep inheritance hierarchy
class Animal { }
class Mammal extends Animal { }
class Dog extends Mammal { }
class FlyingDog extends Dog { } // Dogs can't fly, awkward

// ✅ Composition with behaviors
interface CanWalk {
  walk(): void;
}

interface CanFly {
  fly(): void;
}

interface CanSwim {
  swim(): void;
}

class Dog implements CanWalk {
  walk() { console.log('Walking'); }
}

class Duck implements CanWalk, CanFly, CanSwim {
  walk() { console.log('Walking'); }
  fly() { console.log('Flying'); }
  swim() { console.log('Swimming'); }
}
```

### Design Patterns

#### Repository Pattern

```typescript
// ✅ Repository abstracts data access
interface UserRepository {
  find(id: string): Promise<User | null>;
  findAll(): Promise<User[]>;
  save(user: User): Promise<User>;
  delete(id: string): Promise<void>;
}

class PostgresUserRepository implements UserRepository {
  async find(id: string) {
    const result = await this.db.query('SELECT * FROM users WHERE id = $1', [id]);
    return result.rows[0] ? this.mapToUser(result.rows[0]) : null;
  }
  // ...
}

class InMemoryUserRepository implements UserRepository {
  private users: Map<string, User> = new Map();

  async find(id: string) {
    return this.users.get(id) ?? null;
  }
  // ... great for testing
}
```

#### Strategy Pattern

```typescript
// ✅ Encapsulate algorithms
interface PaymentStrategy {
  pay(amount: number): Promise<PaymentResult>;
}

class CreditCardPayment implements PaymentStrategy {
  async pay(amount: number) {
    // Credit card processing
  }
}

class PayPalPayment implements PaymentStrategy {
  async pay(amount: number) {
    // PayPal processing
  }
}

class PaymentProcessor {
  constructor(private strategy: PaymentStrategy) {}

  async processPayment(amount: number) {
    return this.strategy.pay(amount);
  }
}

// Usage - swap strategies at runtime
const processor = new PaymentProcessor(new CreditCardPayment());
await processor.processPayment(100);
```

#### Factory Pattern

```typescript
// ✅ Encapsulate object creation
interface Notification {
  send(message: string): Promise<void>;
}

class NotificationFactory {
  static create(type: 'email' | 'sms' | 'push'): Notification {
    switch (type) {
      case 'email':
        return new EmailNotification();
      case 'sms':
        return new SmsNotification();
      case 'push':
        return new PushNotification();
      default:
        throw new Error(`Unknown notification type: ${type}`);
    }
  }
}

// Usage
const notification = NotificationFactory.create('email');
await notification.send('Hello!');
```

### Functions & Methods

#### Small Functions

```typescript
// ❌ Long function doing many things
function processOrder(order) {
  // Validate (20 lines)
  // Calculate totals (30 lines)
  // Apply discounts (25 lines)
  // Process payment (40 lines)
  // Send notifications (15 lines)
  // Update inventory (20 lines)
}

// ✅ Small, focused functions
function processOrder(order) {
  validateOrder(order);
  const totals = calculateTotals(order);
  const finalAmount = applyDiscounts(totals, order.customer);
  await processPayment(order, finalAmount);
  await sendOrderConfirmation(order);
  await updateInventory(order.items);
}
```

#### Meaningful Names

```typescript
// ❌ Cryptic names
function calc(a, b, c) {
  return a * b * (1 - c);
}

const d = calc(100, 5, 0.1);

// ✅ Intention-revealing names
function calculateDiscountedTotal(
  unitPrice: number,
  quantity: number,
  discountRate: number
): number {
  return unitPrice * quantity * (1 - discountRate);
}

const orderTotal = calculateDiscountedTotal(100, 5, 0.1);
```

#### Avoid Magic Numbers

```typescript
// ❌ Magic numbers
if (user.age >= 18) { }
if (password.length < 8) { }
if (items.length > 100) { }

// ✅ Named constants
const MINIMUM_AGE = 18;
const MINIMUM_PASSWORD_LENGTH = 8;
const MAX_ITEMS_PER_PAGE = 100;

if (user.age >= MINIMUM_AGE) { }
if (password.length < MINIMUM_PASSWORD_LENGTH) { }
if (items.length > MAX_ITEMS_PER_PAGE) { }
```

## Output Format

When auditing code, output findings in this format:

```
file:line - [principle] Description of issue
```

Example:
```
src/services/UserService.ts:15 - [solid-srp] Class handles validation, persistence, and notifications
src/utils/helpers.ts:42 - [core-dry] Email validation duplicated from validators/email.ts
src/models/Order.ts:28 - [name-meaningful] Variable 'x' should describe its purpose
```

## How to Use

Read individual rule files for detailed explanations:

```
rules/solid-srp.md
rules/core-dry.md
rules/pattern-repository.md
```
