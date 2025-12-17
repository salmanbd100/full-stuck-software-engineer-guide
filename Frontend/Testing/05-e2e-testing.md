# End-to-End (E2E) Testing

## Overview

End-to-End testing validates complete user workflows from start to finish, simulating real user interactions with your application. Unlike unit or integration tests, E2E tests verify that all parts of your application work together correctly - frontend, backend, databases, and third-party services. Understanding E2E testing is crucial for ensuring application reliability and is a common topic in senior-level interviews.

## Table of Contents
- [What is E2E Testing](#what-is-e2e-testing)
- [When to Use E2E Tests](#when-to-use-e2e-tests)
- [Playwright vs Cypress](#playwright-vs-cypress)
- [Writing User Flow Tests](#writing-user-flow-tests)
- [Page Object Model Pattern](#page-object-model-pattern)
- [Selectors and Best Practices](#selectors-and-best-practices)
- [CI/CD Integration](#cicd-integration)
- [Visual Regression Testing](#visual-regression-testing)
- [Common Patterns](#common-patterns)
- [Interview Questions](#interview-questions)

## What is E2E Testing

### Definition and Scope

```javascript
// E2E tests simulate real user journeys
// Example: Complete login flow

test('user can login and view dashboard', async () => {
  // 1. Navigate to app
  await page.goto('https://example.com');

  // 2. Fill login form
  await page.fill('[name="email"]', 'user@example.com');
  await page.fill('[name="password"]', 'password123');

  // 3. Submit form
  await page.click('button[type="submit"]');

  // 4. Verify redirect to dashboard
  await expect(page).toHaveURL('/dashboard');

  // 5. Verify dashboard content
  await expect(page.locator('h1')).toContainText('Welcome');
});
```

### E2E vs Other Test Types

```javascript
// Unit Test: Tests individual function
test('validateEmail returns true for valid email', () => {
  expect(validateEmail('test@example.com')).toBe(true);
});

// Integration Test: Tests component with API
test('UserList fetches and displays users', async () => {
  render(<UserList />);
  await waitFor(() => {
    expect(screen.getByText('John Doe')).toBeInTheDocument();
  });
});

// E2E Test: Tests complete user flow
test('user can create account and login', async () => {
  await page.goto('/signup');
  await page.fill('[name="email"]', 'new@example.com');
  await page.fill('[name="password"]', 'password123');
  await page.click('button[type="submit"]');
  await expect(page).toHaveURL('/dashboard');
});
```

### Testing Pyramid Position

```
         /\
        /  \     E2E Tests (Few)
       /    \    - 5-10% of tests
      /______\   - Slow (~10-60s per test)
     /        \  - Expensive to maintain
    /          \ - Test critical paths
   / Integration\ (Some)
  /    Tests     \
 /________________\ - 20-30% of tests
/                  \
/    Unit Tests     \ (Many)
/____________________\
      - 60-75% of tests
      - Fast (~10-100ms)
      - Cheap to maintain
```

## When to Use E2E Tests

### Good Use Cases

```javascript
// ✅ Critical user journeys
test('user can complete checkout', async () => {
  await addItemsToCart();
  await proceedToCheckout();
  await fillShippingInfo();
  await fillPaymentInfo();
  await confirmOrder();
  await expect(page.locator('.order-confirmation')).toBeVisible();
});

// ✅ Authentication flows
test('user can login with OAuth', async () => {
  await page.click('button:text("Login with Google")');
  await page.fill('[name="email"]', 'user@gmail.com');
  await page.fill('[name="password"]', 'password');
  await page.click('button:text("Sign in")');
  await expect(page).toHaveURL('/dashboard');
});

// ✅ Multi-step forms
test('user can complete onboarding', async () => {
  await fillPersonalInfo();
  await page.click('button:text("Next")');
  await fillCompanyInfo();
  await page.click('button:text("Next")');
  await selectPreferences();
  await page.click('button:text("Finish")');
  await expect(page.locator('.welcome-message')).toBeVisible();
});

// ✅ Payment processing
test('user can purchase subscription', async () => {
  await selectPlan('premium');
  await enterPaymentDetails();
  await submitPayment();
  await expect(page.locator('.success-message')).toContainText('Subscription active');
});
```

### When NOT to Use E2E Tests

```javascript
// ❌ Unit-level logic
test('sum function adds numbers', async () => {
  // Too slow for simple function - use unit test
  await page.evaluate(() => sum(2, 3));
});

// ❌ Edge cases and validation
test('email validation shows error for invalid format', async () => {
  // Better as integration test with React Testing Library
  await page.fill('[name="email"]', 'invalid');
  await page.click('button[type="submit"]');
  await expect(page.locator('.error')).toBeVisible();
});

// ❌ Component styling
test('button has correct background color', async () => {
  // Visual regression or snapshot test is better
  const color = await page.locator('button').evaluate(el =>
    getComputedStyle(el).backgroundColor
  );
});
```

## Playwright vs Cypress

### Feature Comparison

```javascript
// Feature matrix
const comparison = {
  browsers: {
    playwright: 'Chromium, Firefox, WebKit (Safari)',
    cypress: 'Chrome, Firefox, Edge (Chromium-based only)'
  },

  language: {
    playwright: 'JavaScript, TypeScript, Python, Java, .NET',
    cypress: 'JavaScript, TypeScript only'
  },

  multiTab: {
    playwright: 'Yes - native support',
    cypress: 'Limited - workarounds needed'
  },

  iframes: {
    playwright: 'Full support',
    cypress: 'Limited support'
  },

  autoWait: {
    playwright: 'Built-in automatic waiting',
    cypress: 'Built-in automatic waiting'
  },

  speed: {
    playwright: 'Faster execution',
    cypress: 'Slightly slower'
  },

  debugging: {
    playwright: 'Inspector, trace viewer',
    cypress: 'Time-travel debugging, great DX'
  }
};
```

### Playwright Example

```javascript
// Playwright: Modern, fast, cross-browser
import { test, expect } from '@playwright/test';

test.describe('Login flow', () => {
  test('user can login with valid credentials', async ({ page }) => {
    // Navigate
    await page.goto('https://example.com/login');

    // Fill form
    await page.fill('[name="email"]', 'user@example.com');
    await page.fill('[name="password"]', 'password123');

    // Submit
    await page.click('button[type="submit"]');

    // Assertions
    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('h1')).toContainText('Dashboard');
  });

  test('handles invalid credentials', async ({ page }) => {
    await page.goto('https://example.com/login');
    await page.fill('[name="email"]', 'wrong@example.com');
    await page.fill('[name="password"]', 'wrongpass');
    await page.click('button[type="submit"]');

    // Error message appears
    await expect(page.locator('.error')).toContainText('Invalid credentials');
  });
});

// Multi-browser testing
test.describe('Cross-browser tests', () => {
  test.use({ browserName: 'firefox' });

  test('works in Firefox', async ({ page }) => {
    await page.goto('https://example.com');
    await expect(page.locator('h1')).toBeVisible();
  });
});

// Mobile viewport
test.describe('Mobile tests', () => {
  test.use({ viewport: { width: 375, height: 667 } });

  test('mobile navigation works', async ({ page }) => {
    await page.goto('https://example.com');
    await page.click('.hamburger-menu');
    await expect(page.locator('nav')).toBeVisible();
  });
});
```

### Cypress Example

```javascript
// Cypress: Great DX, time-travel debugging
describe('Login flow', () => {
  beforeEach(() => {
    cy.visit('https://example.com/login');
  });

  it('user can login with valid credentials', () => {
    // Fill form
    cy.get('[name="email"]').type('user@example.com');
    cy.get('[name="password"]').type('password123');

    // Submit
    cy.get('button[type="submit"]').click();

    // Assertions
    cy.url().should('include', '/dashboard');
    cy.get('h1').should('contain', 'Dashboard');
  });

  it('handles invalid credentials', () => {
    cy.get('[name="email"]').type('wrong@example.com');
    cy.get('[name="password"]').type('wrongpass');
    cy.get('button[type="submit"]').click();

    // Error message appears
    cy.get('.error').should('contain', 'Invalid credentials');
  });
});

// API mocking with Cypress
describe('User dashboard', () => {
  beforeEach(() => {
    // Intercept API calls
    cy.intercept('GET', '/api/user', {
      statusCode: 200,
      body: {
        name: 'John Doe',
        email: 'john@example.com'
      }
    }).as('getUser');

    cy.visit('/dashboard');
    cy.wait('@getUser');
  });

  it('displays user information', () => {
    cy.get('.user-name').should('contain', 'John Doe');
    cy.get('.user-email').should('contain', 'john@example.com');
  });
});

// Custom commands
Cypress.Commands.add('login', (email, password) => {
  cy.visit('/login');
  cy.get('[name="email"]').type(email);
  cy.get('[name="password"]').type(password);
  cy.get('button[type="submit"]').click();
});

// Usage
it('user can access protected page after login', () => {
  cy.login('user@example.com', 'password123');
  cy.visit('/protected');
  cy.get('.protected-content').should('be.visible');
});
```

## Writing User Flow Tests

### Complete User Journeys

```javascript
// Playwright: E-commerce checkout flow
import { test, expect } from '@playwright/test';

test('complete purchase flow', async ({ page }) => {
  // 1. Browse products
  await page.goto('https://shop.example.com');
  await expect(page.locator('h1')).toContainText('Products');

  // 2. Search for product
  await page.fill('[data-testid="search"]', 'laptop');
  await page.press('[data-testid="search"]', 'Enter');
  await page.waitForURL('**/search?q=laptop');

  // 3. Select product
  await page.click('text=MacBook Pro');
  await expect(page.locator('.product-title')).toContainText('MacBook Pro');

  // 4. Add to cart
  await page.click('button:text("Add to Cart")');
  await expect(page.locator('.cart-count')).toHaveText('1');

  // 5. Go to cart
  await page.click('[data-testid="cart-icon"]');
  await expect(page).toHaveURL('/cart');

  // 6. Proceed to checkout
  await page.click('button:text("Checkout")');
  await expect(page).toHaveURL('/checkout');

  // 7. Fill shipping information
  await page.fill('[name="firstName"]', 'John');
  await page.fill('[name="lastName"]', 'Doe');
  await page.fill('[name="address"]', '123 Main St');
  await page.fill('[name="city"]', 'New York');
  await page.fill('[name="zip"]', '10001');
  await page.click('button:text("Continue")');

  // 8. Fill payment information
  await page.fill('[name="cardNumber"]', '4242424242424242');
  await page.fill('[name="expiry"]', '12/25');
  await page.fill('[name="cvc"]', '123');

  // 9. Place order
  await page.click('button:text("Place Order")');

  // 10. Verify order confirmation
  await expect(page).toHaveURL('/order/confirmation');
  await expect(page.locator('.success-message')).toContainText('Order placed successfully');

  // 11. Verify order details
  const orderNumber = await page.locator('.order-number').textContent();
  expect(orderNumber).toMatch(/^ORD-\d+$/);
});
```

### Authentication Flows

```javascript
// User registration and login
test.describe('Authentication', () => {
  test('new user can sign up and login', async ({ page }) => {
    const email = `test-${Date.now()}@example.com`;
    const password = 'SecurePass123!';

    // Sign up
    await page.goto('/signup');
    await page.fill('[name="email"]', email);
    await page.fill('[name="password"]', password);
    await page.fill('[name="confirmPassword"]', password);
    await page.check('[name="terms"]');
    await page.click('button:text("Sign Up")');

    // Verify email verification page
    await expect(page).toHaveURL('/verify-email');
    await expect(page.locator('.message')).toContainText('Check your email');

    // Simulate email verification (in real test, you'd check email)
    await page.goto(`/verify?token=mock-token`);
    await expect(page.locator('.success')).toContainText('Email verified');

    // Login with new account
    await page.goto('/login');
    await page.fill('[name="email"]', email);
    await page.fill('[name="password"]', password);
    await page.click('button:text("Login")');

    // Verify successful login
    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('.welcome')).toContainText('Welcome');
  });

  test('user can reset password', async ({ page }) => {
    await page.goto('/forgot-password');
    await page.fill('[name="email"]', 'user@example.com');
    await page.click('button:text("Send Reset Link")');

    await expect(page.locator('.message')).toContainText('Reset link sent');

    // Simulate clicking reset link from email
    await page.goto('/reset-password?token=mock-token');
    await page.fill('[name="newPassword"]', 'NewPassword123!');
    await page.fill('[name="confirmPassword"]', 'NewPassword123!');
    await page.click('button:text("Reset Password")');

    await expect(page.locator('.success')).toContainText('Password reset successful');
  });
});
```

### Multi-Step Forms

```javascript
// Job application form with multiple steps
test('user can submit job application', async ({ page }) => {
  await page.goto('/careers/apply');

  // Step 1: Personal Information
  await page.fill('[name="firstName"]', 'Jane');
  await page.fill('[name="lastName"]', 'Smith');
  await page.fill('[name="email"]', 'jane@example.com');
  await page.fill('[name="phone"]', '+1234567890');
  await page.click('button:text("Next")');

  // Step 2: Education
  await page.fill('[name="degree"]', 'Bachelor of Science');
  await page.fill('[name="university"]', 'MIT');
  await page.fill('[name="graduationYear"]', '2020');
  await page.click('button:text("Next")');

  // Step 3: Experience
  await page.fill('[name="company"]', 'Tech Corp');
  await page.fill('[name="position"]', 'Software Engineer');
  await page.fill('[name="years"]', '3');
  await page.click('button:text("Next")');

  // Step 4: Upload Resume
  await page.setInputFiles('[name="resume"]', './fixtures/resume.pdf');
  await expect(page.locator('.file-name')).toContainText('resume.pdf');
  await page.click('button:text("Next")');

  // Step 5: Review and Submit
  await expect(page.locator('.review-name')).toContainText('Jane Smith');
  await expect(page.locator('.review-degree')).toContainText('Bachelor of Science');
  await page.click('button:text("Submit Application")');

  // Confirmation
  await expect(page).toHaveURL('/application/confirmation');
  await expect(page.locator('.success')).toContainText('Application submitted');
});
```

## Page Object Model Pattern

### Basic Page Object

```javascript
// pages/LoginPage.js
export class LoginPage {
  constructor(page) {
    this.page = page;

    // Selectors
    this.emailInput = '[name="email"]';
    this.passwordInput = '[name="password"]';
    this.submitButton = 'button[type="submit"]';
    this.errorMessage = '.error-message';
  }

  // Actions
  async goto() {
    await this.page.goto('https://example.com/login');
  }

  async login(email, password) {
    await this.page.fill(this.emailInput, email);
    await this.page.fill(this.passwordInput, password);
    await this.page.click(this.submitButton);
  }

  async fillEmail(email) {
    await this.page.fill(this.emailInput, email);
  }

  async fillPassword(password) {
    await this.page.fill(this.passwordInput, password);
  }

  async submit() {
    await this.page.click(this.submitButton);
  }

  // Assertions
  async expectErrorMessage(message) {
    await expect(this.page.locator(this.errorMessage)).toContainText(message);
  }
}

// Usage in test
import { test } from '@playwright/test';
import { LoginPage } from './pages/LoginPage';

test('user can login', async ({ page }) => {
  const loginPage = new LoginPage(page);

  await loginPage.goto();
  await loginPage.login('user@example.com', 'password123');

  await expect(page).toHaveURL('/dashboard');
});
```

### Advanced Page Object with Getters

```javascript
// pages/DashboardPage.js
export class DashboardPage {
  constructor(page) {
    this.page = page;
  }

  // Getters for locators
  get welcomeMessage() {
    return this.page.locator('.welcome-message');
  }

  get userMenu() {
    return this.page.locator('[data-testid="user-menu"]');
  }

  get logoutButton() {
    return this.page.locator('button:text("Logout")');
  }

  get notificationCount() {
    return this.page.locator('.notification-badge');
  }

  // Actions
  async goto() {
    await this.page.goto('/dashboard');
  }

  async logout() {
    await this.userMenu.click();
    await this.logoutButton.click();
  }

  async openNotifications() {
    await this.page.click('.notification-icon');
  }

  // Assertions
  async expectWelcomeMessage(name) {
    await expect(this.welcomeMessage).toContainText(`Welcome, ${name}`);
  }

  async expectNotificationCount(count) {
    await expect(this.notificationCount).toHaveText(count.toString());
  }
}
```

### Component-Based Page Objects

```javascript
// components/NavigationComponent.js
export class NavigationComponent {
  constructor(page) {
    this.page = page;
    this.nav = page.locator('nav');
  }

  async clickHome() {
    await this.nav.locator('a:text("Home")').click();
  }

  async clickProfile() {
    await this.nav.locator('a:text("Profile")').click();
  }

  async search(query) {
    await this.nav.locator('[data-testid="search"]').fill(query);
    await this.nav.locator('[data-testid="search"]').press('Enter');
  }
}

// pages/BasePage.js
import { NavigationComponent } from '../components/NavigationComponent';

export class BasePage {
  constructor(page) {
    this.page = page;
    this.navigation = new NavigationComponent(page);
  }
}

// pages/ProfilePage.js
export class ProfilePage extends BasePage {
  constructor(page) {
    super(page);
  }

  get profileName() {
    return this.page.locator('.profile-name');
  }

  async goto() {
    await this.page.goto('/profile');
  }

  async editProfile() {
    await this.page.click('button:text("Edit Profile")');
  }
}

// Usage
test('user can navigate to profile', async ({ page }) => {
  const profilePage = new ProfilePage(page);

  await profilePage.goto();
  await profilePage.navigation.search('settings');
});
```

## Selectors and Best Practices

### Selector Priority

```javascript
// Priority order (best to worst)
const selectorPriority = {
  1: 'data-testid attributes',
  2: 'ARIA roles and labels',
  3: 'Semantic HTML',
  4: 'User-visible text',
  5: 'CSS classes (least preferred)'
};
```

### Data-TestId Selectors

```javascript
// ✅ Best practice: data-testid
// HTML
<button data-testid="submit-button">Submit</button>
<input data-testid="email-input" name="email" />

// Test
await page.click('[data-testid="submit-button"]');
await page.fill('[data-testid="email-input"]', 'user@example.com');

// Playwright built-in
await page.locator('data-testid=submit-button').click();
```

### ARIA and Semantic Selectors

```javascript
// ✅ Good: ARIA roles
await page.click('button[role="button"]:text("Submit")');
await page.getByRole('button', { name: 'Submit' }).click();
await page.getByRole('textbox', { name: 'Email' }).fill('user@example.com');

// ✅ Good: Semantic HTML
await page.locator('nav >> a:text("Home")').click();
await page.locator('main >> h1').textContent();

// Playwright built-in queries
await page.getByLabel('Email').fill('user@example.com');
await page.getByPlaceholder('Enter email').fill('user@example.com');
await page.getByText('Welcome back').click();
await page.getByTitle('Close').click();
```

### Avoid Fragile Selectors

```javascript
// ❌ Bad: CSS classes (can change)
await page.click('.btn-primary.btn-lg.submit-btn');

// ❌ Bad: Complex CSS selectors
await page.click('div > div > button:nth-child(3)');

// ❌ Bad: XPath (hard to read)
await page.click('//div[@class="container"]//button[text()="Submit"]');

// ✅ Good: Stable, semantic selectors
await page.click('[data-testid="submit-button"]');
await page.getByRole('button', { name: 'Submit' }).click();
await page.locator('button:text("Submit")').click();
```

### Combining Selectors

```javascript
// Filter within container
await page.locator('[data-testid="user-list"]')
  .locator('[data-testid="user-item"]')
  .filter({ hasText: 'John Doe' })
  .click();

// Chain selectors
await page.locator('nav')
  .locator('button:text("Menu")')
  .click();

// Multiple conditions
await page.locator('button')
  .filter({ hasText: 'Submit' })
  .filter({ has: page.locator('.icon-send') })
  .click();
```

## CI/CD Integration

### GitHub Actions with Playwright

```yaml
# .github/workflows/e2e-tests.yml
name: E2E Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        browser: [chromium, firefox, webkit]

    steps:
      - uses: actions/checkout@v3

      - uses: actions/setup-node@v3
        with:
          node-version: 18

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright browsers
        run: npx playwright install --with-deps ${{ matrix.browser }}

      - name: Run E2E tests
        run: npx playwright test --project=${{ matrix.browser }}
        env:
          BASE_URL: http://localhost:3000

      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: playwright-report-${{ matrix.browser }}
          path: playwright-report/
          retention-days: 7

      - name: Upload screenshots
        if: failure()
        uses: actions/upload-artifact@v3
        with:
          name: screenshots-${{ matrix.browser }}
          path: test-results/
```

### Playwright Configuration

```javascript
// playwright.config.js
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',

  // Run tests in parallel
  fullyParallel: true,

  // Fail build on CI if tests were accidentally skipped
  forbidOnly: !!process.env.CI,

  // Retry on CI
  retries: process.env.CI ? 2 : 0,

  // Workers (parallel execution)
  workers: process.env.CI ? 1 : undefined,

  // Reporter
  reporter: process.env.CI
    ? [['html'], ['github'], ['junit', { outputFile: 'results.xml' }]]
    : [['html'], ['list']],

  use: {
    // Base URL
    baseURL: process.env.BASE_URL || 'http://localhost:3000',

    // Screenshots
    screenshot: 'only-on-failure',

    // Videos
    video: 'retain-on-failure',

    // Traces
    trace: 'on-first-retry',
  },

  // Projects for multiple browsers
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'mobile-chrome',
      use: { ...devices['Pixel 5'] },
    },
    {
      name: 'mobile-safari',
      use: { ...devices['iPhone 12'] },
    },
  ],

  // Web server
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
    timeout: 120000,
  },
});
```

### Cypress CI Configuration

```yaml
# .github/workflows/cypress.yml
name: Cypress Tests

on: [push]

jobs:
  cypress-run:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Cypress run
        uses: cypress-io/github-action@v5
        with:
          build: npm run build
          start: npm start
          wait-on: 'http://localhost:3000'
          browser: chrome
          headless: true

      - name: Upload screenshots
        if: failure()
        uses: actions/upload-artifact@v3
        with:
          name: cypress-screenshots
          path: cypress/screenshots

      - name: Upload videos
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: cypress-videos
          path: cypress/videos
```

## Visual Regression Testing

### Playwright Visual Comparisons

```javascript
// Visual regression with Playwright
import { test, expect } from '@playwright/test';

test('homepage looks correct', async ({ page }) => {
  await page.goto('/');

  // Full page screenshot comparison
  await expect(page).toHaveScreenshot('homepage.png');
});

test('login form looks correct', async ({ page }) => {
  await page.goto('/login');

  // Element screenshot
  const loginForm = page.locator('[data-testid="login-form"]');
  await expect(loginForm).toHaveScreenshot('login-form.png');
});

// With options
test('responsive design', async ({ page }) => {
  await page.setViewportSize({ width: 375, height: 667 });
  await page.goto('/');

  await expect(page).toHaveScreenshot('mobile-homepage.png', {
    fullPage: true,
    animations: 'disabled',
    maxDiffPixels: 100,
  });
});

// Multiple states
test('button states', async ({ page }) => {
  await page.goto('/components');

  const button = page.locator('[data-testid="submit-button"]');

  // Default state
  await expect(button).toHaveScreenshot('button-default.png');

  // Hover state
  await button.hover();
  await expect(button).toHaveScreenshot('button-hover.png');

  // Focus state
  await button.focus();
  await expect(button).toHaveScreenshot('button-focus.png');

  // Disabled state
  await page.evaluate(() => {
    document.querySelector('[data-testid="submit-button"]').disabled = true;
  });
  await expect(button).toHaveScreenshot('button-disabled.png');
});
```

### Percy Integration

```javascript
// Visual testing with Percy
import percySnapshot from '@percy/playwright';
import { test } from '@playwright/test';

test('visual regression with Percy', async ({ page }) => {
  await page.goto('/');

  // Take Percy snapshot
  await percySnapshot(page, 'Homepage');
});

test('responsive snapshots', async ({ page }) => {
  await page.goto('/dashboard');

  // Multiple widths
  await percySnapshot(page, 'Dashboard', {
    widths: [375, 768, 1280, 1920],
  });
});

test('component library', async ({ page }) => {
  await page.goto('/components');

  // Snapshot with Percy-specific options
  await percySnapshot(page, 'Component Library', {
    percyCSS: '.ad-banner { display: none; }', // Hide dynamic content
    minHeight: 1024,
  });
});
```

## Common Patterns

### Authentication State Management

```javascript
// Save authentication state
import { test as setup } from '@playwright/test';

setup('authenticate', async ({ page }) => {
  await page.goto('/login');
  await page.fill('[name="email"]', 'user@example.com');
  await page.fill('[name="password"]', 'password123');
  await page.click('button[type="submit"]');

  await page.waitForURL('/dashboard');

  // Save storage state
  await page.context().storageState({ path: 'auth.json' });
});

// Use authenticated state in tests
import { test } from '@playwright/test';

test.use({ storageState: 'auth.json' });

test('user can view profile', async ({ page }) => {
  // Already logged in
  await page.goto('/profile');
  // Test protected page
});
```

### API Mocking

```javascript
// Mock API responses
test('display loading state', async ({ page }) => {
  // Delay API response
  await page.route('**/api/users', async route => {
    await new Promise(resolve => setTimeout(resolve, 5000));
    await route.fulfill({
      status: 200,
      body: JSON.stringify([{ id: 1, name: 'John' }])
    });
  });

  await page.goto('/users');
  await expect(page.locator('.loading')).toBeVisible();
});

test('handle API errors', async ({ page }) => {
  // Mock error response
  await page.route('**/api/users', route => {
    route.fulfill({
      status: 500,
      body: JSON.stringify({ error: 'Server error' })
    });
  });

  await page.goto('/users');
  await expect(page.locator('.error')).toContainText('Failed to load users');
});
```

### Waiting Strategies

```javascript
// Wait for navigation
await page.click('a[href="/about"]');
await page.waitForURL('/about');

// Wait for selector
await page.waitForSelector('[data-testid="content"]');

// Wait for network idle
await page.goto('/', { waitUntil: 'networkidle' });

// Wait for custom condition
await page.waitForFunction(() => {
  return document.querySelectorAll('.list-item').length > 5;
});

// Wait for response
const responsePromise = page.waitForResponse('**/api/data');
await page.click('button:text("Load Data")');
const response = await responsePromise;
expect(response.status()).toBe(200);
```

## Interview Questions

**Q1: What is E2E testing and when should you use it?**

A: E2E (End-to-End) testing validates complete user workflows from start to finish, simulating real user interactions across the entire application stack.

**When to use:**
- Critical user journeys (login, checkout, signup)
- Multi-step processes (onboarding, forms)
- Integration between systems (frontend, backend, third-party APIs)
- Cross-browser compatibility
- Production-like environment testing

**When NOT to use:**
- Unit-level logic (too slow)
- Edge cases (better for integration tests)
- Every possible scenario (too expensive)

**Example:**
```javascript
// ✅ Good E2E test - critical flow
test('user can complete purchase', async ({ page }) => {
  await addItemToCart(page);
  await checkout(page);
  await enterPayment(page);
  await expect(page).toHaveURL('/order-confirmation');
});

// ❌ Bad E2E test - should be unit test
test('calculateTotal adds numbers correctly', async ({ page }) => {
  const result = await page.evaluate(() => calculateTotal(10, 5));
  expect(result).toBe(15);
});
```

**Q2: Compare Playwright and Cypress.**

A: Both are modern E2E frameworks with different strengths:

**Playwright:**
- **Browsers:** Chromium, Firefox, WebKit (Safari)
- **Language:** JS, TS, Python, Java, .NET
- **Speed:** Faster execution
- **Features:** Multi-tab, iframe support, auto-wait
- **Best for:** Cross-browser, parallel execution, complex scenarios

**Cypress:**
- **Browsers:** Chrome, Firefox, Edge (Chromium-based)
- **Language:** JavaScript, TypeScript only
- **Developer Experience:** Excellent time-travel debugging
- **Features:** Auto-wait, great documentation
- **Best for:** Developer-friendly DX, debugging

**Code comparison:**
```javascript
// Playwright
test('login', async ({ page }) => {
  await page.goto('/login');
  await page.fill('[name="email"]', 'user@example.com');
  await page.click('button[type="submit"]');
});

// Cypress
it('login', () => {
  cy.visit('/login');
  cy.get('[name="email"]').type('user@example.com');
  cy.get('button[type="submit"]').click();
});
```

**Q3: Explain the Page Object Model pattern.**

A: Page Object Model (POM) is a design pattern that creates an abstraction layer between tests and page elements, improving maintainability.

**Benefits:**
- **DRY principle:** Reusable page interactions
- **Maintainability:** Change selectors in one place
- **Readability:** Tests focus on behavior, not implementation
- **Separation of concerns:** Test logic separate from page details

**Implementation:**
```javascript
// pages/LoginPage.js
export class LoginPage {
  constructor(page) {
    this.page = page;
    this.emailInput = '[name="email"]';
    this.passwordInput = '[name="password"]';
    this.submitButton = 'button[type="submit"]';
  }

  async goto() {
    await this.page.goto('/login');
  }

  async login(email, password) {
    await this.page.fill(this.emailInput, email);
    await this.page.fill(this.passwordInput, password);
    await this.page.click(this.submitButton);
  }
}

// test.spec.js
test('user can login', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.goto();
  await loginPage.login('user@example.com', 'password');
});
```

**Q4: What are best practices for selecting elements in E2E tests?**

A: Use stable, meaningful selectors that won't break with UI changes.

**Priority order:**
1. **data-testid** (best)
2. **ARIA roles/labels**
3. **Semantic HTML**
4. **User-visible text**
5. **CSS classes** (avoid)

```javascript
// ✅ Best: data-testid
await page.click('[data-testid="submit-button"]');

// ✅ Good: ARIA roles
await page.getByRole('button', { name: 'Submit' }).click();

// ✅ Good: Semantic selectors
await page.getByLabel('Email').fill('user@example.com');

// ⚠️ Okay: User-visible text
await page.click('button:text("Submit")');

// ❌ Bad: CSS classes (fragile)
await page.click('.btn-primary.submit');

// ❌ Bad: Complex selectors
await page.click('div > div > button:nth-child(3)');
```

**Why data-testid?**
- Stable (not tied to styling)
- Explicit testing intent
- Won't break with refactoring

**Q5: How do you handle authentication in E2E tests?**

A: Use authentication state management to avoid logging in before every test.

```javascript
// setup.js - Run once to save auth state
import { test as setup } from '@playwright/test';

setup('authenticate', async ({ page }) => {
  await page.goto('/login');
  await page.fill('[name="email"]', 'user@example.com');
  await page.fill('[name="password"]', 'password123');
  await page.click('button[type="submit"]');
  await page.waitForURL('/dashboard');

  // Save cookies and localStorage
  await page.context().storageState({ path: 'auth.json' });
});

// test.spec.js - Reuse auth state
test.use({ storageState: 'auth.json' });

test('view profile', async ({ page }) => {
  // Already authenticated
  await page.goto('/profile');
  await expect(page.locator('.user-name')).toBeVisible();
});
```

**Benefits:**
- Faster tests (no repeated logins)
- Test isolation maintained
- Realistic authentication flow

**Q6: How do you make E2E tests reliable and avoid flakiness?**

A: Follow these practices:

**1. Use built-in waits:**
```javascript
// ✅ Good: Auto-wait
await page.click('button'); // Waits for button to be ready

// ❌ Bad: Arbitrary timeout
await page.waitForTimeout(2000); // Fragile
```

**2. Wait for specific conditions:**
```javascript
// ✅ Wait for element
await page.waitForSelector('[data-testid="content"]');

// ✅ Wait for URL
await page.waitForURL('/dashboard');

// ✅ Wait for network
await page.waitForLoadState('networkidle');
```

**3. Avoid timing dependencies:**
```javascript
// ❌ Bad
setTimeout(() => expect(element).toBeVisible(), 1000);

// ✅ Good
await expect(element).toBeVisible({ timeout: 5000 });
```

**4. Use test isolation:**
```javascript
// Reset state between tests
test.beforeEach(async ({ page }) => {
  await page.goto('/');
  // Clear cookies, localStorage
  await page.context().clearCookies();
});
```

**5. Mock external dependencies:**
```javascript
// Don't depend on external APIs
await page.route('**/api/external', route => {
  route.fulfill({ body: JSON.stringify({ data: 'mock' }) });
});
```

**Q7: How do you integrate E2E tests into CI/CD?**

A: Configure tests to run automatically on code changes.

**GitHub Actions example:**
```yaml
name: E2E Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npx playwright test

      - name: Upload artifacts
        if: failure()
        uses: actions/upload-artifact@v3
        with:
          name: test-results
          path: test-results/
```

**Best practices:**
- Run on every PR
- Parallel execution for speed
- Retry flaky tests (up to 2 times)
- Upload screenshots/videos on failure
- Run critical tests on main branch only
- Use matrix strategy for multi-browser

**Q8: What is visual regression testing?**

A: Visual regression testing compares screenshots to detect unintended UI changes.

**How it works:**
1. Take baseline screenshot
2. Make code changes
3. Take new screenshot
4. Compare: flag differences

**Playwright example:**
```javascript
test('button looks correct', async ({ page }) => {
  await page.goto('/');

  // First run: creates baseline
  // Subsequent runs: compares against baseline
  await expect(page).toHaveScreenshot('homepage.png');
});

// With options
await expect(page).toHaveScreenshot({
  maxDiffPixels: 100, // Allow minor differences
  animations: 'disabled', // Disable animations
  fullPage: true
});
```

**When to use:**
- UI component libraries
- Design system changes
- Critical pages (homepage, checkout)
- Cross-browser visual differences

**Tools:**
- Playwright (built-in)
- Percy
- Chromatic
- BackstopJS

**Q9: How do you handle asynchronous operations in E2E tests?**

A: Modern E2E frameworks auto-wait, but you can explicitly wait for specific conditions.

**Playwright auto-wait:**
```javascript
// Auto-waits for element to be ready
await page.click('button'); // Waits until clickable
await page.fill('input', 'text'); // Waits until fillable
```

**Explicit waits:**
```javascript
// Wait for selector
await page.waitForSelector('[data-testid="content"]');

// Wait for response
const responsePromise = page.waitForResponse('**/api/data');
await page.click('button:text("Load")');
const response = await responsePromise;

// Wait for function
await page.waitForFunction(() => {
  return document.querySelectorAll('.item').length > 5;
});

// Wait for load state
await page.goto('/', { waitUntil: 'networkidle' });

// Wait with timeout
await expect(page.locator('.message')).toBeVisible({ timeout: 10000 });
```

**Handling loading states:**
```javascript
test('displays loading spinner', async ({ page }) => {
  // Slow down API
  await page.route('**/api/data', async route => {
    await new Promise(resolve => setTimeout(resolve, 3000));
    await route.fulfill({ body: JSON.stringify({ data: [] }) });
  });

  await page.goto('/');
  await page.click('button:text("Load Data")');

  // Loading spinner appears
  await expect(page.locator('.spinner')).toBeVisible();

  // Then disappears
  await expect(page.locator('.spinner')).not.toBeVisible();

  // Content loads
  await expect(page.locator('.data')).toBeVisible();
});
```

**Q10: How do you debug failing E2E tests?**

A: Use these debugging techniques:

**1. Screenshots and videos:**
```javascript
// playwright.config.js
use: {
  screenshot: 'only-on-failure',
  video: 'retain-on-failure',
}
```

**2. Traces:**
```javascript
// Enable traces
use: {
  trace: 'on-first-retry',
}

// View trace
// npx playwright show-trace trace.zip
```

**3. Debug mode:**
```bash
# Playwright
npx playwright test --debug

# Cypress
npx cypress open
```

**4. Pause execution:**
```javascript
await page.pause(); // Opens inspector

// Or use debugger
await page.evaluate(() => debugger);
```

**5. Console logs:**
```javascript
page.on('console', msg => console.log(msg.text()));

test('debug', async ({ page }) => {
  await page.goto('/');

  // Log element state
  const text = await page.locator('h1').textContent();
  console.log('Heading:', text);
});
```

**6. Slow down execution:**
```javascript
// playwright.config.js
use: {
  launchOptions: {
    slowMo: 100, // Slow down by 100ms
  }
}
```

**7. Check network:**
```javascript
page.on('response', response => {
  console.log(`${response.status()} ${response.url()}`);
});
```

## Summary

**E2E Testing Checklist:**
- [ ] Test critical user journeys only
- [ ] Use Page Object Model for maintainability
- [ ] Use stable selectors (data-testid, ARIA roles)
- [ ] Implement authentication state management
- [ ] Configure CI/CD integration
- [ ] Handle async operations properly
- [ ] Mock external dependencies
- [ ] Enable screenshots/videos on failure
- [ ] Use visual regression for UI components
- [ ] Keep tests fast and reliable

**Key Principles:**
1. **Test behavior, not implementation** - Focus on user actions
2. **Keep tests independent** - No dependencies between tests
3. **Use realistic data** - Test with production-like scenarios
4. **Avoid over-testing** - E2E is expensive, test critical paths only
5. **Make tests maintainable** - Use POM and clear naming

**Tool Selection:**
- **Playwright:** Cross-browser, fast, modern features
- **Cypress:** Great DX, time-travel debugging
- **Percy/Chromatic:** Visual regression testing

**Performance:**
- E2E test: 10-60 seconds per test
- Run in parallel for faster CI
- Use authentication state to avoid repeated logins
- Mock external APIs to reduce flakiness

---

[← Integration Testing](./04-integration-testing.md) | [Next: Test-Driven Development →](./06-test-driven-development.md)
