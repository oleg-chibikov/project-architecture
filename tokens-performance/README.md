# Tokens performance

[Back to all projects](../README.md)

The reasoning behind the main choices is in [design-notes.md](./design-notes.md).

Product pages ship up to 90 KB less gzipped CSS, and LCP on core pages improved
by about 3%. I removed hard-coded fallbacks from design token calls across
several repositories. Codemods made the edits and an ESLint rule stops new
fallbacks.

## Problem

- **Every fallback made its own CSS.** Code called `token('color.text',
  '#172B4D')`. Each different fallback for the same token produced a separate
  CSS rule.
- **Most of that CSS did nothing.** In products with the tokens Babel plugin,
  the plugin ignored the hand-written fallback.
- **Some products still needed fallbacks.** Products without the Babel plugin
  depended on the fallback values.
- **No feature flag to hide behind.** The edits touched code in many products.
  A flag per edit wasn't possible, so a bad edit would reach customers.
- **Fallbacks kept coming back.** Nothing stopped an engineer from adding a new
  one.

## Context

- **Design tokens.** Named values such as `color.text`. `token()` returns a CSS
  variable, so a theme can change the colour.
- **Fallback.** The second argument of `token()`. The browser uses it when the
  theme hasn't set the variable.
- **Tokens Babel plugin.** It rewrites `token()` calls at build time. Most
  products had it, some didn't.
- **Two kinds of consumers.** Products in the monorepo use the tokens package
  from source, so a change reaches them on merge. Products in other
  repositories install a published version.
- **Test tools.** Visual regression tests compare page screenshots. Criterion
  is the company's tool for page performance tests.

## My ownership

- **Found the problem.** A performance review pointed at CSS size on core
  pages. In the CSS bundle I saw the same token rules repeated with different
  fallbacks.
- **Took it on.** Tokens sat with my team, the design system team. No one was
  working on the fallbacks, and the fix looked cheap for the gain.
- **Sold it to management.** I measured the CSS bytes fallbacks added per page
  and estimated the LCP gain. I showed the duplicated CSS on a real page and
  tied the work to the company performance goal. Staged codemods kept the risk
  low.
- **Led the migration.** I planned the stages and wrote the codemod with
  jscodeshift, a tool that edits code through its syntax tree. I compared each
  fallback with its token's real value to decide what the codemod could remove
  on its own. Each product team reviewed and merged the PR for its own code.
- **Locked it in and measured.** I wrote the ESLint rule that blocks new
  fallbacks. CSS analysis scripts and Criterion tests measured the result.

## Architecture

The CSS before and after:

```mermaid
flowchart LR
  subgraph Before["Before: one rule per fallback"]
    B1["token('color.text', '#172B4D')"] --> BR1["CSS rule 1"]
    B2["token('color.text', '#333')"] --> BR2["CSS rule 2"]
    B3["token('color.text', 'black')"] --> BR3["CSS rule 3"]
  end
  subgraph After["After: one rule per token"]
    A1["token('color.text')"] --> AR["One CSS rule"]
  end
  Before --> After
```

How each `token()` call was handled:

```mermaid
flowchart TD
  P["Product"] --> HasPlugin{"Has the tokens<br/>Babel plugin?"}
  HasPlugin -->|no| Add["Stage 1: add the plugin"] --> Call
  HasPlugin -->|yes| Call["token() call<br/>with a fallback"]
  Call --> Same{"Fallback equals<br/>the token value?"}
  Same -->|yes| Sweep["Stage 2: broad codemod sweep"]
  Same -->|no| Group["Stage 3: group by owning team"] --> PR["PR opened by script<br/>one per team"] --> VR["Team checks<br/>visual regression results"] --> Merge["Team merges"]
  Sweep --> Lint["ESLint rule<br/>blocks new fallbacks"]
  Merge --> Lint
```

## Decisions

- **Compare values before removing anything.** A fallback equal to its token
  value changes nothing on screen. A different one can change a colour.
- **Stages over one big run.** The Babel plugin went in first. Safe fallbacks
  went next, and risky ones last.
- **Team PRs for risky fallbacks.** A script grouped these calls by owning team
  and opened a PR for each. The team knows its own pages best.
