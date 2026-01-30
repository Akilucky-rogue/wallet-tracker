# Wallet Tracker - Testing Plan

## Overview

This document outlines the comprehensive testing strategy for the Wallet Tracker application, including unit tests, integration tests, end-to-end tests, and quality assurance processes.

**Testing Goal**: Achieve 80%+ code coverage with robust quality assurance.

---

## Table of Contents

1. [Testing Pyramid](#testing-pyramid)
2. [Unit Testing](#unit-testing)
3. [Integration Testing](#integration-testing)
4. [End-to-End Testing](#end-to-end-testing)
5. [Performance Testing](#performance-testing)
6. [Security Testing](#security-testing)
7. [Accessibility Testing](#accessibility-testing)
8. [Mobile Testing](#mobile-testing)
9. [Bug Tracking](#bug-tracking)
10. [Test Automation](#test-automation)

---

## Testing Pyramid

```
         /\
        /  \
       / E2E \       (10-20 tests)
      /______\
     /        \
    / Integr.  \    (30-50 tests)
   /____________\
  /              \
 /     Units      \  (200-300 tests)
/__________________\
```

---

## Unit Testing

### Overview
Unit tests verify individual components, functions, and methods in isolation.

**Target Coverage**: 80%+  
**Tools**: Jest, Mocha, or similar

### 1. Data Model Tests

**File**: `tests/unit/models/*.test.js`

#### Transaction Model
```javascript
describe('Transaction Model', () => {
  
  test('should create transaction with valid data', () => {
    const transaction = new Transaction({
      amount: 50.00,
      direction: 'EXPENSE',
      category: 'Groceries',
      date: new Date('2026-01-30')
    });
    
    expect(transaction.id).toBeDefined();
    expect(transaction.amount).toBe(50.00);
    expect(transaction.isDeleted).toBe(false);
  });
  
  test('should reject invalid amount', () => {
    expect(() => {
      new Transaction({
        amount: -50.00,
        direction: 'EXPENSE',
        category: 'Groceries',
        date: new Date()
      });
    }).toThrow('Amount must be greater than 0');
  });
  
  test('should reject future date', () => {
    const futureDate = new Date();
    futureDate.setDate(futureDate.getDate() + 1);
    
    expect(() => {
      new Transaction({
        amount: 50.00,
        direction: 'EXPENSE',
        category: 'Groceries',
        date: futureDate
      });
    }).toThrow('Date cannot be in the future');
  });
  
  test('should require sourceId for income', () => {
    expect(() => {
      new Transaction({
        amount: 1000.00,
        direction: 'INCOME',
        category: 'Salary',
        date: new Date(),
        sourceId: null
      });
    }).toThrow('Income transactions require a source');
  });
});
```

#### IncomeSource Model
```javascript
describe('IncomeSource Model', () => {
  
  test('should create income source', () => {
    const source = new IncomeSource({
      name: 'Salary',
      type: 'SALARY'
    });
    
    expect(source.name).toBe('Salary');
    expect(source.isActive).toBe(true);
    expect(source.totalReceived).toBe(0);
  });
  
  test('should enforce unique names', () => {
    const source1 = new IncomeSource({ name: 'Salary', type: 'SALARY' });
    
    expect(() => {
      new IncomeSource({ name: 'Salary', type: 'FREELANCE' });
    }).toThrow('Name already exists');
  });
  
  test('should validate type enum', () => {
    expect(() => {
      new IncomeSource({ name: 'Test', type: 'INVALID' });
    }).toThrow('Invalid type');
  });
});
```

### 2. Validation Tests

**File**: `tests/unit/validators/*.test.js`

```javascript
describe('Transaction Validator', () => {
  
  test('should validate required fields', () => {
    const errors = validateTransaction({});
    
    expect(errors).toContainEqual(
      expect.objectContaining({ field: 'amount' })
    );
    expect(errors).toContainEqual(
      expect.objectContaining({ field: 'direction' })
    );
  });
  
  test('should validate amount is positive decimal', () => {
    const errors = validateTransaction({
      amount: 'invalid',
      direction: 'EXPENSE',
      category: 'Food',
      date: '2026-01-30'
    });
    
    expect(errors).toContainEqual(
      expect.objectContaining({ 
        field: 'amount',
        message: 'Must be a positive number with up to 2 decimals'
      })
    );
  });
  
  test('should validate direction enum', () => {
    const errors = validateTransaction({
      amount: 50,
      direction: 'INVALID',
      category: 'Food',
      date: '2026-01-30'
    });
    
    expect(errors.length).toBeGreaterThan(0);
  });
});
```

### 3. Calculation Logic Tests

**File**: `tests/unit/calculations/*.test.js`

```javascript
describe('Income Stability Calculator', () => {
  
  test('should calculate stability score for consistent income', () => {
    const monthlyAmounts = [5000, 5000, 5000, 5000, 5000];
    const stability = calculateStability(monthlyAmounts);
    
    expect(stability).toBe('STABLE');
    expect(stability.score).toBeGreaterThan(0.9);
  });
  
  test('should calculate stability for volatile income', () => {
    const monthlyAmounts = [1000, 5000, 2000, 8000, 500];
    const stability = calculateStability(monthlyAmounts);
    
    expect(stability).toBe('VOLATILE');
    expect(stability.score).toBeLessThan(0.5);
  });
});

describe('Category Distribution Calculator', () => {
  
  test('should calculate category percentages', () => {
    const transactions = [
      { amount: 40, category: 'Groceries' },
      { amount: 60, category: 'Utilities' }
    ];
    
    const distribution = calculateCategoryDistribution(transactions);
    
    expect(distribution[0].percentage).toBe(40);
    expect(distribution[1].percentage).toBe(60);
  });
});
```

### Test Checklist - Unit Tests
- [ ] All model validations tested
- [ ] All calculation functions tested
- [ ] Error cases covered
- [ ] Edge cases handled (zero values, null, empty strings)
- [ ] Date/time handling correct
- [ ] Enum validations working
- [ ] Math calculations accurate

---

## Integration Testing

### Overview
Integration tests verify that multiple components work together correctly.

**Target**: 30-50 tests  
**Tools**: Jest with database, Supertest for APIs

### 1. Database Integration Tests

**File**: `tests/integration/database/*.test.js`

```javascript
describe('Transaction Repository', () => {
  let db;
  
  beforeAll(async () => {
    db = await setupTestDatabase();
  });
  
  afterEach(async () => {
    await db.clearAll();
  });
  
  afterAll(async () => {
    await db.close();
  });
  
  test('should save and retrieve transaction', async () => {
    const transaction = {
      amount: 50.00,
      direction: 'EXPENSE',
      category: 'Groceries',
      date: new Date('2026-01-30')
    };
    
    const saved = await TransactionRepository.create(transaction);
    const retrieved = await TransactionRepository.findById(saved.id);
    
    expect(retrieved.amount).toBe(50.00);
    expect(retrieved.category).toBe('Groceries');
  });
  
  test('should filter transactions by date range', async () => {
    await TransactionRepository.create({
      amount: 50,
      direction: 'EXPENSE',
      category: 'Groceries',
      date: new Date('2026-01-15')
    });
    
    await TransactionRepository.create({
      amount: 100,
      direction: 'EXPENSE',
      category: 'Utilities',
      date: new Date('2026-01-25')
    });
    
    const results = await TransactionRepository.findByDateRange(
      new Date('2026-01-20'),
      new Date('2026-01-31')
    );
    
    expect(results).toHaveLength(1);
    expect(results[0].category).toBe('Utilities');
  });
  
  test('should update transaction', async () => {
    const original = await TransactionRepository.create({
      amount: 50,
      direction: 'EXPENSE',
      category: 'Groceries',
      date: new Date()
    });
    
    await TransactionRepository.update(original.id, {
      amount: 75,
      category: 'Food'
    });
    
    const updated = await TransactionRepository.findById(original.id);
    
    expect(updated.amount).toBe(75);
    expect(updated.category).toBe('Food');
  });
  
  test('should soft delete transaction', async () => {
    const transaction = await TransactionRepository.create({
      amount: 50,
      direction: 'EXPENSE',
      category: 'Groceries',
      date: new Date()
    });
    
    await TransactionRepository.delete(transaction.id);
    
    const deleted = await TransactionRepository.findById(transaction.id);
    expect(deleted.isDeleted).toBe(true);
  });
});
```

### 2. API Integration Tests

**File**: `tests/integration/api/*.test.js`

```javascript
describe('Transaction API', () => {
  let app;
  
  beforeAll(() => {
    app = createTestApp();
  });
  
  test('POST /transactions should create transaction', async () => {
    const response = await request(app)
      .post('/api/v1/transactions')
      .set('Authorization', `Bearer ${testToken}`)
      .send({
        amount: 50.00,
        direction: 'EXPENSE',
        category: 'Groceries',
        date: '2026-01-30'
      });
    
    expect(response.status).toBe(201);
    expect(response.body.success).toBe(true);
    expect(response.body.data.id).toBeDefined();
  });
  
  test('GET /transactions should filter correctly', async () => {
    // Create test transactions
    await request(app)
      .post('/api/v1/transactions')
      .set('Authorization', `Bearer ${testToken}`)
      .send({
        amount: 50,
        direction: 'EXPENSE',
        category: 'Groceries',
        date: '2026-01-15'
      });
    
    const response = await request(app)
      .get('/api/v1/transactions')
      .set('Authorization', `Bearer ${testToken}`)
      .query({
        fromDate: '2026-01-01',
        toDate: '2026-01-31',
        direction: 'EXPENSE'
      });
    
    expect(response.status).toBe(200);
    expect(response.body.data.transactions).toHaveLength(1);
  });
  
  test('POST /transactions should validate required fields', async () => {
    const response = await request(app)
      .post('/api/v1/transactions')
      .set('Authorization', `Bearer ${testToken}`)
      .send({
        direction: 'EXPENSE',
        category: 'Groceries'
        // Missing amount and date
      });
    
    expect(response.status).toBe(400);
    expect(response.body.success).toBe(false);
    expect(response.body.error.details.length).toBeGreaterThan(0);
  });
});
```

### 3. PDF Import Integration Tests

**File**: `tests/integration/import/*.test.js`

```javascript
describe('PDF Import Flow', () => {
  
  test('should parse IDFC PDF correctly', async () => {
    const pdfPath = 'tests/fixtures/statement_jan_2026.pdf';
    const parsed = await parsePDFStatement(pdfPath, 'IDFC');
    
    expect(parsed.transactions).toHaveLength(45);
    expect(parsed.periodStart).toEqual(new Date('2026-01-01'));
    expect(parsed.totalCredit).toBe(8000);
    expect(parsed.totalDebit).toBe(5000);
  });
  
  test('should detect duplicates during import', async () => {
    // Create existing transaction
    await TransactionRepository.create({
      amount: 5000,
      direction: 'INCOME',
      date: new Date('2026-01-28')
    });
    
    const pdfPath = 'tests/fixtures/statement_jan_2026.pdf';
    const result = await importStatement(pdfPath, 'IDFC');
    
    expect(result.duplicates).toHaveLength(1);
    expect(result.newTransactions).toHaveLength(44);
  });
  
  test('should apply ignore rules during import', async () => {
    // Create ignore rule
    await IgnoreRuleRepository.create({
      matchText: 'Interest',
      scope: 'FUTURE_IMPORTS',
      isEnabled: true
    });
    
    const pdfPath = 'tests/fixtures/statement_jan_2026.pdf';
    const result = await importStatement(pdfPath, 'IDFC');
    
    expect(result.ignored.length).toBeGreaterThan(0);
  });
});
```

### Test Checklist - Integration Tests
- [ ] Database CRUD operations working
- [ ] Relationships and constraints enforced
- [ ] API endpoints returning correct status codes
- [ ] Request/response formats correct
- [ ] Validation errors handled properly
- [ ] Pagination working
- [ ] PDF parsing accurate
- [ ] Ignore rules applied correctly

---

## End-to-End Testing

### Overview
E2E tests verify complete user workflows through the UI.

**Target**: 10-20 tests  
**Tools**: Cypress or Playwright

### 1. User Workflows

**File**: `tests/e2e/workflows.spec.js`

```javascript
describe('Complete Transaction Entry Workflow', () => {
  
  beforeEach(() => {
    cy.visit('http://localhost:3000');
    cy.login();
  });
  
  it('should add transaction from form', () => {
    // Navigate to add transaction
    cy.contains('Add Entry').click();
    
    // Fill form
    cy.get('[data-testid="amount-input"]').type('50.00');
    cy.get('[data-testid="direction-expense"]').click();
    cy.get('[data-testid="category-select"]').select('Groceries');
    cy.get('[data-testid="date-input"]').clear().type('01/30/2026');
    cy.get('[data-testid="notes-input"]').type('Weekly shopping');
    
    // Submit
    cy.get('[data-testid="save-button"]').click();
    
    // Verify success
    cy.contains('Transaction saved').should('be.visible');
    cy.get('[data-testid="form"]').should('have.value', '');
  });
});

describe('Complete PDF Import Workflow', () => {
  
  beforeEach(() => {
    cy.visit('http://localhost:3000');
    cy.login();
  });
  
  it('should import PDF statement end-to-end', () => {
    // Step 1: Upload
    cy.contains('Import Statement').click();
    cy.get('[data-testid="file-input"]').attachFile('statement_jan_2026.pdf');
    cy.contains('Next: Preview').click();
    
    // Step 2: Preview
    cy.contains('45 transactions found').should('be.visible');
    cy.contains('Next: Check Duplicates').click();
    
    // Step 3: Duplicates
    cy.contains('44 transactions are new').should('be.visible');
    cy.contains('Next: Apply Rules').click();
    
    // Step 4: Rules
    cy.contains('43 transactions to import').should('be.visible');
    cy.contains('Next: Confirm Import').click();
    
    // Step 5: Confirm
    cy.contains('Confirm Import').click();
    
    // Verify success
    cy.contains('Import successful!').should('be.visible');
    cy.contains('43 transactions imported').should('be.visible');
  });
});
```

### 2. User Interface Workflows

```javascript
describe('Dashboard Interactions', () => {
  
  it('should navigate between months', () => {
    cy.visit('http://localhost:3000/dashboard');
    
    cy.contains('January 2026').should('be.visible');
    cy.get('[data-testid="next-month-button"]').click();
    cy.contains('February 2026').should('be.visible');
  });
  
  it('should filter transactions', () => {
    cy.visit('http://localhost:3000/transactions');
    
    // Apply filters
    cy.get('[data-testid="category-filter"]').select('Groceries');
    cy.get('[data-testid="direction-filter"]').select('EXPENSE');
    
    // Verify filtered results
    cy.get('[data-testid="transaction-list"]')
      .should('contain', 'Groceries');
  });
});

describe('Analytics Charts', () => {
  
  it('should display and update charts', () => {
    cy.visit('http://localhost:3000/analytics/monthly');
    
    // Verify chart elements exist
    cy.get('[data-testid="daily-spend-chart"]').should('be.visible');
    cy.get('[data-testid="month-comparison-chart"]').should('be.visible');
    
    // Change month and verify update
    cy.get('[data-testid="month-selector"]').select('2025-12');
    cy.get('[data-testid="daily-spend-chart"]').should('be.visible');
  });
});
```

### Test Checklist - E2E Tests
- [ ] Complete workflows execute end-to-end
- [ ] UI updates correctly
- [ ] Data persists across navigation
- [ ] Error handling works in UI
- [ ] Forms validate and display errors
- [ ] Charts render and update
- [ ] Mobile navigation works
- [ ] Accessibility features work

---

## Performance Testing

### Load Testing

**File**: `tests/performance/load.test.js`

```javascript
describe('Performance: Load Testing', () => {
  
  test('should load 1000 transactions in < 2 seconds', async () => {
    const startTime = Date.now();
    
    const response = await fetch(
      'http://localhost:5000/api/v1/transactions?limit=1000'
    );
    
    const endTime = Date.now();
    const loadTime = endTime - startTime;
    
    expect(loadTime).toBeLessThan(2000);
    expect(response.status).toBe(200);
  });
  
  test('should render chart with 500 data points in < 1 second', () => {
    const startTime = performance.now();
    
    const chart = renderChart({
      dataPoints: generateDataPoints(500)
    });
    
    const endTime = performance.now();
    
    expect(endTime - startTime).toBeLessThan(1000);
  });
});
```

### Response Time Testing

```javascript
describe('Performance: Response Times', () => {
  
  test('GET /transactions should respond in < 500ms', async () => {
    const start = Date.now();
    const response = await fetch('http://localhost:5000/api/v1/transactions');
    const duration = Date.now() - start;
    
    expect(duration).toBeLessThan(500);
  });
  
  test('POST /transactions should respond in < 300ms', async () => {
    const start = Date.now();
    const response = await fetch('http://localhost:5000/api/v1/transactions', {
      method: 'POST',
      body: JSON.stringify(testTransaction)
    });
    const duration = Date.now() - start;
    
    expect(duration).toBeLessThan(300);
  });
});
```

### Performance Checklist
- [ ] Page load time < 2 seconds
- [ ] API response time < 500ms
- [ ] Chart rendering < 1 second
- [ ] No memory leaks
- [ ] No unused dependencies
- [ ] Images optimized
- [ ] Code minified in production
- [ ] Database queries optimized

---

## Security Testing

### Input Validation Testing

```javascript
describe('Security: Input Validation', () => {
  
  test('should reject SQL injection attempts', async () => {
    const response = await request(app)
      .post('/api/v1/transactions')
      .send({
        amount: "50; DROP TABLE transactions;",
        direction: 'EXPENSE',
        category: 'Groceries',
        date: '2026-01-30'
      });
    
    expect(response.status).toBe(400);
  });
  
  test('should reject XSS attempts', async () => {
    const response = await request(app)
      .post('/api/v1/transactions')
      .send({
        amount: 50,
        direction: 'EXPENSE',
        category: '<script>alert("XSS")</script>',
        date: '2026-01-30'
      });
    
    expect(response.status).toBe(400);
  });
  
  test('should sanitize output', () => {
    const unsafeData = {
      category: '<img src=x onerror="alert(\'XSS\')">'
    };
    
    const sanitized = sanitizeForDisplay(unsafeData);
    expect(sanitized.category).not.toContain('<script>');
  });
});
```

### Authentication & Authorization

```javascript
describe('Security: Authentication', () => {
  
  test('should reject requests without token', async () => {
    const response = await request(app)
      .get('/api/v1/transactions');
    
    expect(response.status).toBe(401);
  });
  
  test('should reject invalid tokens', async () => {
    const response = await request(app)
      .get('/api/v1/transactions')
      .set('Authorization', 'Bearer invalid_token');
    
    expect(response.status).toBe(401);
  });
});
```

### Security Checklist
- [ ] SQL injection prevented
- [ ] XSS prevention enabled
- [ ] CSRF tokens working
- [ ] Input validation strict
- [ ] Output encoding correct
- [ ] Secrets not exposed
- [ ] HTTPS enforced
- [ ] No sensitive data logged

---

## Accessibility Testing

### WCAG 2.1 Compliance

**File**: `tests/accessibility/wcag.test.js`

```javascript
describe('Accessibility: WCAG 2.1 AA', () => {
  
  test('should have proper heading hierarchy', () => {
    cy.visit('http://localhost:3000/dashboard');
    
    cy.get('h1').should('exist');
    cy.get('h1').then($h1 => {
      const h1Count = $h1.length;
      expect(h1Count).toBe(1); // Only one h1
    });
  });
  
  test('should have sufficient color contrast', () => {
    cy.visit('http://localhost:3000/dashboard');
    
    cy.get('body *').each($el => {
      const contrast = getContrastRatio($el);
      expect(contrast).toBeGreaterThan(4.5); // WCAG AA
    });
  });
  
  test('should have alt text for images', () => {
    cy.visit('http://localhost:3000/dashboard');
    
    cy.get('img').each($img => {
      expect($img).to.have.attr('alt');
    });
  });
  
  test('should be keyboard navigable', () => {
    cy.visit('http://localhost:3000');
    
    // Tab through interactive elements
    cy.get('body').tab();
    cy.focused().should('have.attr', 'tabindex', '0');
  });
});
```

### Accessibility Checklist
- [ ] WCAG 2.1 Level AA compliant
- [ ] Color contrast 4.5:1
- [ ] Keyboard navigation works
- [ ] Screen reader compatible
- [ ] Focus indicators visible
- [ ] Form labels associated
- [ ] Error messages clear
- [ ] Touch targets >= 48px

---

## Mobile Testing

### Responsive Design Testing

```javascript
describe('Responsive Design', () => {
  
  const viewports = [
    { width: 375, height: 667, name: 'iPhone' },
    { width: 768, height: 1024, name: 'iPad' },
    { width: 1920, height: 1080, name: 'Desktop' }
  ];
  
  viewports.forEach(({ width, height, name }) => {
    describe(`on ${name} (${width}x${height})`, () => {
      
      beforeEach(() => {
        cy.viewport(width, height);
        cy.visit('http://localhost:3000/dashboard');
      });
      
      it('should display layout correctly', () => {
        cy.get('[data-testid="dashboard"]').should('be.visible');
        cy.get('[data-testid="sidebar"]').should('be.visible');
      });
      
      it('should render navigation properly', () => {
        cy.get('nav').should('be.visible');
        if (width < 768) {
          cy.get('[data-testid="mobile-menu"]').should('be.visible');
        }
      });
    });
  });
});
```

### Touch Interaction Testing

```javascript
describe('Mobile: Touch Interactions', () => {
  
  beforeEach(() => {
    cy.viewport('iphone-x');
    cy.visit('http://localhost:3000');
  });
  
  it('should handle swipe gestures', () => {
    cy.get('[data-testid="month-slider"]').swipe('right');
    cy.contains('December 2025').should('be.visible');
  });
  
  it('should handle long press', () => {
    cy.get('[data-testid="transaction-item"]')
      .first()
      .longPress();
    
    cy.get('[data-testid="context-menu"]').should('be.visible');
  });
});
```

### Mobile Checklist
- [ ] Responsive at 375px, 768px, 1920px
- [ ] Touch targets >= 48px
- [ ] No horizontal scroll (mobile)
- [ ] Readable text at mobile size
- [ ] Images scale properly
- [ ] Navigation accessible on mobile
- [ ] Forms usable on mobile
- [ ] Performance acceptable on 3G

---

## Bug Tracking

### Bug Report Template

```markdown
## Bug Report

**Title**: [Component] Brief description

**Severity**: Critical | High | Medium | Low

**Environment**:
- Browser: Chrome 96
- OS: Windows 10
- Device: Desktop

**Steps to Reproduce**:
1. ...
2. ...
3. ...

**Expected Result**:
...

**Actual Result**:
...

**Screenshots/Video**:
[Attach if possible]

**Logs**:
```
[Error messages or console logs]
```

**Test Case**:
[Link to test case if applicable]
```

### Bug Priority Matrix

| Severity | Impact | Priority |
|----------|--------|----------|
| Critical | Functionality broken | P0 - Fix immediately |
| High | Major feature impacted | P1 - Fix this sprint |
| Medium | Minor feature impacted | P2 - Fix next sprint |
| Low | Cosmetic/Nice-to-have | P3 - Fix when possible |

---

## Test Automation

### Continuous Integration

**File**: `.github/workflows/test.yml`

```yaml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:13
        options: --health-cmd pg_isready
    
    steps:
      - uses: actions/checkout@v2
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linter
        run: npm run lint
      
      - name: Run unit tests
        run: npm run test:unit
      
      - name: Run integration tests
        run: npm run test:integration
      
      - name: Run E2E tests
        run: npm run test:e2e
      
      - name: Upload coverage
        run: npm run coverage
```

### Test Scripts

**In `package.json`**:
```json
{
  "scripts": {
    "test": "jest",
    "test:unit": "jest --testPathPattern=unit",
    "test:integration": "jest --testPathPattern=integration",
    "test:e2e": "cypress run",
    "test:watch": "jest --watch",
    "coverage": "jest --coverage",
    "test:performance": "jest --testPathPattern=performance",
    "test:security": "npm audit && npm run test:unit -- --testPathPattern=security"
  }
}
```

---

## Test Execution Schedule

### Daily (On Commit)
- Unit tests
- Linting
- Quick smoke tests

### Weekly (Monday Morning)
- Full test suite
- Performance tests
- Security scans

### Monthly (First Friday)
- Full E2E test suite
- Load testing
- Accessibility audit

---

## Success Criteria

- ✅ **Code Coverage**: > 80%
- ✅ **Test Pass Rate**: 100%
- ✅ **CI/CD Pipeline**: Green
- ✅ **No Critical Bugs**: In production
- ✅ **Performance**: All metrics met
- ✅ **Security**: No vulnerabilities

---
