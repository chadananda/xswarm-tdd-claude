# TDD Protocol — Strict Red-Green-Refactor

## Rules

1. **Never write implementation without a failing test.** A failing test proves the requirement exists and that the implementation will satisfy it. Without this, you're guessing.
2. **One test at a time.** Keeps the feedback loop tight and isolates failures to a single behavior. Multiple tests at once obscure which change broke what.
3. **Run tests at every phase.** Assumptions about test outcomes are unreliable — execution is proof. Never say "this would fail" — show it.
4. **Minimum code to pass.** Over-engineering makes later refactoring harder and tests less meaningful. Don't anticipate future tests; triangulate via successive tests.
5. **When a test fails, fix the code — not the test.** Changing tests to match broken code hides bugs. Only change a test if the expectation itself was wrong, and state why before changing it.
6. **Refactoring changes structure, not behavior.** Behavioral changes without corresponding test updates create silent regressions. If a test breaks during refactor, undo.
7. **Report each phase explicitly.** Explicit reporting creates an audit trail and catches skipped steps:
   - 🔴 RED: test name + failure output
   - 🟢 GREEN: what changed + all tests pass
   - 🔵 REFACTOR: what improved + all tests pass

## Anti-Patterns — Hard No

- **Writing test + implementation in the same step** — can't verify the test actually detects failure if you never see it fail.
- **Hardcoding returns to pass multiple tests** — passes tests without solving the actual problem; creates false confidence.
- **Writing tests after implementation** — post-hoc tests rationalize existing code instead of specifying behavior.
- **Batch-writing tests then implementing** — loses the red-green feedback loop that guides incremental design.
- **Skipping test execution** — unexecuted tests provide zero information; they're just comments.

## BDD Conventions

- `describe` = subject or scenario context; `it` = "it should [behavior]"
- **Name tests by observable behavior, not implementation details.** Implementation-coupled names break when code is refactored even though behavior hasn't changed.
- Structure test bodies: `// given` → `// when` → `// then`
- **One logical assertion per test; tests must be independent and deterministic.** Multiple assertions obscure which behavior failed; dependent tests create cascading false failures.

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

**Spike code is throwaway.** Learn from it, delete it, start fresh with TDD. Promoting spike code bypasses the test-driven design process, producing untested code with accidental complexity baked in.