# security-slam/skills

[![OSPS Baseline](https://github.com/security-slam/skills/actions/workflows/osps-baseline.yaml/badge.svg)](https://github.com/security-slam/skills/actions/workflows/osps-baseline.yaml)
[![OpenSSF Best Practices](https://www.bestpractices.dev/projects/15132/baseline)](https://www.bestpractices.dev/projects/15132/baseline-1)

A [Claude Code](https://docs.claude.com/en/docs/claude-code) plugin marketplace for earning [Security Slam](https://securityslam.com) project badges.

The marketplace is named `security-slam`. It ships one plugin, `security-slam-skills`, with one skill per project badge and a `slam-status` skill that checks all six. Each skill audits a repository against the badge requirements, reports gaps with evidence, drafts only what the maintainers confirm, and ends with a submission checklist.

## Status

Early preview. The skills work end to end, but they have been tested on one repository: this one. Using its own skills, this repository earned evidence for all six badges, including a passing OSPS Baseline scan and the [bestpractices.dev Baseline Level 1 badge](https://www.bestpractices.dev/projects/15132/baseline-1).

Not yet tested:

- Projects that build and release artifacts. This repository ships Markdown and YAML only.
- OSPS Baseline Level 2 and 3 paths, such as release signing, SBOMs (software bills of materials), and release verification docs.
- The prose self-assessment option of `inspector`, and its hand-off to the gemara-ai plugin.
- Projects hosted outside GitHub, and projects that span several repositories.

The skills pin the specification versions listed under [Versions Targeted](#versions-targeted). When those specifications change, the skills can lag behind until a release updates them.

If a skill gets something wrong on your project, please [report it](CONTRIBUTING.md#reporting-bugs).

## Install

From inside Claude Code:

```
/plugin marketplace add security-slam/skills
/plugin install security-slam-skills@security-slam
```

Or from your shell:

```bash
claude plugins marketplace add security-slam/skills
claude plugins install security-slam-skills@security-slam
```

To try a local checkout, pass the directory path in place of `security-slam/skills`.

## Skills

Not sure where a project stands? Run `/slam-status` first. Otherwise start with Cleaner. Every other badge adds links to the Security Insights file it creates.

| Skill | Badge requirement |
| --- | --- |
| `cleaner` | Complete and validate the project's Security Insights YAML file. |
| `chronicler` | Complete every OSPS Baseline documentation control (OSPS-DO) for the project's maturity level. |
| `inspector` | Complete a Gemara threat assessment or an OSPS self-assessment. |
| `mechanizer` | Automate Baseline evaluation and publish results: 100% on LFX Insights or zero failures from the OSPS Baseline GitHub Action. |
| `defender` | Meet the OSPS Baseline for the project's maturity level and earn the bestpractices.dev Baseline badge. |
| `cra` | Implement voluntary EU Cyber Resilience Act (CRA) readiness practices and publish a readiness checklist with a disclaimer. |
| `slam-status` | Check a repo against all six badges at once and recommend which one to work on next. Read-only. |

Claude picks a skill when your request matches it. You can also call one by name, for example `/cleaner`.

The individual recognitions (Security Advocate, Security Champion, Advisor) go to people, not projects, so they have no skills.

## Versions Targeted

| Dependency | Version |
| --- | --- |
| OSPS Baseline | 2026-08-28 |
| Security Insights schema | 2.2.0 |
| Gemara schema | 1.5.0 |

The `inspector` skill hands Gemara authoring to the [gemara-ai](https://github.com/gemaraproj/gemara-ai) plugin when it is installed, and explains how to migrate catalogs written for Gemara 0.19.x.

## Update

```bash
claude plugins marketplace update security-slam
```

## Releasing

Claude Code caches plugins by version, so users only receive changes that come with a version bump.

- Any PR that changes `security-slam-skills/` or `.claude-plugin/marketplace.json` must bump `version` in `security-slam-skills/.claude-plugin/plugin.json` and `metadata.version` in `.claude-plugin/marketplace.json` to the same value. The `version-check` workflow enforces this.
- The auto-labeler adds the `release` label to those PRs, and merging one publishes a release automatically.
- CI, docs, and other repo changes never trigger a release.

## Uninstall

```bash
claude plugins uninstall security-slam-skills@security-slam
claude plugins marketplace remove security-slam
```

## Security

See [SECURITY.md](SECURITY.md) to report a vulnerability. The project voluntarily documents its practices in [CRA-READINESS.md](CRA-READINESS.md).

## License

[Apache 2.0](LICENSE)
