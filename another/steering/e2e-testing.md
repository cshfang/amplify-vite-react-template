# End-to-End Testing Strategies

## Overview
E2E testing validates complete user workflows from start to finish, simulating real user interactions with your application. This guide covers strategies for effective E2E testing without brittleness.

## When to Use E2E Testing
- Testing critical user journeys (signup, checkout, core workflows)
- Validating features work together in production-like environment
- Testing across multiple services/systems
- Regression testing for major features
- Smoke testing deployments
- User acceptance testing

## Framework Selection

### Popular E2E Frameworks
- **Playwright** - Fast, reliable, multi-browser
- **Cypress** - Developer-friendly, great DX
- **Selenium** - Established, multi-language support
- **Puppeteer** - Chrome-focused, good for scraping too

## Test Structure

### Page Object Pattern
```javascript
// pages/LoginPage.js
class LoginPage {
  constructor(page) {
    this.page = page;
    this.emailInput = page.locator('[data-testid="email"]');
    this.passwordInput = page.locator('[data-testid="password"]');
    this.submitButton = page.locator('[data-testid="submit"]');
  }

  async login(email, password) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }
}

// tests/login.spec.js
test('should login successfully', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await page.goto('/login');

  await loginPage.login('user@example.com', 'password123');

  await expect(page).toHaveURL('/dashboard');
  await expect(page.locator('text=Welcome')).toBeVisible();
});
```

### Testing Complete User Flows
```javascript
test('complete checkout flow', async ({ page }) => {
  // 1. Browse products
  await page.goto('/products');
  await page.click('[data-product-id="123"]');

  // 2. Add to cart
  await page.click('[data-testid="add-to-cart"]');
  await expect(page.locator('.cart-count')).toHaveText('1');

  // 3. Go to checkout
  await page.click('[data-testid="cart-icon"]');
  await page.click('[data-testid="checkout"]');

  // 4. Fill shipping info
  await page.fill('[name="address"]', '123 Main St');
  await page.fill('[name="city"]', 'Seattle');
  await page.fill('[name="zip"]', '98101');

  // 5. Payment
  await page.fill('[name="card"]', '4242424242424242');
  await page.click('[data-testid="place-order"]');

  // 6. Verify confirmation
  await expect(page.locator('text=Order Confirmed')).toBeVisible();
  await expect(page.locator('.order-number')).toBeVisible();
});
```

## Selector Strategies

### Reliable Selectors
```javascript
// ✅ Best: Data attributes
await page.click('[data-testid="submit-button"]');
await page.click('[data-test="user-menu"]');

// ✅ Good: ARIA labels
await page.click('[aria-label="Close dialog"]');
await page.click('button:has-text("Submit")');

// ⚠️ Okay: Stable IDs
await page.click('#login-form');

// ❌ Avoid: CSS classes (change frequently)
await page.click('.btn.btn-primary.submit-btn');

// ❌ Avoid: Complex XPath
await page.click('//div[@class="container"]//button[2]');
```

## Waiting and Synchronization

### Proper Waiting
```javascript
// ✅ Good: Wait for specific condition
await page.waitForSelector('[data-testid="results"]');
await page.waitForLoadState('networkidle');
await page.waitForResponse(resp => resp.url().includes('/api/data'));

// ❌ Bad: Arbitrary sleep
await page.waitForTimeout(3000); // Flaky!
```

### Handling Dynamic Content
```javascript
test('should load search results', async ({ page }) => {
  await page.goto('/search');
  await page.fill('[data-testid="search-input"]', 'laptop');
  await page.click('[data-testid="search-button"]');

  // Wait for results to appear
  await page.waitForSelector('[data-testid="search-results"]');

  const results = await page.locator('[data-testid="result-item"]').count();
  expect(results).toBeGreaterThan(0);
});
```

## Test Data Management

### Creating Test Data
```javascript
test('user registration flow', async ({ page, request }) => {
  // Create test data via API
  const testUser = {
    email: `test-${Date.now()}@example.com`,
    password: 'Test123!'
  };

  await page.goto('/register');
  await page.fill('[name="email"]', testUser.email);
  await page.fill('[name="password"]', testUser.password);
  await page.click('[type="submit"]');

  await expect(page).toHaveURL('/dashboard');
});
```

