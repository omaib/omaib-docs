# Contributing to OMAIB Docs

Thank you for your interest in contributing to the OMAIB Platform Documentation.

## How to Contribute

### Reporting Issues

- Use [GitHub Issues](https://github.com/omaib/omaib-docs/issues) to report errors, gaps, or suggest improvements.
- Label issues with `type: documentation`, `type: bug`, or `type: enhancement`.

### Submitting Changes

1. Fork the repository and create a branch from `develop`.
2. Make your changes in the `docs/` directory.
3. Ensure your changes pass all CI checks:
   - `markdownlint` — Markdown style
   - `prettier` — Formatting
   - `yamllint` — YAML validity
   - `stylelint` — CSS lint
   - `mkdocs build --strict` — Build with no warnings
4. Open a pull request against `develop`.

### Local Development

```bash
# Install dependencies
pip install -r requirements.txt

# Serve locally
mkdocs serve

# Build and verify
mkdocs build --strict
```

### Writing Guidelines

- Use [ATX headings](https://spec.commonmark.org/0.31.2/#atx-headings) (`#`, `##`, etc.).
- One sentence per line (makes diffs cleaner).
- Use admonitions (`!!! note`, `!!! warning`) for callouts.
- Add code blocks with language identifiers (` ```yaml `, ` ```bash `).
- Keep filenames lowercase with hyphens: `my-new-page.md`.

### Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

- `docs: update onboarding prerequisites`
- `feat: add new troubleshooting section`
- `fix: correct broken link in CI setup guide`
- `style: format CSS for footer`

## Release Process

Releases follow [SemVer](https://semver.org/). To create a release:

1. Open a PR with the title `Release vX.Y.Z (omaib-docs)`.
2. `changelog-ci` will auto-generate the changelog entry.
3. On merge, `release.yml` creates the git tag and GitHub Release.

## Code of Conduct

This project follows the [OMAIB Code of Conduct](https://omaib.github.io/).
All contributors are expected to uphold a respectful and inclusive environment.

## Licence

By contributing, you agree that your contributions will be licensed under the [MIT Licence](LICENSE).
