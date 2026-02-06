---
name: bdd-playwright
description: BDD with Gherkin feature files and Playwright step definitions using playwright-bdd. Enforces ARIA-first locators, axe-core accessibility audits, and accessible test patterns. Use when writing end-to-end tests, acceptance tests, or feature files for web projects.
---

# BDD with Playwright

Write Gherkin feature files with Playwright step definitions using `playwright-bdd`. All locators MUST use ARIA-first strategy. All web scenarios MUST include accessibility assertions.

## When to Use

- Writing acceptance tests or end-to-end tests for web projects
- User says "BDD", "Gherkin", "feature file", "acceptance test", or "step definitions"
- Adding tests to an existing Playwright project
- Testing user-facing behavior (not implementation details)

## Stack

| Tool | Purpose |
|------|---------|
| `playwright-bdd` | Gherkin → Playwright test generation |
| `@playwright/test` | Test runner |
| `@axe-core/playwright` | Automated accessibility audits |
| `playwright` CLI | Codegen, trace viewer, debugging |

## Setup

```bash
npm i -D @playwright/test playwright-bdd @axe-core/playwright
npx playwright install
```

**playwright.config.ts:**
```typescript
import { defineConfig } from '@playwright/test';
import { defineBddConfig } from 'playwright-bdd';

const testDir = defineBddConfig({
  features: 'features/**/*.feature',
  steps: 'steps/**/*.ts',
});

export default defineConfig({
  testDir,
  use: { baseURL: 'http://localhost:3000' },
});
```

## Project Structure

```
features/           # Gherkin .feature files
  login.feature
  checkout.feature
steps/              # Step definitions (ARIA locators only)
  common.steps.ts   # Shared steps (navigation, a11y)
  login.steps.ts
  checkout.steps.ts
```

## Feature File Conventions

```gherkin
Feature: User login

  Background:
    Given I am on the login page

  Scenario: Successful login with valid credentials
    When I fill in "Email" with "user@example.com"
    And I fill in "Password" with "secret123"
    And I click the "Sign in" button
    Then I should see the "Dashboard" heading
    And the page should have no accessibility violations

  Scenario: Show error for invalid credentials
    When I fill in "Email" with "wrong@example.com"
    And I fill in "Password" with "bad"
    And I click the "Sign in" button
    Then I should see "Invalid email or password" alert
```

**Rules:**
- Steps reference elements by **visible label or role**, never by selector
- Every scenario with UI interaction ends with an accessibility check
- Background sets up shared preconditions
- Scenarios describe user behavior, not implementation

## Step Definitions --- ARIA-First Locators

```typescript
import { expect } from '@playwright/test';
import { createBdd } from 'playwright-bdd';
import AxeBuilder from '@axe-core/playwright';

const { Given, When, Then } = createBdd();

// --- Navigation ---

Given('I am on the {word} page', async ({ page }, pageName: string) => {
  const routes: Record<string, string> = {
    login: '/login', home: '/', checkout: '/checkout',
  };
  await page.goto(routes[pageName] ?? `/${pageName}`);
});

Given('I navigate to {string}', async ({ page }, url: string) => {
  await page.goto(url);
});

// --- Interaction (ARIA locators ONLY) ---

When('I click the {string} button', async ({ page }, name: string) => {
  await page.getByRole('button', { name }).click();
});

When('I click the {string} link', async ({ page }, name: string) => {
  await page.getByRole('link', { name }).click();
});

When('I fill in {string} with {string}',
  async ({ page }, label: string, value: string) => {
    await page.getByLabel(label).fill(value);
  }
);

When('I check {string}', async ({ page }, label: string) => {
  await page.getByLabel(label).check();
});

When('I select {string} from {string}',
  async ({ page }, option: string, label: string) => {
    await page.getByLabel(label).selectOption(option);
  }
);

// --- Assertions (user-visible behavior) ---

Then('I should see the {string} heading', async ({ page }, name: string) => {
  await expect(page.getByRole('heading', { name })).toBeVisible();
});

Then('I should see {string} alert', async ({ page }, text: string) => {
  await expect(page.getByRole('alert').filter({ hasText: text })).toBeVisible();
});

Then('I should see {string}', async ({ page }, text: string) => {
  await expect(page.getByText(text)).toBeVisible();
});

Then('the {string} button should be disabled', async ({ page }, name: string) => {
  await expect(page.getByRole('button', { name })).toBeDisabled();
});

// --- Accessibility (mandatory for web scenarios) ---

Then('the page should have no accessibility violations', async ({ page }) => {
  const results = await new AxeBuilder({ page })
    .withTags(['wcag2a', 'wcag2aa'])
    .analyze();
  expect(results.violations).toEqual([]);
});

Then('the {string} region should have no accessibility violations',
  async ({ page }, role: string) => {
    const results = await new AxeBuilder({ page })
      .include(`[role="${role}"]`)
      .withTags(['wcag2a', 'wcag2aa'])
      .analyze();
    expect(results.violations).toEqual([]);
  }
);
```

