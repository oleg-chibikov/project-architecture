# Project architecture

- [Prototyping kit](./prototyping-kit/README.md): Claude builds a screen from
  the real design system and shares it as a draft PR with a Sandpit link.
- [Accessibility tooling](./accessibility-tooling/README.md): a codemod adds an
  axe-core check to existing tests. New violations fail CI, old ones become
  tickets.
- [UI Settings Platform](./ui-settings-platform/README.md): a person picks a
  theme once and every Atlassian app shows it, loaded during server rendering.
- [Build pipeline optimisation](./build-pipeline/README.md): the Atlaskit
  website build went from 1 to 2 hours to about 7 minutes with Rspack and
  selective branch builds.
- [Tokens performance](./tokens-performance/README.md): codemods removed
  hard-coded fallbacks from design token calls. Pages ship up to 90 KB less
  CSS.

[Values questions](./values.md): short answers, each backed by a project.
