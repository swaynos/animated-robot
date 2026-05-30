# Project Pitch: Playwright Flow Compiler for OpenCode

## One-line pitch

An OpenCode skill that uses Playwright MCP as an exploratory browser layer to discover user flows, then compiles those discoveries into deterministic Playwright tests or scripts that can run reliably without an LLM.

## The problem

Browser automation is often split between two weak extremes.

Traditional browser automation scripts are deterministic and CI-friendly, but they are expensive to author and brittle when the app changes. AI-driven browser agents are flexible and fast at exploration, but they are often too nondeterministic for production regression testing or repeatable workflow automation.

Teams want the best of both worlds:

- Let an agent explore an unfamiliar web flow.
- Let the agent identify labels, roles, navigation states, forms, assertions, and edge cases.
- Convert that exploration into explicit Playwright code.
- Run the final script deterministically in CI, local development, or scheduled automation.

The missing layer is a structured workflow that keeps AI in the authoring phase and removes it from the runtime phase.

## The idea

Create an OpenCode skill called `playwright-flow-compiler`.

The skill teaches OpenCode to use Playwright MCP only as an exploratory and discovery tool. After exploration, the skill requires the agent to produce normal Playwright code using stable locators, explicit assertions, environment-driven configuration, and no MCP dependency at runtime.

The guiding principle is simple:

```text
Explore flexibly. Run deterministically.
```

## Target users

This project is for developers, QA engineers, platform teams, and automation-heavy product teams who:

- Already use or want to use Playwright.
- Want AI-assisted test authoring without AI-controlled runtime execution.
- Need to convert manual user flows into reliable test scripts.
- Maintain end-to-end tests for rapidly changing products.
- Want browser automation that can be reviewed, committed, and run in CI.

## Why OpenCode

OpenCode skills are a natural fit because this is not just a library problem. It is a repeatable agent workflow problem.

The agent needs clear rules for:

- When to use Playwright MCP.
- What to observe during exploration.
- How to record a user flow.
- How to choose stable Playwright locators.
- How to generate deterministic test code.
- How to harden the result.
- What not to leave in the final runtime path.

An OpenCode skill can encode those rules as reusable project behavior.

## Core workflow

```text
User describes a flow
        ↓
OpenCode invokes the skill
        ↓
Playwright MCP explores the app
        ↓
The agent records a structured flow plan
        ↓
The agent generates deterministic Playwright code
        ↓
The script is run and hardened
        ↓
The final output is committed as normal test code
```

## Example user request

```text
Use Playwright to automate the signup flow on my local app.
Start at http://localhost:3000, create a test account, verify the onboarding page appears, and turn it into a Playwright test.
```

## Expected agent behavior

The skill should guide OpenCode to:

1. Inspect the app with Playwright MCP.
2. Discover the signup path.
3. Record the required inputs, controls, transitions, and success criteria.
4. Identify stable locators such as roles, labels, placeholder text, and test IDs.
5. Generate a Playwright test using `@playwright/test`.
6. Run the test if possible.
7. Fix brittle selectors.
8. Report what changed and how to run it.

## What the project produces

The project produces a reusable OpenCode skill, plus optional starter files, for turning exploratory browser sessions into deterministic Playwright automation.

Suggested repository structure:

```text
playwright-flow-compiler/
  README.md
  skills/
    playwright-flow-compiler/
      SKILL.md
  examples/
    opencode.json
    generated-test.example.ts
    flow-plan.example.md
  templates/
    playwright-test.template.ts
    flow-plan.template.md
```

## Skill behavior

The skill enforces a five-phase process.

### Phase 1: Understand the target flow

The agent identifies:

- Starting URL
- User goal
- Required credentials or test data
- Expected success condition
- Whether output should be a test, script, fixture, or helper
- Whether the target is local, staging, or production
- Any destructive actions to avoid

The agent should continue with safe assumptions unless credentials or destructive actions are involved.

### Phase 2: Explore with Playwright MCP

The agent uses Playwright MCP to inspect and perform the flow interactively.

During exploration, the agent records:

- Pages visited
- Forms completed
- Buttons and links clicked
- Navigation changes
- Modals, toasts, validation messages, and loading states
- Candidate locators
- Assertions that prove success
- Failure states or edge cases

MCP is used here because exploration benefits from flexibility.

