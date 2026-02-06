# TDD Protocol — Strict Red-Green-Refactor

## Rules

1. **Never write implementation without a failing test.** No exceptions.
2. **One test at a time.** Write one test, see it fail, make it pass, refactor. Then next test.
3. **Run tests at every phase.** Never say "this would fail" — execute and show output.
4. **Minimum code to pass.** Don't anticipate future tests. Triangulate via successive tests.
5. **When a test fails, fix the code — not the test.** Only change a test if the expectation itself was wrong, and state why before changing it.
6. **Refactoring changes structure, not behavior.** If a test breaks during refactor, undo.
7. **Report each phase explicitly:**
   - 🔴 RED: test name + failure output
   - 🟢 GREEN: what changed + all tests pass
   - 🔵 REFACTOR: what improved + all tests pass

## Anti-Patterns — Hard No

- Writing test + implementation in the same step
- Hardcoding returns to pass multiple tests
- Writing tests after implementation
- Batch-writing tests then implementing
- Skipping test execution

## BDD Conventions

- `describe` = subject or scenario context; `it` = "it should [behavior]"
- Name tests by observable behavior, not implementation details
- Structure test bodies: `// given` → `// when` → `// then`
- One logical assertion per test; tests must be independent and deterministic

### Web Projects: Gherkin + Playwright

For web projects, use Gherkin feature files with Playwright step definitions:
- Feature files describe user behavior in business language
- Step definitions use ARIA-first locators (`getByRole`, `getByLabel`, `getByText`)
- Never use XPath, deep CSS, or class-name selectors in step definitions
- Every scenario includes an accessibility assertion (axe-core WCAG 2.1 AA)
- See `bdd-playwright` skill for full patterns and examples

## Workflow

1. Restate requirement → identify smallest testable increments
2. Outer loop: write failing acceptance test (stays red across multiple unit cycles)
3. Inner loop: 🔴→🟢→🔵 at unit level until acceptance test goes green
4. Refactor the whole; run full suite; stay green

## Spikes

Spike code is throwaway. Learn from it, delete it, start fresh with TDD. Never promote spike code.