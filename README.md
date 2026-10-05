# Security Slam Agent Skills

[![OSPS Baseline](https://github.com/security-slam/skills/actions/workflows/osps-baseline.yaml/badge.svg)](https://github.com/security-slam/skills/actions/workflows/osps-baseline.yaml)
[![OpenSSF Best Practices](https://www.bestpractices.dev/projects/15132/baseline)](https://www.bestpractices.dev/projects/15132/baseline-1)

[Agent Skills](https://agentskills.io) for earning [Security Slam](https://securityslam.com) project badges.

The skills follow the open Agent Skills standard, so they work with any compatible agent, including Claude Code, Codex, GitHub Copilot, Cursor, and Gemini CLI. For Claude Code, this repository is also a plugin marketplace named `security-slam` that ships them as one plugin, `security-slam-skills`.

There is one skill per project badge, plus a `slam-status` skill that checks all six. Each skill audits a repository against the badge requirements, reports gaps with evidence, drafts only what the maintainers confirm, and ends with a submission checklist.

## Quick start

Install the skills, then open your agent in the repository you want to check and ask: "Where does this repo stand in the Security Slam?"

Claude Code:

```bash
claude plugin marketplace add security-slam/skills
claude plugin install security-slam-skills@security-slam
claude "/security-slam-skills:slam-status"
```

Any other agent:

```bash
npx skills add security-slam/skills
```

That question runs `slam-status`, which is read-only. It reports where you stand on all six badges and suggests one next step.

## Status

Early preview. The Spring 2026 versions of the skills ran end to end on four repositories, and each one earned evidence for all six badges using the skills. The Fall 2026 badge pages moved Mechanizer and Defender evidence to [grc.store](https://grc.store), and no repository has been through that flow with the skills yet:

| Repository | What it is | Baseline scan | grc.store target |
| --- | --- | --- | --- |
| [security-slam/skills](https://github.com/security-slam/skills) | This repository: Markdown and YAML, released on each plugin change | [Scanner action](https://github.com/security-slam/skills/actions/workflows/osps-baseline.yaml) | Pending first run: [`publish-results.yaml`](.github/workflows/publish-results.yaml) publishes to `eddie-knight/skills`, a personal namespace bound to this repository |
| [security-slam/website](https://github.com/security-slam/website) | A TypeScript site with npm dependencies, deployed and released on every merge, with existing Gemara catalogs | [Scanner action](https://github.com/security-slam/website/actions/workflows/osps-baseline.yaml) | Not yet |
| [privateerproj/privateer](https://github.com/privateerproj/privateer) | A Go CLI that ships release binaries with GoReleaser | Scanner action | Not yet |
| [privateerproj/privateer-sdk](https://github.com/privateerproj/privateer-sdk) | A Go library that inherits its security policy and contributing guide from an org `.github` repository | Scanner action | Not yet |

The Spring bestpractices.dev Baseline Level 1 entries ([skills](https://www.bestpractices.dev/projects/15132/baseline-1), [website](https://www.bestpractices.dev/projects/15142/baseline-1), [privateer](https://www.bestpractices.dev/projects/15145/baseline-1), [privateer-sdk](https://www.bestpractices.dev/projects/12018/baseline-1)) stay up but no longer count toward Defender.

Tested with Claude Code (plugin marketplace) and Codex (installed with `npx skills`). On privateer-sdk, Codex picked `slam-status` from a plain question and produced the same report as Claude Code.

Not yet tested:

- The grc.store publish workflow (`mechanizer` option 2) and the `defender` flow that reads the published result. Both were written against a working run on [revanite-io/pvtr-aws-s3](https://github.com/revanite-io/pvtr-aws-s3), not against a run the skills produced.
- The GV and VM documentation controls `chronicler` now covers, and drafting from the OSPS Templates.
- Projects that publish container images or registry packages.
- OSPS Baseline Level 2 and 3 paths, such as release signing, SBOMs (software bills of materials), and release verification docs. All four repositories stopped at Level 1.
- The prose self-assessment option of `inspector`, and its hand-off to the gemara-ai plugin.
- Projects hosted outside GitHub.
- Agents other than Claude Code and Codex, and every skill except `slam-status` outside Claude Code.

The skills pin the specification versions listed under [Versions Targeted](#versions-targeted). When those specifications change, the skills can lag behind until a release updates them.

If a skill gets something wrong on your project, please [report it](CONTRIBUTING.md#reporting-bugs).

## Install

### Prerequisites

Every skill starts by running the OSPS Baseline scanner locally, so the report starts from real results instead of inference. That needs three tools on your `PATH`:

- [`gh`](https://cli.github.com), signed in. The scanner reads the repository with your token, read-only.
- [`pvtr`](https://github.com/privateerproj/pvtr), the Privateer CLI: `brew install privateerproj/tap/pvtr`, or the install script in its README. The skills install and update the `openssf/github-repo` scanner plugin themselves.
- [`yq`](https://github.com/mikefarah/yq) to read the results.

The `cleaner` skill also needs [`cue`](https://cuelang.org) to validate the Security Insights file.

### Claude Code

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

### Other agents

[`skills`](https://github.com/vercel-labs/skills) installs Agent Skills into most compatible agents:

```bash
npx skills add security-slam/skills
```

It asks which agents to install into. Add `--agent <name>` to pick one, `--skill <name>` to install a single skill, or `--global` to install for your user instead of the current project. The installer sends anonymous usage data by default. Set `DO_NOT_TRACK=1` to turn that off.

You can also copy the folders under [`security-slam-skills/skills/`](security-slam-skills/skills) into your agent's skills directory.

## Skills

Not sure where a project stands? Run `slam-status` first. Otherwise start with Cleaner. Every other badge adds links to the Security Insights file it creates.

| Skill | Badge requirement |
| --- | --- |
| `cleaner` | Complete and validate the project's Security Insights YAML file. |
| `chronicler` | Complete every documentation-shaped OSPS Baseline control (DO, GV, and VM) for the project's maturity level, drafting policies from the OpenSSF OSPS Templates. |
| `inspector` | Complete a Gemara threat assessment or an OSPS self-assessment. |
| `mechanizer` | Wire a live, recurring OSPS Baseline scan into the default branch, as the scanner action or the grc.store publish workflow, and record it in Security Insights. |
| `defender` | Reach a passing OSPS Baseline result for the project's maturity level, published on grc.store, with Security Insights evidence for the controls the scanner cannot check. |
| `cra` | Voluntary EU Cyber Resilience Act (CRA) readiness. Details for this badge will be released on October 12, 2026, and the skill says so until then. |
| `slam-status` | Check a repo against all six badges at once and recommend which one to work on next. Read-only. |

Every skill begins with a local run of the OSPS Baseline scanner over each repository in the project, and infers only what the scanner cannot check. Your agent picks a skill when your request matches its description. You can also ask for one by name, for example "use the cleaner skill". In Claude Code, `/cleaner` works too.

The individual recognitions (Security Advocate, Security Champion, Advisor) go to people, not projects, so they have no skills.

## Versions Targeted

| Dependency | Version |
| --- | --- |
| OSPS Baseline | 2026-08-28 |
| Security Insights schema | 2.2.0 |
| Gemara schema | 1.5.0 |
| grc.store publish workflow (`revanite-io/pvtr-publish-results`) | `v1` |
| OSPS Baseline scanner plugin (`openssf/github-repo` on grc.store) | latest at run time; `0.31.0` tested |

The `inspector` skill hands Gemara authoring to the [gemara-ai](https://github.com/gemaraproj/gemara-ai) plugin when it is installed, and explains how to migrate catalogs written for Gemara 0.19.x.

## Update

Claude Code:

```bash
claude plugins marketplace update security-slam
claude plugins update security-slam-skills@security-slam
```

Other agents:

```bash
npx skills update
```

## Releasing

Claude Code caches plugins by version, so Claude Code users only receive changes that come with a version bump. `npx skills` installs from the repository, so its users get the latest `main`.

- Any PR that changes `security-slam-skills/` or `.claude-plugin/marketplace.json` must bump `version` in `security-slam-skills/.claude-plugin/plugin.json` and `metadata.version` in `.claude-plugin/marketplace.json` to the same value. The `version-check` workflow enforces this.
- The auto-labeler adds the `release` label to those PRs, and merging one publishes a release automatically.
- CI, docs, and other repo changes never trigger a release.

## Uninstall

Claude Code:

```bash
claude plugins uninstall security-slam-skills@security-slam
claude plugins marketplace remove security-slam
```

Other agents:

```bash
npx skills remove cleaner chronicler inspector mechanizer defender cra slam-status
```

## Security

See [SECURITY.md](SECURITY.md) to report a vulnerability. The project voluntarily documents its practices in [CRA-READINESS.md](CRA-READINESS.md).

## License

[Apache 2.0](LICENSE)
