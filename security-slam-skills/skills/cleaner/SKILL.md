---
name: cleaner
description: Earn the Security Slam Cleaner badge by creating, completing, and validating a project's Security Insights YAML file. Use when the user mentions the Cleaner badge, Security Slam, SECURITY-INSIGHTS.yml, security-insights.yml, or wants machine-readable security metadata for an open source project. Audits the repository, drafts an honest Security Insights v2 file from real evidence, and validates it against the official CUE schema.
---

# Cleaner Badge

The Cleaner badge requires one thing: complete Security Insights YAML documentation for the project.

Security Slam recommends Cleaner as the first badge. Every other badge adds links to this file, so start it early with basic information and grow it as the project completes more work.

Source: [securityslam.com/library/cleaner](https://securityslam.com/library/cleaner) and the [Security Insights specification](https://github.com/ossf/security-insights).

## Critical Rules

- **Never claim a practice the repository does not implement.** Every URL in the file must resolve to a real artifact. Leave a field out rather than point it at a placeholder or an aspirational doc.
- **Never invent contacts.** Take names and emails from existing files (MAINTAINERS, CODEOWNERS, SECURITY.md, GOVERNANCE.md). Ask the user for anything missing.
- **Reuse an existing file.** If the repo already has a Security Insights file, update it in place. Never create a second one.
- **Always validate before declaring done.**
- **Start from a fresh scanner run.** Never infer a result the OSPS Baseline scanner reports. See [Scan First](#scan-first).

## Scan First

Before the audit, run the OSPS Baseline scanner as [the scanner reference](../mechanizer/references/local-scan.md) describes. Five Level 1 controls read the Security Insights file directly (OSPS-DO-01.01, DO-02.01, GV-03.01, QA-04.01, and VM-02.01), and each failing message names the field the scanner looked for. Those fields are the first rows of the table in step 3, and the scan is the check that the file written in step 4 says what the scanner needs.

## Workflow

### 1. Find an existing file

Search these locations, case-insensitively:

```bash
find . -maxdepth 2 -iname 'security-insights.y*ml' -not -path './node_modules/*'
```

If one exists, read it and note its `header.schema-version`. A `schema-version` of `1.x.x` is the retired v1 format. Tell the user and migrate it to v2 in step 4.

If none exists, create `security-insights.yml` at the repository root. That is the name and location the Slam instructions use. The upstream spec also accepts `.github/security-insights.yml`, and the `find` above matches other casings such as `SECURITY-INSIGHTS.yml`. Keep whatever name an existing file or the project's other tooling already uses.

### 2. Gather evidence

Collect facts from the repository before writing any YAML:

| Security Insights field | Where to look |
| --- | --- |
| `project.name`, `repository.url` | `git remote get-url origin`, README |
| `project.administrators`, `repository.core-team` | MAINTAINERS, CODEOWNERS, GOVERNANCE.md, OWNERS |
| `project.vulnerability-reporting` | SECURITY.md, GitHub private vulnerability reporting (`gh api repos/{owner}/{repo}/private-vulnerability-reporting`) |
| `repository.license` | LICENSE / COPYING, SPDX identifier |
| `repository.status` | Activity and README badges. Use a [repostatus.org](https://repostatus.org) value such as `active`, `wip`, `inactive`, or `abandoned`. |
| `project.documentation.*` | README, docs site, `docs/` |
| `repository.documentation.*` | CONTRIBUTING.md, SECURITY.md, GOVERNANCE.md, dependency and review policies |
| `repository.release` | GitHub releases, CHANGELOG, release workflows, package registries |
| `repository.security.tools` | `.github/workflows/`, dependabot/renovate config, CodeQL, Scorecard, SAST and SCA tools |
| `repository.security.assessments` | Self-assessment docs, threat models, third-party audits |

For a multi-repository project, list every repository under `project.repositories`. Consider hosting the project section once and pointing each repo at it with `header.project-si-source`.

### 3. Report what you found

Before writing, show the user a table of each field with its evidence or `MISSING`. Ask the user to fill required gaps (usually contacts and status). Optional gaps become the to-do list for the Chronicler, Inspector, and Mechanizer badges.

### 4. Write the file

Start from [assets/security-insights-minimum.yml](assets/security-insights-minimum.yml). Add optional fields only when evidence exists. See [references/security-insights-fields.md](references/security-insights-fields.md) for required fields and the Baseline control each field supports.

Always:

- Set `header.schema-version` to the latest released spec version (check `gh api repos/ossf/security-insights/releases/latest --jq .tag_name`, drop the leading `v`).
- Set `header.url` to the raw URL of the file on the default branch.
- Set `last-updated` and `last-reviewed` to today in `YYYY-MM-DD` form, quoted.
- Use absolute HTTPS URLs.
- Set `project.documentation.detailed-guide` whenever a user guide exists, including a README with usage sections. The OSPS Baseline scanner reads only this field for OSPS-DO-01.01, so leaving it out fails that control even when the guide exists. `quickstart-guide` alone does not count.

Correct:

```yaml
repository:
  documentation:
    security-policy: https://github.com/example/widget/blob/main/SECURITY.md
```

Wrong - points at a file that does not exist yet:

```yaml
repository:
  documentation:
    security-policy: https://github.com/example/widget/blob/main/docs/security-policy.md  # TODO write this
```

Wrong - claims an assessment the project never performed:

```yaml
security:
  assessments:
    self:
      evidence: https://example.com/assessment
      date: '2026-01-01'
      comment: Completed self-assessment.
```

When no self-assessment exists, keep the required `assessments` object and say so:

```yaml
security:
  assessments:
    self:
      comment: A self-assessment has not been completed yet.
```

### 5. Validate

Validate against the CUE schema that matches `header.schema-version`. The spec does not publish a CUE module, so download the schema file for that tag:

```bash
v=$(yq '.header.schema-version' security-insights.yml)
curl -sfL "https://raw.githubusercontent.com/ossf/security-insights/v${v}/spec/schema.cue" -o /tmp/si-schema.cue
cue vet -d '#SecurityInsights' /tmp/si-schema.cue security-insights.yml
```

If `cue` is missing, install it with `go install cuelang.org/go/cmd/cue@latest` or `brew install cue-lang/tap/cue`. Silent output means the file is valid.

Then check every URL in the file resolves:

```bash
grep -oE 'https://[^ "]+' security-insights.yml | sort -u | while read -r u; do
  code=$(curl -s -o /dev/null -w '%{http_code}' -L "$u"); echo "$code $u"; done
```

Report any non-200 URL to the user. Do not mark the badge ready while a URL is broken.

Expect a 404 for `header.url`, and for links to files added in the same change, until the change merges to the default branch. Tell the user to re-check those URLs after merging. Treat any other non-200 URL as broken.

### 6. Offer CI validation

Offer to add the [Security Insights Action](https://github.com/revanite-io/security-insights-action) so the file stays valid. Its default path is `.github/security-insights.yml`, so set `file:` when the file lives elsewhere. Pin the action to a full commit SHA with a version comment. Resolve the SHA with `gh api repos/revanite-io/security-insights-action/git/ref/tags/<tag>`. Never guess a SHA.

### 7. Submission checklist

Suggest the [Security Insights Editor](https://security-insights.openssf.org/editor/) for the maintainer's review: it loads the generated file and validates it section by section. Then give the user this checklist:

- [ ] Security Insights file committed on the default branch
- [ ] `cue vet` passes
- [ ] Every URL resolves
- [ ] Contacts confirmed by a maintainer
- [ ] Completion notification submitted on the Cleaner badge page

## Edge Cases

- **No releases yet:** omit `repository.release`. It is optional.
- **Non-GitHub forge:** use the forge's raw file URL for `header.url` and skip GitHub-only API checks.
- **Archived or maintenance-only repo:** set `status` accurately (for example `inactive`) and consider `bug-fixes-only: true`.
- **LFX Insights project:** LFX Insights can generate a starter file from the project's "Security & Best Practices" page. Treat that output as a draft and verify every value.