### Phase 3: Write a flow plan

The agent creates a structured plan before generating code.

```md
## Flow Plan

### Goal

...

### Preconditions

...

### Test Data

...

### Steps

1. ...
2. ...
3. ...

### Candidate Locators

- ...

### Assertions

- ...

### Risks

- ...

### Open Questions

- ...
```

This plan is the bridge between nondeterministic exploration and deterministic implementation.

### Phase 4: Compile to deterministic Playwright

The agent converts the plan into Playwright code.

Locator priority:

1. `page.getByRole(...)`
2. `page.getByLabel(...)`
3. `page.getByPlaceholder(...)`
4. `page.getByText(...)`, only when the text is stable
5. `page.getByTestId(...)`, if the project already uses test IDs
6. CSS or XPath only as a last resort

The final code should use explicit assertions, avoid arbitrary sleeps, and run without Playwright MCP.

Example output:

```ts
import { test, expect } from '@playwright/test';

test('user can complete signup', async ({ page }) => {
  await page.goto('/signup');

  await page.getByLabel(/email/i).fill(`test-${Date.now()}@example.com`);
  await page.getByLabel(/password/i).fill(process.env.TEST_USER_PASSWORD!);
  await page.getByRole('button', { name: /create account/i }).click();

  await expect(page.getByRole('heading', { name: /welcome/i })).toBeVisible();
});
```

### Phase 5: Harden and report

After generating the script, the agent should:

- Run the test if the environment allows it.
- Fix locator failures.
- Replace brittle selectors.
- Add intermediate assertions.
- Extract repeated login or setup logic into helpers.
- Use environment variables for credentials and base URLs.
- Avoid hardcoded secrets.
- Avoid destructive actions against production.

The final report should include:

- Flow discovered
- Files created or changed
- How to run the test
- Assumptions made
- Remaining risks
- Whether MCP is still needed at runtime

The ideal final state:

```text
Playwright MCP used during authoring: yes
Playwright MCP needed during runtime: no
```

## What this is not

This project is not a general-purpose AI browser agent.

It should not:

- Leave LLM-driven browser actions in the committed test path.
- Depend on MCP during CI runs.
- Treat screenshots as the main source of truth when accessibility snapshots are available.
- Generate brittle CSS-heavy scripts by default.
- Perform destructive production actions without explicit approval.
- Store credentials in source code.

## MVP scope

The MVP is a single OpenCode skill that can be copied into a repository and used immediately.

MVP deliverables:

- `SKILL.md` for `playwright-flow-compiler`
- Example `opencode.json` MCP configuration
- Example flow plan
- Example generated Playwright test
- README with installation and usage instructions

MVP success criteria:

- A user can describe a web flow in natural language.
- OpenCode uses Playwright MCP to explore the app.
- OpenCode creates a written flow plan.
- OpenCode generates a deterministic Playwright test.
- The final test does not require MCP or an LLM at runtime.

## Stretch goals

Future versions could add:

- A CLI wrapper for invoking the workflow.
- Flow plan diffing between app versions.
- Automatic selector stability scoring.
- Suggested `data-testid` improvements.
- Trace-based failure analysis.
- Test flake reports.
- Integration with GitHub Actions.
- Support for authenticated storage state setup.
- Support for converting exploratory flows into reusable page objects or fixtures.

## Example OpenCode configuration

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "playwright": {
      "type": "local",
      "command": ["npx", "@playwright/mcp@latest"],
      "enabled": true
    }
  }
}
```

## Example skill file

```md
---
name: playwright-flow-compiler
description: Explore a browser user flow with Playwright MCP, then convert the discovered path into deterministic Playwright tests or scripts. Use this when asked to automate, test, debug, or codify a web user flow.
---

# Playwright Flow Compiler

## Purpose

Use Playwright MCP as an exploratory layer only. Do not leave MCP or LLM-driven browser actions in the final runtime path unless the user explicitly asks for adaptive automation.

The final deliverable should be deterministic Playwright code that can run without an LLM.

## Core Principle

Exploration is allowed to be flexible. Runtime must be explicit.

Use MCP to discover:

- Pages involved
- User-visible labels
- Stable locators
- Required inputs
- Navigation transitions
- Network or UI wait conditions
- Assertions that prove the flow succeeded
- Edge cases and failure states

Then translate the discovered behavior into:

