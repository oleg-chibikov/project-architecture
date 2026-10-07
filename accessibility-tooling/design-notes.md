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

- **One check in each `describe` block.** Each block sets the component up in
  its own state, such as an open menu. A check in every block covers the most
  interactive states.
- **Setup from `beforeEach`.** When the block has one, the new check runs
  after it.
- **Otherwise setup from the first test.** The codemod copies that test up to
  its first real assertion after a wait. By then the component has rendered and
  settled.
- **Teams write no tests.** They review a diff, so adoption is a migration.

## Baseline

### Why old violations don't fail the build

- **A check that fails on the first day gets deleted.** Teams with hundreds of
  red builds skip or remove the check to get CI green again.
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

- **Builds stay green on the first day.** Nothing breaks for a team that
  adopts the check. It only stops new problems.
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
- **Fewer violations is a first sign.** Proof that real users have it easier
  needs an audit or user reports.

## What I would change

- **Ticket matching.** A key that survives a shifted line, so a refactor
  doesn't churn EngHealth tickets.
- **Coverage.** Checks beyond what axe-core finds.
- **One backlog.** Manual audit findings in EngHealth next to the automated
  ones.
- **Manager scorecards.** One view per team of open violations, overdue tickets
  and suppressed rules. Suppressions already get their own tickets, so the data
  is there. The head of engineering sees which teams skip accessibility and
  follows up with them.
