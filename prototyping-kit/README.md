# Prototyping kit: interview notes

The full architecture, phase by phase, is in [detailed.md](./detailed.md).

## The pitch

A designer or PM asks Claude for a screen. The kit builds a working prototype
from the real design system, checks it, and shares a link. An engineer can lift
the code instead of starting from scratch.

## The big picture

```mermaid
flowchart LR
  Author["Designer or PM<br/>asks for a screen"] --> Kit["Claude + prototyping kit"]
  Kit --> Proto["Working prototype<br/>real design system, sandbox app"]
  Proto --> Share["Draft PR + Sandpit link"]
  Share --> Eng["Engineer<br/>keep, rework or rewrite"]
  DS["Design system<br/>tokens, components, page layouts"] --> Kit
  Guard["Guardrails<br/>checks, gates, self-review"] --> Kit
  Kit -.->|what slowed it down| Jira["Findings in Jira"]
  Jira -.-> Team["Platform team<br/>improves the kit"]
  Team -.-> Kit
```

## How a run goes

The author answers a few questions. Claude does the rest and pings the Mac
whenever it needs them.

```mermaid
flowchart TD
  Ask["Ask<br/>screenshot, Figma or text"] --> Scope["Claude restates the goal<br/>author confirms"]
  Scope --> Mode["Author picks where it runs<br/>local or Sandpit, mock or real data"]
  Mode --> Layout["Author picks a standard layout"]
  Layout --> Build["Claude builds it<br/>in its own copy of the repo"]
  Build --> Check["Claude checks itself<br/>against the design, clicks through,<br/>runs the tests"]
  Check --> Decide{"Author decides"}
  Decide -->|change something| Build
  Decide -->|share| PR["Draft PR + Sandpit link"]
  Decide -->|keep local| Done["Handover"]
  PR --> Done
  Done --> Eng["Engineer review"]
```

## Design tokens pipeline

Figma changes reach the code with no one copying values or guessing a version.

```mermaid
flowchart LR
  Figma["Designers publish<br/>in Figma"] --> Sync["Sync job<br/>daily or on publish"]
  Sync --> SyncPR["PR with the changes<br/>and the bump it will cause"]
  SyncPR -->|human merges| Bump{"Pipeline compares<br/>the token list<br/>with the last release"}
  Bump -->|removed or renamed| Major["major"]
  Bump -->|added or visible change| Minor["minor"]
  Bump -->|invisible fix| Patch["patch"]
  Bump -->|no change| NoRel["no release"]
  Label["PR label<br/>for a dramatic restyle"] -.-> Major
  Major & Minor & Patch --> Npm["design-tokens<br/>npm package"]
  Npm --> WP["web-platform daily bump<br/>generates CSS and DESIGN.md tokens"]
  WP --> Agent["Claude builds with them"]
```

## The story

- **Situation.** Prototypes from Figma got thrown away, and engineers rebuilt
  each screen. An AI agent left alone made up colours and stopped halfway.
- **Task.** Let a non-engineer get a prototype in production-quality code on
  the design system.
- **Action.** Put Claude on rails: one step at a time, the author approving
  each decision, and every step checked by code. Gave it the design system as
  rules it can read. Made the agent report where the kit slowed it down.
- **Action, tokens.** Replaced hand-copied CSS and hand-picked versions with a
  pipeline that syncs from Figma and picks the version itself.
- **Result.** Your numbers here: prototypes built, code engineers kept, findings
  fixed.

## Decisions I'd defend

- **Sandbox only.** A product app is another team's code and release. Moving a
  prototype there is engineering work for later.
- **Code over prompts.** When the agent got a step wrong twice, the step moved
  from instructions into a script. Scripts behave the same every run.
- **The design system wins.** A colour the system lacks snaps to the nearest
  token, and Claude tells the author.
- **The machine picks the version.** Almost every token PR is a Figma sync, so
  a person choosing patch or minor was guessing. A human label covers the one
  case code can't judge: a restyle that keeps every name.
- **Visible changes are minor.** Tokens are a visual system. A restyle shipped
  as a patch would reach apps unseen.

## Questions they may ask

- **What was hardest?** Keeping the agent going. It ended turns mid-run, so
  hooks block the stop until something will wake it.
- **How do you know the output is good?** Claude compares its screenshot with
  the design, runs the product's lint and tests, and the engineer rates each
  part keep, rework or rewrite.
- **How does it get better?** Each run files what slowed it down. The platform
  team fixes those, so the next run hits fewer walls.
- **Why not changesets for tokens?** Changesets asks a person per PR. Here the
  diff of the token list already says what changed.