## Locator Priority (Non-Negotiable)

| Priority | Locator | Use For |
|----------|---------|---------|
| 1st | `getByRole('button', { name })` | All interactive elements |
| 2nd | `getByLabel('Email')` | Form inputs |
| 3rd | `getByPlaceholder('Search...')` | Inputs without labels (fix the label!) |
| 4th | `getByText('Welcome')` | Static text content |
| 5th | `getByAltText('Logo')` | Images |
| 6th | `getByTitle('Close')` | Title attributes |
| Last | `getByTestId('widget')` | Only when ARIA isn't specific enough |

**NEVER use:**
- XPath (`//div[@class="foo"]/button[1]`)
- Deep CSS (`#app > div > div.main > button:nth-child(2)`)
- CSS classes as locators (`.btn-primary`)
- Position-based (`.nth(2)`, `:nth-child(3)`)
- `page.locator('css=...')` for user-facing elements

## Using Playwright CLI

**Generate step definitions from user interaction:**
```bash
# Record interactions — generates accessible locators by default
npx playwright codegen http://localhost:3000

# Record with specific viewport
npx playwright codegen --viewport-size=1280,720 http://localhost:3000

# Record for mobile
npx playwright codegen --device="iPhone 13" http://localhost:3000
```

**Debug failing tests:**
```bash
# Step through test with inspector
npx playwright test --debug

# Debug a specific feature
npx playwright test features/login --debug

# Open trace viewer for failed test
npx playwright show-trace trace.zip
```

**Run with UI mode for visual debugging:**
```bash
npx playwright test --ui
```

**View test report:**
```bash
npx playwright show-report
```

**Tip:** When writing new step definitions, use `npx playwright codegen` to discover the right ARIA locators interactively, then adapt the generated code into step definitions. Codegen defaults to accessible locators.

## Chaining Locators for Context

When multiple elements share the same role/name, narrow with parent context:

```typescript
// Button inside a specific dialog
await page.getByRole('dialog', { name: 'Confirm' })
  .getByRole('button', { name: 'Delete' }).click();

// Link inside navigation
await page.getByRole('navigation')
  .getByRole('link', { name: 'Settings' }).click();

// Input inside a specific form section
await page.getByRole('group', { name: 'Billing' })
  .getByLabel('ZIP code').fill('12345');
```

## Accessibility Testing Pattern

Every web project should include a shared accessibility step and a dedicated feature:

```gherkin
Feature: Accessibility compliance

  Scenario Outline: <page> meets WCAG 2.1 AA
    Given I navigate to "<url>"
    Then the page should have no accessibility violations

    Examples:
      | page      | url        |
      | Home      | /          |
      | Login     | /login     |
      | Dashboard | /dashboard |
      | Settings  | /settings  |
```

## Anti-Patterns to Catch in Review

| Pattern | Problem | Fix |
|---------|---------|-----|
| `page.locator('#login-form > div:nth-child(2) > input')` | Brittle CSS chain | `page.getByLabel('Password')` |
| `page.locator('xpath=//button[1]')` | XPath breaks with DOM changes | `page.getByRole('button', { name: '...' })` |
| `page.locator('.btn-primary')` | Class names change | `page.getByRole('button', { name: '...' })` |
| `page.waitForTimeout(3000)` | Arbitrary wait | Let Playwright auto-wait |
| `page.getByTestId('x')` without ARIA | No accessibility benefit | Add `aria-label`, use `getByRole` |
| `page.getByRole('button').first()` | Ambiguous, position-dependent | Add `{ name: '...' }` |
| Feature steps with selectors | Gherkin should be human-readable | Reference labels and roles |

## Workflow

1. Write `.feature` file with Gherkin scenarios (business language)
2. Use `npx playwright codegen` to discover ARIA locators interactively
3. Implement step definitions with ARIA-first locators
4. Add `Then the page should have no accessibility violations` to scenarios
5. Run: `npx playwright test`
6. Debug failures: `npx playwright test --debug` or `--ui`
7. View traces: `npx playwright show-trace` / `npx playwright show-report`
