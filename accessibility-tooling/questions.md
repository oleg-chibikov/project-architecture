# Accessibility tooling: likely questions

[Back to the interview notes](./README.md) · [All projects](../README.md)

Short answers to the follow-ups. A line marked "guess" comes from the design,
so check it against what really happened.

## Confirm before the interview

- **Numbers.** Violations at the start and now, teams on board, tickets closed
  on time, CI time added.
- **Requirement.** Who asked for it and the WCAG level.
- **Baseline.** What identifies a violation across runs, and what an axe-core
  upgrade did.
- **Work split.** Who built what among the 4 engineers.
- **Stories.** One real pushback, one missed deadline, one technical failure.

## Why tests

### Why tests over lint, design system rules or a scanner?

Tests already ran on every PR and already blocked CI. A failure points at the
code and the PR that caused it.

| Option | Good at | Misses |
| --- | --- | --- |
| ESLint `jsx-a11y` | Fast and cheap | The rendered page, ARIA set at runtime |
| Design system rules | One fix reaches every consumer | Custom components, how parts combine |
| Manual audit, browser extension | Finds what tools can't | Doesn't scale, runs only when someone remembers |
| Crawler on staging | The real app | Late, gives a URL instead of code |
| Check inside tests (picked) | Runs on every PR, points at code | Untested code |

### How did one codemod handle 4 frameworks?

- **It finds the render step.** `render()` in Testing Library, `cy.mount` in
  Cypress, `page` in Playwright.
- **It adds one assertion after it.** The codemod writes no new tests.
- **Running it twice adds nothing** (guess).
- **Odd tests needed hand fixes.** Custom render wrappers and shared helpers
  (guess). Don't claim 100% automation.

### Was there a pre-commit step?

No (guess). A render plus an axe scan is too slow for a hook. If the
interviewer assumes one, correct them.

## Baseline

### What identifies a violation across runs?

- **File and line break.** Any refactor would lose old violations or invent new
  ones.
- **A sturdier key** (guess). The axe rule id, the axe target selector and the
  component or test.
- **A big DOM change resets it.** That is fine.

### How do you tell a moved violation from a new one?

1. Full key match: the same violation.
2. Rule and component match, selector changed: it probably moved.
3. No match: new.

When unsure, call it new. A missed regression costs more than a few minutes of
re-baselining.

### Why not make teams fix the backlog first?

- **A red day one gets switched off.** Hundreds of failing builds kill trust.
- **Fairness.** A PR fails for what it adds. Old debt goes to the team that owns
  it.

### What stops the baseline from becoming permanent?

Each existing violation gets a ticket with a deadline set by severity. Confirm
what happened past the deadline.

### What happens on an axe-core upgrade?

Confirm. A good answer: pin the version, re-baseline on upgrade and ticket the
new findings, so builds don't all fail at once.

## Tickets and ownership

### How did you dedupe across 4 test types?

The same component can fail the same rule in Jest and in Playwright. Keying by
repo, component path and rule id makes both update one ticket (guess).

### Who owns a violation in a shared component?

EngHealth already routed by code ownership, such as CODEOWNERS (guess). A fix in
a shared component helps every consumer, so it goes to the design system team.

### What happens when a team misses the deadline?

Confirm. Blocking merges would punish the next PR author for old debt. The
likely answer is dashboards and leadership review.

## Rollout and people

### How did you get a blocking gate accepted with no authority over teams?

- **The baseline.** Nothing failed on day one. The message: "Nothing changes
  today. You just can't add new problems."
- **Metrics teams already watch.** Compliance risk and EngHealth scores.
- **A pilot first.** A team with few violations and a keen lead. Confirm which.

### What pushback did you get?

- **"Not my bug."** A consumer's test found a violation in a shared component.
- **Slower CI.** Teams blamed the new scans.
- **Exemptions.** "This code is going away soon." One policy for every request.
- **A real story.** Fill in one with names and the outcome.

### How did you split the work?

- **The split** (guess). Codemod, axe integration with baseline, EngHealth and
  Jira, framework adapters with migration support.
- **What I kept.** Baseline rules and rollout. A wrong baseline loses trust,
  and rollout needs context across teams.

## Limits

### What doesn't axe-core catch?

- **Coverage.** It tests about a third to half of WCAG criteria automatically.
- **Gaps.** Reading order, error message quality, keyboard flow, screen reader
  experience.
- **The answer.** It was the automated floor. Design system components made
  contrast, focus and semantics right by default. Confirm any manual audits.

### What about components with no tests?

They are invisible to the check. Say it as a known limit. The ESLint rule only
makes sure existing test files include the check.

### Iframes and third-party apps?

Jira and Confluence render Connect and Forge apps in iframes, where axe-core is
limited. Likely a gap (guess).

## Compliance

### Where did the requirement come from?

Guess: enterprise and government sales that need a VPAT, the accessibility
report buyers ask for.

- **US federal.** Section 508.
- **EU and UK public sector.** EN 301 549.
- **Australia.** The Disability Discrimination Act.

### What is the difference between A, AA and AAA?

- **A.** The minimum.
- **AA.** What laws and VPATs target.
- **AAA.** Aspirational. Some criteria can't be met for every kind of content.

Say "we targeted WCAG 2.1 AA" once confirmed.

## Impact

### How do you know risk went down?

- **The count.** Fewer violations is an early signal.
- **Stronger proof.** Fewer accessibility complaints, or an external audit
  finding fewer issues.
- **If neither existed.** Say the count was the best signal available.

### What would you change?

- **Baseline matching.** Fewer false new violations after a refactor.
- **Coverage.** Checks beyond what axe-core finds.
- **One backlog.** Manual audit findings in EngHealth next to the automated
  ones.
