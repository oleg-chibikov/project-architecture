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
| Crawler on staging | The real app | Late, gives a URL instead of code |
| Check inside tests (picked) | Runs on every PR, points at code | Untested code |

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

### Matching violations across runs

- **File and line are too fragile.** A refactor would lose old violations or
  invent new ones.
- **The two mistakes cost differently.** A missed regression ships a real
  problem. A false new violation costs the author a few minutes.

## Getting teams on board

- **Day one stays green.** Nothing changes for a team today. It just can't add
  new problems.
- **Old debt has an owner.** EngHealth puts each existing violation on the
  owning team's backlog, with a deadline by severity.
- **Lint keeps the check in.** An ESLint rule flags a relevant test file with
  no check.

## Limits

- **axe-core covers part of WCAG.** It can't judge reading order, error message
  quality, keyboard flow or the screen reader experience. Those still need a
  person.
- **Untested code is invisible.** The ESLint rule only makes sure existing test
  files include the check.
- **A count is a proxy.** Fewer violations is an early signal. Proof of a better
  experience needs an audit or fewer user complaints.

## What I would change

- **Baseline matching.** Fewer false new violations after a refactor.
- **Coverage.** Checks beyond what axe-core finds.
- **One backlog.** Manual audit findings in EngHealth next to the automated
  ones.
