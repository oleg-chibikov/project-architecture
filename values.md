# Values questions

Short answers, each backed by a project. Projects:
[prototyping kit](./prototyping-kit/README.md),
[accessibility tooling](./accessibility-tooling/README.md),
[UI settings](./ui-settings-platform/README.md),
[build pipeline](./build-pipeline/README.md),
[tokens](./tokens-performance/README.md).

## Customer outcomes

- **Customer outcome first?** UI settings. People set their theme again in each
  app. I picked a shared backend store over a cookie or localStorage, so one
  choice reaches every app and device. An SSR preload shows the right theme on
  first paint.
- **How do you know what customers need?** I write down what the person should
  see end to end, then check it with the people closest to them. Product teams'
  feedback moved the onboarding from a modal to a spotlight.
- **Convenience against outcome?** I take on complexity when it removes a
  problem people feel. SettingsService needed replication and deletion. A
  cookie needed neither and wouldn't follow the person.
- **Another example?** Accessibility. A broken interface shuts people out of
  the product, so a CI gate was worth the friction.

## Ambiguity

- **An ambiguous task?** UI settings started open: which apps, per app or per
  workspace, how to onboard. I fixed the data contracts and a mock API first.
  Frontend and backend moved while the rest settled.
- **Deciding without full information?** I write down the assumptions and ask
  how costly the call is to undo. A cheap one I make now. An expensive one
  waits for the cheapest test that would change my mind.
- **Requirements keep changing?** Keep the open parts flexible and the settled
  parts strict. In the prototyping kit a step lived as instructions while we
  learned it. When the agent got it wrong twice, it became a script.

## Communication and trust

- **A disagreement?** UI settings. The designer wanted a full-page modal. I
  brought the product teams' concerns, and we agreed on criteria before
  options. The spotlight met them and the designer agreed.
- **Challenging a senior engineer?** I make the criteria explicit and compare
  options against them. In accessibility tooling I brought the principal
  engineer options with a pick. A RACI matrix set who decides what.
- **A decision went against you?** If the risk is understood and accepted, I
  back it and help it succeed. I state my concern once, with the signal that
  would make us revisit.
- **Building trust?** Be predictable. I say risks early, mark guesses as
  guesses and own my calls. In UI settings I found the latency problem myself
  and fixed it before release.

## Collaboration

- **Cross-team example?** UI settings needed Jira, Confluence, Home and Studio
  to integrate. Clear GraphQL contracts and shared packages kept their part
  small. My team wrote most of the integration code, and partner teams
  reviewed it.
- **Influence without authority?** Make the right path the easiest one. In
  accessibility tooling a codemod added the check to existing tests. Teams
  adopted it without writing a test.
- **Getting buy-in?** Agree on the problem and the criteria before the
  solution. For tokens I showed the duplicated CSS on a real page and tied it
  to the company performance goal.
- **The other team disagrees?** I look for their constraint. Teams usually
  share the goal and pay different local costs.

## Engineering standards

- **Raising the bar?** Accessibility tooling. Nothing stopped a PR from adding
  a violation. A shared `toBeAccessible` check went into existing tests, and a
  new violation now fails CI.
- **How did you convince people?** I made the standard cheap. A codemod did
  the adoption, and old violations became tickets for their owners.
- **Quality against velocity?** Enforce only what gives a reliable signal.
  Block what a PR adds and send old debt to its owner with a deadline.
- **When quality beats speed?** When a failure is costly to people or to
  operations. Token edits had no feature flag, so the removals went in stages
  with visual tests between them.

## Beyond your remit

- **An example?** Build pipeline. The build was outside my project, and no
  team owned it. I timeboxed it beside my main work and cut builds from 1 to 2
  hours to about 7 minutes.
- **Why?** The failures hit many teams and blocked every platform change.
  Shared pain with no owner stays.
- **When do you step in?** When the impact crosses teams and no team owns it. I
  also need a real chance to fix it in a timebox.

## Adapting to AI

- **How AI changed your work?** I moved from AI writing code to systems built
  around agents. The prototyping kit gives Claude the design system, a CLI,
  hooks and a fresh subagent per phase.
- **The main lesson?** The model scales execution. The rails around it decide
  whether that's safe.
- **What you keep from AI?** Architecture, product calls, accepting risk and
  the final say on correctness. People merge the Figma sync and decide to
  publish.
- **What you hand to AI?** Bounded work that a check can verify and that is
  cheap to undo. A prototype on an unmerged sandbox branch fits.
