---
name: "testing-strategies"
version: "1.0.0"
displayName: "Testing Strategies"
description: "Comprehensive testing patterns and best practices for unit, integration, and E2E testing"
keywords: ["testing", "unit tests", "integration", "e2e", "best practices"]
---

# Unit Testing Strategies

## Overview
Unit testing focuses on testing individual components in isolation. This guide provides patterns and best practices for writing effective, maintainable unit tests.

## When to Use Unit Testing
- Testing pure functions and business logic
- Validating component behavior in isolation
- Fast feedback during development
- Testing edge cases and error handling
- Ensuring code correctness at the smallest level

## Core Principles

### 1. Arrange-Act-Assert (AAA) Pattern
```javascript
test('should calculate total price correctly', () => {
  // Arrange: Set up test data
  const cart = { items: [
    { price: 10, quantity: 2 },
    { price: 5, quantity: 3 }
  ]};

  // Act: Execute the function
  const total = calculateTotal(cart);

  // Assert: Verify the result
  expect(total).toBe(35);
});
```

### 2. Test One Thing Per Test
```javascript
// ✅ Good: Single responsibility
test('should add item to cart', () => {
  const cart = createCart();
  cart.addItem({ id: 1, name: 'Widget' });
  expect(cart.items).toHaveLength(1);
});

test('should calculate cart total', () => {
  const cart = createCart();
  cart.addItem({ price: 10 });
  expect(cart.getTotal()).toBe(10);
});

// ❌ Bad: Testing multiple things
test('should add item and calculate total', () => {
  // Testing two concerns
});
```

### 3. Use Descriptive Test Names
```javascript
// ✅ Good: Clear what's being tested
test('should throw error when quantity is negative', () => {});
test('should apply 10% discount for premium users', () => {});
test('should return empty array when no results found', () => {});

// ❌ Bad: Vague
test('test1', () => {});
test('should work', () => {});
```

## Mocking Strategies

### Dependency Injection for Testability
```javascript
// ✅ Good: Injectable dependencies
class OrderService {
  constructor(private database, private emailService) {}

  async createOrder(order) {
    await this.database.save(order);
    await this.emailService.send(order.email);
  }
}

// Easy to test with mocks
test('should save order to database', () => {
  const mockDb = { save: jest.fn() };
  const mockEmail = { send: jest.fn() };
  const service = new OrderService(mockDb, mockEmail);

  service.createOrder({ id: 1, email: 'test@example.com' });

  expect(mockDb.save).toHaveBeenCalledWith({ id: 1, email: 'test@example.com' });
});
```

### Mocking External Dependencies
```javascript
// Mock HTTP requests
jest.mock('axios');
test('should fetch user data', async () => {
  axios.get.mockResolvedValue({ data: { name: 'John' } });
  const user = await getUserData(1);
  expect(user.name).toBe('John');
});

// Mock file system
jest.mock('fs');
test('should read config file', () => {
  fs.readFileSync.mockReturnValue('{"port": 3000}');
  const config = loadConfig();
  expect(config.port).toBe(3000);
});
```

## Test Organization

### Group Related Tests
```javascript
describe('ShoppingCart', () => {
  describe('addItem', () => {
    test('should add item to empty cart', () => {});
    test('should increment quantity for existing item', () => {});
    test('should throw error for invalid item', () => {});
  });

  describe('removeItem', () => {
    test('should remove item from cart', () => {});
    test('should handle removing non-existent item', () => {});
  });

  describe('getTotal', () => {
    test('should return 0 for empty cart', () => {});
    test('should sum all item prices', () => {});
    test('should apply discounts correctly', () => {});
  });
});
```

### Setup and Teardown
```javascript
describe('UserRepository', () => {
  let repository;
  let mockDatabase;

  beforeEach(() => {
    // Setup before each test
    mockDatabase = createMockDatabase();
    repository = new UserRepository(mockDatabase);
  });

  afterEach(() => {
    // Cleanup after each test
    jest.clearAllMocks();
  });

  test('should create user', () => {
    // Test uses fresh repository and mockDatabase
  });
});
```

## Common Patterns

### Testing Async Code
```javascript
// Using async/await
test('should fetch data successfully', async () => {
  const data = await fetchData();
  expect(data).toBeDefined();
});

// Testing promises
test('should reject with error', () => {
  return expect(fetchData()).rejects.toThrow('Network error');
});

// Testing callbacks
test('should call callback with result', (done) => {
  fetchData((err, result) => {
    expect(err).toBeNull();
    expect(result).toBeDefined();
    done();
  });
});
```

### Testing Error Handling
```javascript
test('should throw validation error for invalid input', () => {
  expect(() => {
    validateEmail('invalid-email');
  }).toThrow('Invalid email format');
});

test('should handle network errors gracefully', async () => {
  mockApi.get.mockRejectedValue(new Error('Network error'));

  await expect(fetchUserData()).rejects.toThrow('Network error');
});
```

### Parameterized Tests
```javascript
test.each([
  [2, 3, 5],
  [10, 15, 25],
  [100, 200, 300],
])('should add %i + %i to equal %i', (a, b, expected) => {
  expect(add(a, b)).toBe(expected);
});

test.each([
  ['user@example.com', true],
  ['invalid', false],
  ['@example.com', false],
])('should validate email %s as %s', (email, isValid) => {
  expect(validateEmail(email)).toBe(isValid);
});
```

## Best Practices

### ✅ Do:
- Test behavior, not implementation details
- Use descriptive test names that explain the scenario
- Keep tests independent (no shared state)
- Mock external dependencies (APIs, databases, file system)
- Test edge cases and error conditions
- Use setup/teardown for common initialization
- Aim for fast test execution (<100ms per test)
- Follow AAA pattern (Arrange-Act-Assert)

### ❌ Don't:
- Test private methods directly (test through public API)
- Make tests dependent on execution order
- Use real databases or external services
- Hardcode test data that could become stale
- Test framework code (assume it works)
- Write tests that depend on current date/time
- Ignore failing tests
- Over-mock (test becomes meaningless)

## Code Coverage Guidelines

**Target Coverage:**
- Functions: 80%+
- Branches: 75%+
- Lines: 80%+
- Critical paths: 100%

**What to Prioritize:**
1. Business logic and calculations
2. Error handling paths
3. Edge cases and boundary conditions
4. Complex conditionals
5. Data transformations

**What to Skip:**
- Simple getters/setters
- Framework boilerplate
- Configuration files
- Type definitions

## Tools and Frameworks

**Popular Testing Frameworks:**
- **Jest** - Full-featured, great for React/Node.js
- **Vitest** - Fast, Vite-native alternative to Jest
- **Mocha + Chai** - Flexible, composable
- **AVA** - Concurrent test execution
- **Tape** - Minimal, straightforward

**Assertion Libraries:**
- **Jest (built-in)** - `expect(value).toBe(expected)`
- **Chai** - `expect(value).to.equal(expected)`
- **Assert** - Node.js built-in

**Mocking Libraries:**
- **Jest mocks** - Built into Jest
- **Sinon** - Standalone spies, stubs, mocks
- **MSW** - Mock Service Worker for API mocking

## Integration with CI/CD

```yaml
# GitHub Actions example
name: Tests
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
      - run: npm ci
      - run: npm test
      - run: npm run test:coverage
```

---

**Remember:** Good unit tests are fast, isolated, and test behavior not implementation.