### Cleanup
```javascript
afterEach(async ({ request }) => {
  // Clean up test data
  await request.delete('/api/test-data');
});
```

## Best Practices

### ✅ Do:
- Use data-testid attributes for stable selectors
- Test critical user journeys only (not every variation)
- Run E2E tests in CI/CD pipeline
- Use page object pattern for maintainability
- Take screenshots on failures
- Wait for specific conditions, not arbitrary timeouts
- Test in production-like environment
- Parallelize when possible
- Keep tests independent

### ❌ Don't:
- Test every possible path (too slow/expensive)
- Use CSS class names as selectors
- Make tests dependent on each other
- Ignore flaky tests
- Test on localhost only
- Hard-code test data
- Skip cleanup
- Test unit-level logic in E2E tests

## Flaky Test Prevention

### Common Causes and Fixes
```javascript
// ❌ Cause: Not waiting for elements
await page.click('.submit');

// ✅ Fix: Wait for visibility
await page.waitForSelector('.submit', { state: 'visible' });
await page.click('.submit');

// ❌ Cause: Race conditions
const text = await page.textContent('.message');
expect(text).toBe('Success');

// ✅ Fix: Wait for expected state
await expect(page.locator('.message')).toHaveText('Success');

// ❌ Cause: Network timing
await page.click('.load-data');
const data = await page.locator('.data').textContent();

// ✅ Fix: Wait for network
await page.click('.load-data');
await page.waitForResponse('/api/data');
const data = await page.locator('.data').textContent();
```

## Performance Optimization

### Parallel Execution
```javascript
// playwright.config.js
export default {
  workers: 4, // Run 4 tests in parallel
  fullyParallel: true
};

// Mark tests that can't run in parallel
test.describe.serial('dependent tests', () => {
  test('step 1', async () => {});
  test('step 2', async () => {});
});
```

### Reuse Authentication
```javascript
// global-setup.js
export default async () => {
  const browser = await chromium.launch();
  const page = await browser.newPage();

  await page.goto('/login');
  await page.fill('[name="email"]', 'test@example.com');
  await page.fill('[name="password"]', 'password');
  await page.click('[type="submit"]');

  // Save auth state
  await page.context().storageState({ path: 'auth.json' });
  await browser.close();
};

// Use in tests
test.use({ storageState: 'auth.json' });
```

## Visual Regression Testing

```javascript
test('homepage should match baseline', async ({ page }) => {
  await page.goto('/');
  await expect(page).toHaveScreenshot('homepage.png');
});

test('button hover state', async ({ page }) => {
  await page.goto('/');
  await page.hover('[data-testid="cta-button"]');
  await expect(page.locator('[data-testid="cta-button"]')).toHaveScreenshot();
});
```

## Testing Different Scenarios

### Mobile Testing
```javascript
test('should work on mobile viewport', async ({ page }) => {
  await page.setViewportSize({ width: 375, height: 667 });
  await page.goto('/');

  // Test mobile-specific UI
  await page.click('[data-testid="mobile-menu"]');
  await expect(page.locator('.mobile-nav')).toBeVisible();
});
```

### Cross-Browser Testing
```javascript
// playwright.config.js
projects: [
  { name: 'chromium' },
  { name: 'firefox' },
  { name: 'webkit' } // Safari
]
```

## When to Run E2E Tests

**Always:**
- Before major releases
- After critical feature changes
- In staging environment

**Selectively:**
- On every commit (only smoke tests)
- Nightly (full suite)
- Before deployments

## Test Organization

```
e2e/
├── specs/
│   ├── authentication.spec.js
│   ├── checkout.spec.js
│   └── search.spec.js
├── pages/
│   ├── LoginPage.js
│   └── CheckoutPage.js
├── fixtures/
│   └── test-data.json
└── utils/
    └── helpers.js
```

---

**Remember:** E2E tests are expensive. Test critical paths, not every combination. Keep tests fast, reliable, and maintainable.