- **The biggest AI risk?** A result that looks right and is wrong. Independent
  checks catch it: a separate verify subagent, CI and a person's review.

## Learning quickly

- **An example?** Build pipeline. I learned Rspack well enough to ship a
  production migration. In UI settings I wrote the Java backend and then
  taught backend and GraphQL to a frontend team.
- **How do you learn?** Through a real problem, with small experiments that
  each test one guess.
- **When do you know enough?** When I can explain why it works, how it fails
  and what it costs.

## Results

- **The most measurable?** Build pipeline. Builds went from 1 to 2 hours to
  about 7 minutes. CI costs about US$180,000 a year less.
- **Other impact?** Tokens cut CSS by up to 90 KB and improved LCP by about 3%.
  With the prototyping kit a designer gets a Sandpit link and a draft PR with
  no terminal.
- **How do you measure impact?** End to end: time saved, adoption, reliability
  and the cost to maintain. A falling count is a first sign. Proof that people
  have it easier needs an audit or user reports.

## Operational excellence

- **Reliability?** Move the rules that matter from prompts into code. A hook
  stops the agent ending a run until something will wake it.
- **Rollout?** Small steps with a way back. Rspack and UI settings each ran
  behind a flag. Tokens had no flag, so staged codemods with checks took its
  place.
- **Quality?** It sits in the pipeline. The accessibility check runs in tests
  that already gate CI. UI settings dashboards went up before testing and
  caught the cross-region latency.

## Go bold

- **A bold example?** Prototyping kit. Designers and PMs build working screens
  in the real repo with the real design system. That moves the line between
  design and engineering.
- **The trade-off?** We took on complexity and experimental risk for a new path
  from design to code. Sandbox-only branches kept the blast radius small.

## Go fast

- **Speeding up delivery?** Rspack cut the build by about 95%. The prototyping
  kit turns an ask into a Sandpit link in minutes.
- **Speed without chaos?** Fast feedback needs a signal you trust. A fast loop
  with a bad signal ships mistakes faster.

## Go further

- **An example?** Accessibility. I went past fixing issues to a process that
  catches them on every PR. Tokens got an ESLint rule so fallbacks stay out
  after the cleanup.

## Go together

- **An example?** UI settings and accessibility. Shared contracts and automated
  adoption moved many teams without forcing one team's choice on the others.
- **Different disciplines?** Agree on the outcome, then turn it into
  constraints each discipline can act on. With the designer, the criteria came
  before the options.
- **Conflict?** Find the other side's constraint and optimise the shared
  result.

## Follow-ups to any story

| Question | Answer with | Example |
| --- | --- | --- |
| Why? | The constraint that drove it | Builds hit CI time limits |
| Why not X? | What X did well and what it would cost | Vite meant rewriting the config and every plugin |
| What did you do? | Your calls, your code, who you coordinated | I wrote the first settings backend and planned the milestones |
| How did you know? | The evidence and how you checked it | Build time rose smoothly, so no single commit caused it |
| How did you measure success? | One main metric and one supporting signal | CSS bytes first, LCP second |
| The biggest risk? | The risk and what failure would mean | No flag on token edits, so a bad edit reaches customers |
| How did you reduce it? | Stages, flags and checks | Visual tests between codemod stages |
| What went wrong? | The surprise and what changed | Global settings sat in one US partition |
| What would you do differently? | What to make explicit earlier | Chart the trend before hunting a cause |
| What did you learn? | The belief that changed | Check what the page shows, beyond what the code says |
| What happens at 10x? | Where it breaks and the next investment | A new setting is a new key, with no schema change |
| What if your assumption was wrong? | How you kept the call cheap to undo | A flag brings the Webpack build back in one step |
| The trade-off? | What you gave, what you got and why here | Global replication: more storage for fast reads everywhere |

## Trade-offs per project

| Project | Trade-offs |
| --- | --- |
| Prototyping kit | Scripts over instructions; a subagent per phase over one session; the sandbox over product apps; people on judgement calls over full automation |
| Accessibility tooling | Tests over lint or a crawler; a baseline over failing on old debt; one API over a setup per framework; axe-core over a manual audit |
| UI settings | A backend store over a cookie or localStorage; SSR over a browser load; global replication over the home region; one theme string over a field per setting; a flag per app over one switch |
| Build pipeline | Rspack over Vite; SWC over Babel; selective over full branch builds; a Webpack fallback over a hard switch |
| Tokens | Removing fallbacks over leaving them; staged codemods and team PRs over one run; a lint rule over a one-off cleanup; visual tests over checking by hand |

---

[All projects](./README.md)
