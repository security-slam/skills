# Security Policy

## Scope

This policy covers the skills and plugin files in this repository (`security-slam-skills/` and `.claude-plugin/`) and its GitHub Actions workflows. It does not cover the tools the skills call (`gh`, `cue`, `git`), the AI agent that runs them, or the projects the skills are run against.

## Reporting a Vulnerability

Report vulnerabilities privately through [GitHub private vulnerability reporting](https://github.com/security-slam/skills/security/advisories/new).

Do not open a public issue, pull request, or discussion for a vulnerability.

## What Counts as a Vulnerability

Report it privately if a skill or workflow could:

- run commands or change repository settings the user did not approve, including through prompt injection from the content of a repository the skill reads,
- leak tokens, secrets, or private repository data,
- or let an attacker change what the skills or a release contain.

A skill giving wrong or incomplete security advice is a bug, not a vulnerability. Report it as a [public issue](CONTRIBUTING.md#reporting-bugs).

Published advisories appear on the repository's Security tab and in the GitHub Advisory Database, which feeds OSV.

## Security Contact

Jason Meridth ([@jmeridth](https://github.com/jmeridth), <jmeridth@gmail.com>) receives and triages vulnerability reports.

## Vulnerability Process

Jason Meridth alone triages reports, through private GitHub security advisories.

1. **Reporting.** A reporter opens a private advisory through the link above.
2. **Identification.** The maintainer confirms whether the report is a vulnerability and which releases it affects.
3. **Remediation.** The fix is developed in the advisory's private fork or a private branch.
4. **Patching.** The fix ships in a new release. Its release notes list it under Security.
5. **Disclosure.** The maintainer publishes the advisory after the fixed release is available, and credits the reporter unless they ask otherwise.

The project promises no acknowledgment or fix timeframe. It is maintained on a volunteer basis.

## Secure Development

- Every change to `main` lands through a pull request. Force pushes and branch deletion are blocked.
- `main` requires signed commits.
- The `pre-commit` and `version-check` checks must pass before merge. `pre-commit` runs actionlint, markdownlint, and YAML and JSON checks.
- GitHub Actions are pinned to full commit SHAs. Workflows default to no permissions and grant each job only what it needs.
- Dependabot proposes weekly updates for GitHub Actions and raises alerts for known-vulnerable dependencies.
- The [OSPS Baseline workflow](https://github.com/security-slam/skills/actions/workflows/osps-baseline.yaml) evaluates the repository against OSPS Baseline Level 1 weekly and on every push to `main`.
- The `security-slam` organization requires two-factor authentication for all members.

See [CONTRIBUTING.md](CONTRIBUTING.md) for the contribution workflow.

## Risk Handling

The maintainer tracks security-relevant reports in private advisories until a fix is released. Non-security bugs go through public issues, as described in [CONTRIBUTING.md](CONTRIBUTING.md#reporting-bugs).

## Support Period and End of Life

Only the latest release receives security fixes. When a new release ships, the previous release reaches end of life and receives no further fixes. Upgrade to the latest release to stay supported. See [Update](README.md#update).