- **An ESLint rule after the cleanup.** The rule fails lint on a new fallback.
  The CSS stays small after the migration ends.
- **Measure on one full run.** Many teams shipped changes at the same time,
  and the removals landed in many PRs. Production numbers mixed all of it. I
  applied every removal in one separate build that didn't merge. Its bundle
  size against the build before gave the saving. Criterion shows whether
  people get the page faster.

## Trade-offs

- **Removing fallbacks over leaving them.**
  - **Why:** the CSS gets smaller. Left alone, it keeps growing.
  - **Gave up:** leaving them carries no risk. Every page keeps its exact
    colours. Where a fallback differed, the owning team approved the token's
    colour.
- **Staged codemods and team PRs over one codemod for every fallback.**
  - **Why:** safe edits go fast and risky ones get a review.
  - **Gave up:** one codemod finishes in one PR and asks nothing of the
    teams. Team PRs waited on each team's review.
- **An ESLint rule over a one-off cleanup.**
  - **Why:** the CSS stays small after the migration ends.
  - **Gave up:** an engineer could pass a fallback for a quick fix.
- **Visual tests over checking each page by hand.**
  - **Why:** a test catches a changed colour on every run.
  - **Gave up:** a person looks at every screen, with or without a test.
    Screens with no test rely on the team's review.

## Delivery

1. **Analyse.** I compared every fallback with its token's value and split
   the calls into safe and risky.
2. **Add the Babel plugin.** Products that lacked it got it first. Unit tests
   ran without the plugin too, and some asserted fallback values. Turning the
   plugin on for tests meant updating many of them to expect the token.
3. **Sweep safe fallbacks.** The codemod removed fallbacks equal to the token
   value in broad runs.
4. **Team PRs for the rest.** A script grouped risky calls by owning team and
   opened the PRs.
5. **Verify the monorepo, release a major.** I checked each of the 5 to 7
   products in the monorepo before merging. Products outside it got the
   change in a new major version.
6. **Lock it in and measure.** The ESLint rule went on. A separate full run
   with every removal gave the bundle size saving. Criterion measured LCP.

## Risk

- **No feature flag.** Each edit went straight to production. Stages, visual
  tests and team review took the place of a flag.
- **Hidden reliance on fallbacks.** Some pages showed the fallback in place of
  the token. Visual regression tests caught them.
- **Products without the plugin.** Removing their fallbacks first could break
  them. The plugin went in before any of their fallbacks went out.
- **Regrowth.** New fallbacks would bring the CSS back. The ESLint rule blocks
  them.
- **Two kinds of consumers.** Monorepo products get a change the moment it
  merges. I checked each of them first. Products in other repositories saw no
  change until they upgraded to the new major.

## Validation

- **Visual regression tests.** Screenshots before and after each stage.
- **Babel plugin checks.** Each product had the plugin before its fallbacks
  went.
- **Team review.** Each team checked and merged its own PR.
- **CSS analysis scripts.** Gzipped CSS size of one full run with every
  removal, against the build before.
- **Criterion tests.** LCP on core product pages.

## Metrics

| Metric | Result |
| --- | --- |
| CSS size | Up to 90 KB less, gzipped |
| LCP on core product pages | About 3% better |
| Broken experiences | None |

## Failure

- **I assumed a fallback was dead code wherever a theme loaded.** Visual
  regression tests showed two gaps.
- **Some products didn't use theming.** Their pages showed fallback values
  only.
- **Some kinds of tokens didn't apply.** Those always showed the fallback.
- **The cost.** Comparing values in the code wasn't enough. The plan had to
  check what each page showed too.

## Lessons

- **Check what the page shows.** The code said the fallbacks were dead. The
  screenshots said some were live.
- **Sort edits by risk.** Safe edits go in bulk. Risky ones go to the team that
  owns the code.
- **Without a flag, stage it.** Small steps with checks between them replace
  the off switch.
- **Close the door after the cleanup.** A lint rule keeps the gain.
- **Measure what people feel.** Bytes saved matter when LCP moves.

---

[Design notes](./design-notes.md) · [All projects](../README.md)
