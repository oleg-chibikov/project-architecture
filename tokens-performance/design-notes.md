# Design notes: Tokens performance

[Back to the project](./README.md)

## Why fallbacks cost CSS

The styling library makes one rule per unique value. Two fallbacks for the
same token give two values, so two rules:

```css
.a { color: var(--ds-text, #172B4D); }
.b { color: var(--ds-text, #333); }
.c { color: var(--ds-text); }
```

Rule `.c` is all a themed page needs. Every extra fallback adds a rule like
`.a` or `.b`, and across many components and products that came to up to
90 KB gzipped.

## How the analysis sorted fallbacks

I compared each fallback with the token's real value:

| Case | Example | What happened |
| --- | --- | --- |
| Product without the Babel plugin | Any call | The plugin went in first |
| Fallback equals the token value | `'#172B4D'` for `color.text` | Broad codemod sweep |
| Fallback differs from the token value | `'#333'` for `color.text` | Team PR and a visual check |

## Options

| Option | For | Against |
| --- | --- | --- |
| Leave the fallbacks | No risk | CSS stays large and keeps growing |
| One codemod removes every fallback | Fast | Colours change with no review, and no flag to undo it |
| Staged codemods, team PRs and a lint rule | Safe edits go fast, risky ones get a review | Slower, and teams do part of the work |

I chose the staged path. A blind bulk edit had no off switch.

## How the team PRs were made

1. The codemod found `token()` calls whose fallback differed from the token
   value.
2. A script looked up the team that owns each file.
3. The script grouped the calls by team and opened one PR per team.
4. The team checked the visual regression results and merged.

## The ESLint rule

The rule fails lint on a `token()` call with a second argument. It runs in
each repository the migration touched. New code can't bring the CSS back.

## What I would change

- **Check theming first.** A visual pass on each product before the plan would
  have found the pages that showed fallbacks only.
- **Turn the lint rule on earlier.** As a warning from day one, it would stop
  new fallbacks landing during the migration.
- **Measure each stage.** Numbers per stage would show which one gave the most.
