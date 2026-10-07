# Prototyping kit: detailed architecture

[Back to the interview notes](./README.md) · [All projects](../README.md)

An author asks Claude for a screen. The kit builds it in the sandbox app from
the real design system, checks it, and shares a draft PR with a Sandpit link.

## Architecture

The orchestrator decides, the CLI executes and keeps state, hooks stop the
model from cutting corners.

```mermaid
flowchart LR
  Author["Author<br/>Claude Desktop or Claude Code"] --> Plugin["prototype-launcher plugin<br/>finds or clones the repo,<br/>moves the session into it"]
  Plugin --> Orch

  subgraph SkillBox["Skill prototype"]
    Orch["Orchestrator session<br/>asks the author, launches phases,<br/>carries reports to the handover"]
    subgraph Phases["Phase subagents"]
      Endpoints["endpoints<br/>finds API methods for real data"]
      FigmaPh["figma<br/>reads the frame, saves a PNG"]
      Build["build<br/>writes the screen"]
      Verify["verify<br/>browser checks, small fixes"]
      CIPh["ci<br/>fixes red PR checks"]
    end
    Orch -->|launches| Phases
  end

  subgraph CLIBox["CLI: scripts prototype"]
    Sync["sync, jira-login<br/>clean main, Atlassian sign-in"]
    Start["start, resume<br/>worktree, tool checks, reopen a run"]
    Scaffold["scaffold<br/>folder, PROTOTYPE.md, input picture"]
    Browser["browser, dev, connect<br/>dev server on a free port,<br/>Sandpit link-up"]
    Checks["compare, verify<br/>fidelity passes, typecheck,<br/>lint, tests, i18n"]
    Publish["ask-publish, publish<br/>draft PR, preview link"]
    NoteCmd["note, file, handover<br/>findings, close the run"]
  end
  Orch -->|one command per step| CLIBox

  subgraph Bg["Background jobs"]
    Install["pnpm install"]
    Layouts["page pattern screenshots"]
    Icons["icon sheets"]
  end
  Start --> Bg

  subgraph StateBox["State"]
    WT["worktree + branch<br/>prototype/name"]
    Run["run.json<br/>local cache of the run"]
    PMD["PROTOTYPE.md + -screenshots<br/>committed record"]
    NJ["notes.json<br/>findings"]
  end
  CLIBox --> StateBox

  subgraph HooksBox["Hooks: .claude/hooks"]
    TurnGate["turn-gate<br/>no stop while nothing<br/>will wake the session"]
    PubGate["publish-gate<br/>ask about publishing after verify"]
    PatGate["pattern-screenshots<br/>layouts before the question"]
    Routers["browser and dev-server routers"]
    Move["session-move<br/>Desktop into the worktree"]
    Notify["notify<br/>Mac notification"]
  end
  Run -.->|read| HooksBox
  HooksBox -.->|block or wake| Orch

  Phases --> DSSkill["skill design-system<br/>+ DESIGN.md, tokens.yaml"]
  Build --> Patterns["shift-ui-patterns Page<br/>standard page layouts"]
  Layouts --> Patterns
  Publish --> GH["GitHub draft PR<br/>+ Sandpit preview link"]
  NoteCmd --> Jira["Jira sub-tasks<br/>under EX-6185"]
  Notify --> Mac["macOS notifications"]
```

- **Worktree.** Each prototype gets its own folder and branch beside the main
  checkout, so prototypes don't mix and the main checkout stays untouched.
- **Page patterns.** Every screen is `Page` from `shift-ui-patterns`. The author
  picks a layout from screenshots before the build.
- **Notifications.** The Mac pings the author when Claude needs an answer, stops
  on an error, finishes the build, puts up the PR or ends the run.

## The run, step by step

```mermaid
flowchart TD
  A1["1. Ask<br/>screenshot, Figma link,<br/>Claude Design export or text"]
  A1 --> A2["2. Ready the repo<br/>launcher finds or clones web-platform,<br/>sync pulls main, jira-login opens Atlassian"]
  A2 --> A3["3. Scope<br/>Claude sums up user, task, success, route.<br/>Author confirms"]
  A3 --> A4{"4. Run mode<br/>author picks"}
  A4 -->|"Local page, mock data"| B["Dev server and browser<br/>start in the background"]
  A4 -->|"Sandpit, mock data"| B
  A4 -->|"Sandpit, real data"| B
  B -.->|Sandpit| SI["Author signs in.<br/>connect links Sandpit<br/>to the local app"]
  A4 -.->|real data| EP["endpoints phase<br/>finds API methods<br/>per screen part"]
  EP -.->|no API for a part| MK["Author: mock that part?"]
  A1 -.->|Figma input| FG["figma phase<br/>reads the frame"]
  A4 --> A5["5. Worktree<br/>own folder and branch,<br/>session moves there"]
  A5 --> A6["6. Layout<br/>pattern screenshots + live Sandpit links.<br/>Author picks one or Custom"]
  A6 --> A7["7. build phase<br/>writes the screen from Page + shift-ui"]
  A7 --> A8["8. verify phase<br/>fidelity loop, scenario clicks, every state,<br/>typecheck, lint, tests"]
  A8 --> A9{"9. Author decides"}
  A9 -->|Amendments| A7
  A9 -->|Create PR| A10["10. Publish<br/>draft PR + Sandpit preview link.<br/>ci phase watches the checks"]
  A9 -->|Keep local| A11
  A10 --> A11["11. Handover<br/>local link, data source,<br/>design adaptations, folder"]
  A11 --> A12["12. Engineer review<br/>keep, rework or rewrite each part"]
  A11 -.->|change later| R["resume in the worktree"]
  R --> A7
```

## Self-diagnostics

The agent writes down every place the kit got in its way. The findings reach
the web platform team as Jira sub-tasks.

```mermaid
flowchart LR
  Wall["Agent hits a wall<br/>workaround, broken instruction,<br/>missing tool, design system gap"] --> Note["note<br/>title, hit, did, fix, pictures"]
  Note --> NJ["notes.json<br/>committed with the prototype"]
  Note -->|background, one at a time| File["file through twg"]
  File --> Jira["Jira sub-task under EX-6185.<br/>Same title adds a comment"]
  Jira --> Team["Web platform team<br/>fixes the kit"]
  Team --> Next["Next run hits fewer walls"]
  NJ -->|filing failed| HO["handover retries"]
  HO --> File
```

## Design system and tokens

```mermaid
flowchart LR
  Figma["Figma variables<br/>Mitti"] --> Pkg["design-tokens repo<br/>publishes the npm package"]
  Pkg --> Gen["design-tokens-generator<br/>daily bump in web-platform"]
  Gen --> CSS["mfe-tailwind-config CSS<br/>classes, light and dark"]
  Gen --> TY["tokens.yaml<br/>grep a hex to find the class"]
  TY --> MD["DESIGN.md<br/>MUST, SHOULD, MAY rules"]
  MD --> DSS["skill design-system<br/>icon lookup, self-audit"]
  DSS --> Ph["build, verify, ci phases"]
  Ph --> Code["prototype code<br/>Page + shift-ui + token classes"]
  CSS --> Code
  Lint["ESLint + i18n rules"] -->|verify fails on a break| Code
```
