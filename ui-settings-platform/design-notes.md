# UI Settings Platform: design notes

[Back to the overview](./README.md) · [All projects](../README.md)

The reasoning behind the main choices, and the limits they leave.

## Where to store settings

| Option | Good at | Misses |
| --- | --- | --- |
| Domain-wide cookie | No backend work | Lives in one browser. A new device starts from the default |
| Sync the old app backends | Each app keeps its storage | Every app still runs a backend, and the copies drift |
| Store on the account | One record per person | No room for workspace overrides |
| SettingsService (picked) | Built for settings, proven, easy to contribute to | Lacked static partitions, replication and deletion |

The team added the missing parts to SettingsService:

- **Static partitions.** A fixed place for global settings that belong to no
  workspace.
- **Replication.** Global settings copied to every region.
- **Deletion.** DataEngine had no way to delete records.
- **Account events.** A worker deletes a person's settings when the account
  goes.

## Data model

- **The key.** Workspace id as the partition, or none for global. User id as
  the owner. Then the setting name and its value.
- **Where data sits.** Workspace settings sit in the workspace partition. Global
  settings sit in a partition copied to every region.
- **Which value wins.** Workspace beats global, and global beats the default.
- **Only differences.** A value equal to the global theme or the default isn't
  stored.
- **One theme string.** Keys like `colorMode` and `typography` go into one
  serialized value. A new setting is a new key with no schema change.

## Theme on page load

1. **Server.** `preloadTheme` runs a Relay query during SSR and puts the result
   in bootstrap data.
2. **Browser.** `ThemeLoader` reads the bootstrap data and skips the fetch. With
   no data it fetches on the client.
3. **Render.** `ThemeProvider` gets the first theme and holds it in state.
   `useTheme()` reads it, `useSetTheme()` changes it.
4. **Switch.** `ThemeSwitcher` updates the provider and sends the `updateTheme`
   mutation.
5. **Fallback.** The provider keeps the theme in localStorage for when the API
   fails.

## Fixing cross-region latency

Team testing showed AU teammates waiting on long round trips to the US. All
global settings sat in one partition in a US region. I found it before
release, so no customer was affected.

| Option | Good at | Misses |
| --- | --- | --- |
| Global replication (picked) | Fast everywhere. A config change in ERS plus the region in the query | A copy in every region |
| Store in the person's home region | Least storage | Slow when the person travels |
| Store by the region of each request | Fast everywhere | A person sees different settings in each region |

The data was small after storing only differences. The privacy team confirmed
it had no residency limits.

## Onboarding: spotlight or modal

The designer pushed for a full-page modal. I talked to the product teams and
brought their feedback back to the designer:

- **Too intrusive.** Product people didn't want a modal over every page.
- **Pollinator could fail.** These synthetic tests run against live pages. A
  modal on top could break them.
- **Skewed metrics.** A blocking modal changes page metrics and running
  experiments.

The designer's goal was discoverability. We agreed on the criteria first:

- **Discoverable.** People find the new switcher.
- **Low interruption.** Work goes on.
- **No side effects.** Experiments, metrics and Pollinator synthetic tests stay
  intact.

| Option | Good at | Misses |
| --- | --- | --- |
| Full-page modal | Hard to miss | Blocks work, skews metrics, breaks Pollinator tests |
| Spotlight next to the switcher (picked) | Shows the feature where it lives | Easier to miss |

The designer agreed to the spotlight. It goes through PostOffice too.

In Studio the PostOffice setup showed messages on one page
only. The Studio team had no time, so I fixed it and walked them through the
change.

## Rolling out

- **One flag for the feature.** Each app turns it on by itself.
- **Flag off.** The API calls do nothing, the old switcher renders and the
  spotlight stays hidden.
- **Order.** Team dogfooding, then company-wide dogfooding, then a percentage
  rollout to customers.

## The intern project: increased contrast mode

### Picking the project

- **The first idea was too big.** A principal engineer suggested restructuring
  the design system site for LLMs. The scope was unclear and it hung on other
  teams' timelines.
- **Increased contrast mode won.** It had a clear outcome for 3 months. It sat
  next to the settings work, so I could guide it.
- **It was still real work.** Several codebases, several teams and reviews from
  outside owners.
- **It stayed off the critical path.** A delay couldn't put the release at
  risk.

### Guiding the intern

- **Plan.** I set the scope, timeline and milestones with the intern
  coordinator.
- **People.** I connected the intern with the teams and reviewers they needed.
- **Feedback.** The intern polished past what a milestone needed. I explained
  that hitting milestones came first, and the intern adjusted.
- **Assessment.** I ran the midpoint and final assessments and sat on the
  hiring committee.
- **Result.** The intern shipped increased contrast mode with a strong rating.
  They were on track for a permanent role.

## What I would change

- **Plan for regions from day one.** The single global partition cost a
  redesign during testing.
- **Map legacy pages early.** The Jira monolith pages came up only in
  dogfooding, as extra scope.

---

[Back to the overview](./README.md) · [All projects](../README.md)
