# Test Management - Small Business

## Overview

Testing for small businesses should be practical, automated, and provide immediate value. The focus is on building confidence in deployments while keeping testing overhead minimal and cost-effective.

## Testing Strategy for Small Teams

### Testing Pyramid for Small Business

```
        🔺 E2E Tests (Few)
       ───────────────────
      🔺🔺 Integration Tests (Some)
     ───────────────────────────────
   🔺🔺🔺🔺 Unit Tests (Many)
  ─────────────────────────────────────
```

#### Unit Tests (Foundation)
- **Goal**: Test individual functions and components
- **Coverage**: Aim for 70-80% code coverage
- **Tools**: Jest, Mocha, pytest, JUnit
- **Automation**: Run on every commit

#### Integration Tests (Middle Layer)
- **Goal**: Test component interactions
- **Coverage**: Critical user workflows
- **Tools**: Supertest, TestContainers, Spring Boot Test
- **Automation**: Run on pull requests

#### End-to-End Tests (Top Layer)
- **Goal**: Test complete user journeys
- **Coverage**: 3-5 critical business flows
- **Tools**: Playwright, Cypress, Selenium
- **Automation**: Run before production deployment

## Automated Testing Implementation

### Phase 1: Unit Testing Foundation

#### Setting Up Unit Tests

**JavaScript/Node.js Example:**
```javascript
// package.json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  },
  "devDependencies": {
    "jest": "^29.0.0"
  }
}

// jest.config.js
module.exports = {
  testEnvironment: 'node',
  collectCoverageFrom: [
    'src/**/*.js',
    '!src/index.js'
  ],
  coverageThreshold: {
    global: {
      branches: 75,
      functions: 75,
      lines: 75,
      statements: 75
    }
  }
};
```

#### GitHub Actions Integration
```yaml
name: Test Suite

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
    - run: npm install
    - run: npm test
```

### Phase 2: Integration Testing

#### API Testing Example
```javascript
// Integration test example with Supertest
const request = require('supertest');
const app = require('../src/app');

describe('User API', () => {
  test('POST /users creates a new user', async () => {
    const userData = {
      name: 'John Doe',
      email: 'john@example.com'
    };

    const response = await request(app)
      .post('/users')
      .send(userData)
      .expect(201);

    expect(response.body.name).toBe(userData.name);
    expect(response.body.email).toBe(userData.email);
  });
});
```

### Phase 3: End-to-End Testing

#### Playwright Setup (Recommended for Small Teams)
```javascript
// playwright.config.js
module.exports = {
  testDir: './tests/e2e',
  use: {
    baseURL: 'http://localhost:3000',
    headless: true,
    screenshot: 'only-on-failure',
    video: 'retain-on-failure'
  }
};

// tests/e2e/user-journey.spec.js
const { test, expect } = require('@playwright/test');

test('user can sign up and log in', async ({ page }) => {
  await page.goto('/signup');
  await page.fill('[data-testid=email]', 'test@example.com');
  await page.fill('[data-testid=password]', 'password123');
  await page.click('[data-testid=signup-button]');
  
  await expect(page).toHaveURL('/dashboard');
  await expect(page.locator('h1')).toContainText('Welcome');
});
```

## Testing Tools for Small Businesses

### Free/Low-Cost Options

#### Unit Testing
- **Jest** (JavaScript): Comprehensive testing framework
- **pytest** (Python): Simple, powerful testing
- **JUnit** (Java): Standard Java testing
- **RSpec** (Ruby): Behavior-driven testing

#### E2E Testing
- **Playwright**: Modern, fast browser automation
- **Cypress**: Developer-friendly E2E testing
- **Selenium**: Mature cross-browser testing

#### Performance Testing
- **Artillery**: Simple load testing
- **k6**: Developer-centric performance testing

## Testing Metrics to Track

#### Quality Metrics
- **Test Coverage**: Aim for 70-80%
- **Test Pass Rate**: Target >95%
- **Bug Escape Rate**: Bugs found in production
- **Test Execution Time**: Keep under 10 minutes

#### Business Metrics
- **Feature Quality**: Customer-reported issues per feature
- **Release Confidence**: Team confidence in releases
- **Time to Fix**: Average time to resolve issues

## Best Practices

### ✅ Recommendations
1. **Test business logic first**: Focus on what matters most
2. **Keep tests simple**: One assertion per test when possible
3. **Use descriptive test names**: Tests should tell a story
4. **Maintain test code**: Refactor tests like production code
5. **Review test failures**: Don't ignore failing tests

### Budget Planning
- **Time Investment**: 1-2 weeks initial setup, 30 min daily maintenance
- **Tool Costs**: $0-300/month depending on needs
- **ROI**: 60-80% fewer production issues, faster deployments

## Next Steps

With testing practices in place, continue to:
[Building DevOps Culture →](../06-Culture/)

---
*[← Previous: CI/CD](../04-CI-CD/) | [Back to Small Business Guide](../README.md)*
