# Accessibility tooling: design notes

[Back to the overview](./README.md) · [All projects](../README.md)

The reasoning behind the main choices, and the limits they leave.

## Why a check inside tests

Tests already ran on every PR and already blocked CI. A failure points at the
code and the PR that caused it.

| Option | Good at | Misses |
| --- | --- | --- |
| ESLint `jsx-a11y` | Fast and cheap | The rendered page, ARIA set at runtime |
| Design system rules | One fix reaches every consumer | Custom components, how parts combine |
| Manual audit, browser extension | Finds what tools can't | Doesn't scale, runs only when someone remembers |
| Crawler on the live site | The real app | Finds problems after release, misses states behind clicks |
| Check inside tests (picked) | Runs on every PR, points at code | Untested code |

### Why no crawler on the live site

- **Too late.** The crawler finds a problem after release. By then the PR author
  has moved on, and the fix waits in a backlog.
- **Misses interactions.** A dialog shows only after a click. A crawler needs a
  script for each path, and the tests already make those clicks.
- **No pointer to code.** The crawler reports a URL. Someone still has to find
  the component.

## How the codemod works

- **It reuses each test.** The assertion goes into the existing test, after the
  component renders.
- **It writes no new tests.** `toBeAccessible` is one line in each framework.
- **Adoption is a migration.** Teams review a diff instead of writing tests.

## Baseline

### Why old violations don't fail the build

- **A red day one gets switched off.** Hundreds of failing builds kill trust in
  the gate.
- **Fairness.** A PR fails for what it adds. Old debt goes to the team that owns
  it, with a deadline.

### Finding the component behind a violation

- **The test file points at the wrong place.** A violation inside a shared
  button shows up in every test that renders the button.
- **React Fiber knows who rendered the node.** The wrapper walks up the Fiber
  tree from the failing DOM node to its component.
- **Duplicates collapse.** Many test failures from one design system component
  become one ticket.
- **Ownership is right.** The ticket goes to the team that owns the component.
  The teams whose tests caught it get nothing to fix.

### Matching violations across runs

- **The baseline sits in the test.** The `violationsBaseline` parameter lists
  the violations that test already has. Moving code around the file leaves it
  as it is.
- **EngHealth matches on file, line and issue type.** The file and line come
  from the component. A shifted line can close one ticket and open another.
- **The two mistakes cost differently.** A missed regression ships a real
  problem. A false new violation costs the author a few minutes.

## Getting teams on board

- **Day one stays green.** Nothing changes for a team today. It just can't add
  new problems.
- **Old debt has an owner.** EngHealth puts each existing violation on the
  owning team's backlog, with a deadline by severity.
- **Lint keeps the check in.** An ESLint rule flags a relevant test file with
  no check. It starts as a warning.
- **Suppressions ratchet.** A package not on board yet suppresses the rule.
  Each suppression gets an EngHealth ticket, so it doesn't stay forgotten.

## Limits

- **axe-core covers part of WCAG.** It can't judge reading order, error message
  quality, keyboard flow or the screen reader experience. Those still need a
  person.
- **Untested code is invisible.** The ESLint rule only makes sure existing test
  files include the check.
- **A count is a proxy.** Fewer violations is an early signal. Proof of a better
  experience needs an audit or fewer user complaints.

## What I would change

- **Ticket matching.** A key that survives a shifted line, so a refactor
  doesn't churn EngHealth tickets.
- **Coverage.** Checks beyond what axe-core finds.
- **One backlog.** Manual audit findings in EngHealth next to the automated
  ones.
