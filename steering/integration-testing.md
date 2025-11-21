# Integration Testing Strategies

## Overview
Integration testing verifies that different components, modules, or services work together correctly. This guide covers strategies for testing interactions between parts of your system.

## When to Use Integration Testing
- Testing interactions between modules
- Verifying database operations
- Testing API endpoints
- Validating service integrations
- Ensuring data flows correctly through layers
- Testing third-party integrations

## Test Database Strategies

### In-Memory Database
```javascript
describe('UserRepository Integration', () => {
  let db;

  beforeAll(async () => {
    // Use in-memory SQLite for fast tests
    db = await createInMemoryDatabase();
    await db.migrate();
  });

  afterAll(async () => {
    await db.close();
  });

  test('should create and retrieve user', async () => {
    const repository = new UserRepository(db);

    await repository.create({ name: 'John', email: 'john@example.com' });
    const user = await repository.findByEmail('john@example.com');

    expect(user.name).toBe('John');
  });
});
```

### Test Containers
```javascript
const { GenericContainer } = require('testcontainers');

describe('PostgreSQL Integration', () => {
  let container;
  let database;

  beforeAll(async () => {
    // Start real PostgreSQL in Docker
    container = await new GenericContainer('postgres')
      .withExposedPorts(5432)
      .withEnv('POSTGRES_PASSWORD', 'test')
      .start();

    const port = container.getMappedPort(5432);
    database = await connect(`postgresql://postgres:test@localhost:${port}/test`);
  });

  afterAll(async () => {
    await container.stop();
  });

  test('should handle transactions correctly', async () => {
    // Test with real database
  });
});
```

## API Integration Testing

### Testing HTTP Endpoints
```javascript
const request = require('supertest');
const app = require('../app');

describe('User API', () => {
  test('POST /users should create user', async () => {
    const response = await request(app)
      .post('/users')
      .send({ name: 'John', email: 'john@example.com' })
      .expect(201);

    expect(response.body).toMatchObject({
      name: 'John',
      email: 'john@example.com'
    });
    expect(response.body.id).toBeDefined();
  });

  test('GET /users/:id should return user', async () => {
    const response = await request(app)
      .get('/users/1')
      .expect(200);

    expect(response.body.name).toBe('John');
  });
});
```

### Testing External API Integration
```javascript
const nock = require('nock');

describe('Weather Service Integration', () => {
  test('should fetch weather from external API', async () => {
    // Mock external API
    nock('https://api.weather.com')
      .get('/forecast')
      .query({ location: 'Seattle' })
      .reply(200, { temp: 65, condition: 'cloudy' });

    const service = new WeatherService();
    const weather = await service.getForecast('Seattle');

    expect(weather.temp).toBe(65);
  });
});
```

## Service Integration Testing

### Testing Service Layer
```javascript
describe('OrderService Integration', () => {
  let service;
  let mockDb;
  let mockPayment;

  beforeEach(() => {
    mockDb = createMockDatabase();
    mockPayment = createMockPaymentService();
    service = new OrderService(mockDb, mockPayment);
  });

  test('should create order and process payment', async () => {
    const order = {
      items: [{ price: 100 }],
      userId: 1
    };

    const result = await service.createOrder(order);

    expect(mockDb.orders.create).toHaveBeenCalled();
    expect(mockPayment.charge).toHaveBeenCalledWith(100);
    expect(result.status).toBe('completed');
  });
});
```

## Best Practices

### ✅ Do:
- Use real databases for critical tests (with test containers)
- Test happy path AND error scenarios
- Clean up test data after each test
- Use transactions for data isolation
- Test actual HTTP requests (not mocked routing)
- Verify data persistence and retrieval
- Test authentication and authorization flows
- Use separate test database

### ❌ Don't:
- Use production database for tests
- Leave test data in database
- Make tests depend on external services being up
- Ignore performance of integration tests
- Skip cleanup (causes test pollution)
- Test implementation details of dependencies
- Hard-code URLs or credentials

## Test Data Management

### Fixtures and Seed Data
```javascript
const fixtures = {
  users: [
    { id: 1, name: 'John', role: 'admin' },
    { id: 2, name: 'Jane', role: 'user' }
  ],
  orders: [
    { id: 1, userId: 1, total: 100 }
  ]
};

beforeEach(async () => {
  await db.seed(fixtures);
});

afterEach(async () => {
  await db.truncate();
});
```

### Factory Pattern
```javascript
const factory = {
  user: (overrides = {}) => ({
    id: Math.random(),
    name: 'Test User',
    email: 'test@example.com',
    createdAt: new Date(),
    ...overrides
  }),

  order: (overrides = {}) => ({
    id: Math.random(),
    total: 100,
    status: 'pending',
    ...overrides
  })
};

test('should process order', async () => {
  const user = factory.user({ name: 'John' });
  const order = factory.order({ userId: user.id, total: 200 });

  // Test with realistic data
});
```

## Performance Considerations

**Integration tests are slower than unit tests:**
- Target: <5 seconds per test
- Use in-memory databases when possible
- Run in parallel when safe
- Cache expensive setup
- Clean up efficiently

## Common Pitfalls

### Test Pollution
```javascript
// ❌ Bad: Shared state between tests
let sharedUser;
test('create user', () => {
  sharedUser = createUser();
});
test('update user', () => {
  updateUser(sharedUser); // Depends on previous test!
});

// ✅ Good: Independent tests
test('create user', () => {
  const user = createUser();
  expect(user).toBeDefined();
});
test('update user', () => {
  const user = createUser(); // Create fresh
  updateUser(user);
  expect(user.updatedAt).toBeDefined();
});
```

### Flaky Tests
```javascript
// ❌ Bad: Race condition
test('should process async operation', () => {
  processAsync();
  expect(result).toBeDefined(); // May not be ready!
});

// ✅ Good: Properly await
test('should process async operation', async () => {
  await processAsync();
  expect(result).toBeDefined();
});
```

---

**Next Steps:** Combine with unit tests for comprehensive coverage, then add E2E tests for critical user flows.