- Playwright `test()` specs for testing, or
- Playwright scripts for task automation

## Workflow

### Phase 1: Understand the target flow

Before using the browser, identify:

- The starting URL
- The user goal
- Required credentials or test data
- Expected success condition
- Whether the output should be a test, script, fixture, or helper
- Whether the app is local, staging, or production
- Any destructive actions to avoid

If information is missing, make the safest reasonable assumption and continue unless credentials or destructive actions are involved.

### Phase 2: Explore with Playwright MCP

Use Playwright MCP to inspect and perform the flow interactively.

During exploration:

- Prefer accessibility snapshots over screenshots.
- Prefer semantic interactions such as roles, names, labels, and visible text.
- Record each meaningful step.
- Note candidate locators.
- Note alternative locators if the best locator may be unstable.
- Observe loading states, redirects, modals, toasts, validation messages, and final success UI.
- Avoid relying on brittle CSS classes, generated IDs, dynamic timestamps, or text that is likely to change.
- Do not optimize for the fewest browser actions during exploration. Optimize for understanding the page.

Capture findings in this structure:

```md
## Flow Plan

### Goal
...

### Preconditions
...

### Test Data
...

### Steps
1. ...
2. ...
3. ...

### Candidate Locators
- ...

### Assertions
- ...

### Risks
- ...

### Open Questions
- ...
```

### Phase 3: Convert to deterministic Playwright

Create or update Playwright code that can run without MCP.

Use this priority order for locators:

1. `page.getByRole(...)`
2. `page.getByLabel(...)`
3. `page.getByPlaceholder(...)`
4. `page.getByText(...)`, only when text is stable
5. `page.getByTestId(...)`, if the project already uses test IDs
6. CSS or XPath only as a last resort

Use explicit assertions instead of arbitrary sleeps.

Prefer:

```ts
await expect(page.getByRole('heading', { name: /dashboard/i })).toBeVisible();
```

Avoid:

```ts
await page.waitForTimeout(3000);
```

Use Playwright auto-waiting and assertions wherever possible.

### Phase 4: Harden the script

After writing the deterministic script:

- Run the test or script if possible.
- Fix locator failures.
- Add assertions for intermediate and final states.
- Extract repeated login/setup flows into helpers or fixtures.
- Use environment variables for credentials and base URLs.
- Avoid hardcoding secrets.
- Avoid performing destructive actions against production.
- Add comments only where they explain non-obvious behavior.

### Phase 5: Report results

When done, report:

- What flow was discovered
- What files were created or changed
- How to run the deterministic script
- Any remaining assumptions or risks
- Whether MCP is still needed at runtime

The ideal final state is:

```text
Playwright MCP used during authoring: yes
Playwright MCP needed during runtime: no
```

## Output Standards

For Playwright tests:

- Use `@playwright/test`
- Use `test.describe` when grouping related flows
- Use `expect` assertions
- Prefer fixtures for auth/setup
- Keep tests independent
- Make base URL configurable

## Anti-patterns

Do not:

- Keep LLM or MCP calls in the final deterministic Playwright test.
- Use screenshots as the primary source of truth when accessibility snapshots are available.
- Use `waitForTimeout` except as a last resort with a clear explanation.
- Use brittle generated selectors when semantic locators are available.
- Modify production data without explicit user approval.
- Store credentials in source files.
- Treat a single successful exploratory run as sufficient hardening.
```

## Positioning

This project is a compiler pattern for AI-assisted browser automation.

It treats Playwright MCP as the exploratory frontend and deterministic Playwright as the compiled backend.

That makes the project easy to explain:

```text
It lets AI discover how a user flow works, then turns that discovery into boring, reliable Playwright code.
```

## Why now

LLM browser tooling is becoming good enough to explore real applications, but most teams still do not want nondeterministic agents controlling critical runtime checks.

This project gives teams a practical middle path:

- Use AI where it is strong: exploration, interpretation, and first-draft generation.
- Use deterministic code where it is strong: review, repeatability, CI, and long-term maintenance.

## Success metric

The project succeeds when a developer can point OpenCode at a web flow and get back a Playwright test that is:

- Readable
- Reviewable
- Committable
- CI-ready
- Free of runtime LLM dependency

## Final tagline

Playwright Flow Compiler turns AI-assisted browser exploration into deterministic Playwright automation.
