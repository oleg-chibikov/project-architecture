# Project architecture

- [Prototyping kit](./prototyping-kit/README.md): Claude builds a screen from
  the real design system and shares it as a draft PR with a Sandpit link.
- [Accessibility tooling](./accessibility-tooling/README.md): a codemod adds an
  axe-core check to existing tests. New violations fail CI, old ones become
  tickets.
