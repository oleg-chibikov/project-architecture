# Design notes: Build pipeline optimisation

[Back to the overview](./README.md) · [All projects](../README.md)

## What the build time chart showed

The shape of the curve tells the kind of cause:

| Shape | Usual cause | What to do |
| --- | --- | --- |
| A step up | One commit added a heavy step or dependency | Find the commit |
| Spikes | Flaky CI hosts or network | Fix the CI hosts |
| A smooth rise that bends upward | The build grows with the repo | Make the build cheaper |

The Atlaskit chart was a smooth rise. Each new package added more time than
the one before it.

## Why Webpack slows faster than the repo grows

These are known Webpack traits. I didn't profile the build for each one.

- **One thread.** Webpack runs in Node on one main thread. More modules means
  more work in a single line.
- **Babel in JavaScript.** `babel-loader` transpiles each file in JavaScript
  on that same thread.
- **Garbage collection.** A bigger module graph means a bigger heap. Node
  spends more time collecting garbage as the heap nears its limit.
- **Chunk splitting.** Webpack compares modules across chunks to split them.
  Many examples mean many chunks, so this work grows faster than the module
  count.

## Why Rspack is faster

- **Rust.** Compiled native code with no garbage collector.
- **Every core.** Parsing, transforming and code generation run in parallel.
- **SWC.** `builtin:swc-loader` replaces `babel-loader`. SWC is about 20 times
  faster than Babel on one thread.
- **Native minifiers.** Rust minifiers for JavaScript and CSS replace Terser.
- **Webpack config shape.** `entry`, `module.rules`, `plugins` and `resolve`
  stay the same. Most loaders and plugins work as they are.

## Bundler options

| Option | Speed | Migration cost | Result |
| --- | --- | --- | --- |
| Stay on Webpack, trim steps | No real gain | None | Turning off costly steps didn't help |
| Vite | Fast | High: new config and Rollup plugins in place of Webpack ones | A rewrite didn't fit the timebox |
| Rspack | Fast | Low: reads Webpack config | Picked. First build in a couple of days, already used in the company |

## Why both Rspack and selective builds

- **Rspack cuts the cost of each module.** The full build still grows with the
  repo.
- **Selective builds cut the number of modules.** A branch pays for its change
  and the packages that depend on it.
- **Together.** Branch builds dropped to about 7 minutes.

## How a branch picks its packages

1. List the files the branch changes against main.
2. Map each file to its package.
3. Walk the dependency graph to the packages that depend on those.
4. Build the site with docs, changelogs and examples for that set only.

## What I would change

- **A chart from day one.** Build time per run belongs on a dashboard, with an
  alert when the trend climbs.
- **Profile before testing steps.** A build profile shows which phase grew,
  faster than turning steps off one by one.

---

[Back to the overview](./README.md) · [All projects](../README.md)
