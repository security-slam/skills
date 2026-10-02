# Contributing

<!-- OSPS-DO-02.01: defect reporting guide -->

## Reporting Bugs

Open a [GitHub issue](https://github.com/security-slam/skills/issues/new) and include:

- The agent you used and its version, for example Claude Code from `claude --version` or Codex from `codex --version`.
- How you installed the skills: the Claude Code plugin (with its version from `claude plugins list`) or `npx skills`.
- The skill you invoked, for example `cleaner`.
- The repository the skill ran against, if it is public.
- What you expected the skill to produce, and what it produced instead.

Do not report security vulnerabilities in public issues. Follow [SECURITY.md](SECURITY.md) instead.

<!-- OSPS-GV-03.01: contribution process -->

## Submitting Changes

1. Fork the repository, or create a branch if you have write access.
2. Make your change, then run `make precommit`. It needs [pre-commit](https://pre-commit.com) installed.
3. If you change anything under `security-slam-skills/` or `.claude-plugin/marketplace.json`, bump the version in `security-slam-skills/.claude-plugin/plugin.json` and `metadata.version` in `.claude-plugin/marketplace.json` to the same value. The `version-check` workflow fails the PR otherwise. Merging a plugin change publishes a release. See [Releasing](README.md#releasing).
4. Open a pull request against `main` with a [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) title, such as `fix: ...` or `docs: ...`. The prefix sets the pull request's label and its section in the release notes.
5. Sign your commits. The `main` branch requires signed commits, and the `pre-commit` and `version-check` checks must pass.
6. The maintainer adds the `security` label to pull requests that fix a security issue, so the fix appears under Security in the release notes.

Follow the project's [secure development practices](SECURITY.md#secure-development).
