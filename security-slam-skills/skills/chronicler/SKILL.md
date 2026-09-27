---
name: chronicler
description: Earn the Security Slam Chronicler badge by completing every OSPS Baseline documentation control (OSPS-DO) for a project's maturity level. Use when the user mentions the Chronicler badge, Security Slam, OSPS Baseline documentation, user guides, defect reporting guides, dependency policy, build instructions, release verification docs, or support and end-of-life statements. Audits existing docs, drafts only the missing pieces, and links each one from the Security Insights file.
---

# Chronicler Badge

The Chronicler badge requires completing all OSPS (Open Source Project Security) Baseline documentation controls for the project's maturity level.

Source: [securityslam.com/library/chronicler](https://securityslam.com/library/chronicler) and the [Baseline documentation controls](https://baseline.openssf.org/versions/2026-02-19#documentation).

## Critical Rules

- **Confirm the maturity level before auditing.** Ask the user, or read it from existing Slam notes. Never pick a level silently. See [Choosing a Maturity Level](#choosing-a-maturity-level).
- **Document what the project actually does.** Ask the maintainer how they build, release, and support the software. Never write a support window, verification process, or dependency policy the project has not agreed to.
- **Prefer short and accurate over long and stale.** A one-paragraph statement that stays true beats a handbook nobody maintains.
- **Put docs where users look.** Use README.md, CONTRIBUTING.md, SUPPORT.md, SECURITY.md, or the project's docs site.
- **Link every doc from the Security Insights file.** If the file does not exist, run the `cleaner` skill first.

## Controls

Complete every control at and below the chosen level. Level 2 includes Level 1. Level 3 includes Levels 1 and 2.

Controls that start "When the project has made a release" apply only after a first release. For a project with no releases, mark them not applicable and say why.

| Level | Control | Requirement | Usual home | Security Insights field |
| --- | --- | --- | --- | --- |
| 1 | OSPS-DO-01.01 | User guides for all basic functionality | README, docs site | `project.documentation.quickstart-guide`, `detailed-guide` |
| 1 | OSPS-DO-02.01 | A guide for reporting defects | CONTRIBUTING.md, issue templates | `repository.documentation.contributing-guide` |
| 2 | OSPS-DO-06.01 | How the project selects, obtains, and tracks dependencies | CONTRIBUTING.md or a dependency policy doc | `repository.documentation.dependency-management-policy` |
| 2 | OSPS-DO-07.01 | How to build the software, including required libraries, frameworks, SDKs, and dependencies | CONTRIBUTING.md, BUILDING.md | `repository.documentation.contributing-guide` |
| 3 | OSPS-DO-03.01 | How to verify the integrity and authenticity of release assets | Release verification doc | `project.documentation.signature-verification` |
| 3 | OSPS-DO-03.02 | How to verify the identity of the person or process that authored a release | Release verification doc | `project.documentation.signature-verification` |
| 3 | OSPS-DO-04.01 | Scope and duration of support for each release | SUPPORT.md | `project.documentation.support-policy` |
| 3 | OSPS-DO-05.01 | When releases stop receiving security updates | SUPPORT.md, SECURITY.md | `project.documentation.support-policy` |

For drafting guidance and examples for each control, see [references/documentation-controls.md](references/documentation-controls.md).

## Workflow

### 1. Establish context

- Confirm the maturity level with the user.
- Check whether the project has made a release: `gh release list --limit 5` and `git tag --sort=-creatordate | head`.
- Read the Security Insights file if present.

### 2. Audit

For each control in scope, search the repo and docs site for existing content. Record the result as a table:

| Control | Status | Evidence |
| --- | --- | --- |
| OSPS-DO-01.01 | Met | README.md "Usage" section covers install, configure, run |
| OSPS-DO-02.01 | Partial | Issue template exists; no written guide distinguishes bugs from security reports |
| OSPS-DO-06.01 | Gap | No dependency policy found |

Use `Met`, `Partial`, `Gap`, or `N/A`. Every `Met` needs a file path or URL. Every `N/A` needs a reason.

### 3. Close the gaps

Show the audit to the user and ask which gaps to draft. For each one:

1. Ask the maintainer for the facts you cannot find in the repo (support windows, signing process, dependency approval rules).
2. Draft the smallest doc that satisfies the control. Extend an existing file before creating a new one.
3. Quote the control ID in a comment or commit message so reviewers can trace it.

Correct - extends CONTRIBUTING.md with a defect guide that separates security reports:

```markdown
## Reporting Bugs

Open a GitHub issue using the "Bug report" template. Include the version, your
platform, steps to reproduce, and what you expected to happen.

Do not report security vulnerabilities in public issues. Follow SECURITY.md instead.
```

Wrong - invents a support policy the maintainers never agreed to:

```markdown
## Support

Each minor release receives security fixes for 24 months.
```

### 4. Update Security Insights

Add each new doc URL to the matching field from the table above. Update `header.last-updated`. Re-run validation from the `cleaner` skill.

### 5. Submission checklist

- [ ] Every control at the chosen level is `Met` or justified `N/A`
- [ ] Docs merged on the default branch
- [ ] Security Insights file links each doc and validates
- [ ] Completion notification submitted on the Chronicler badge page

## Choosing a Maturity Level

- **Level 1:** any project that has made a release, code or non-code, any number of maintainers.
- **Level 2:** code projects with a small, consistent user base and at least two maintainers.
- **Level 3:** code projects with a large user base, enterprise adoption, critical infrastructure, or sensitive data.

Ask: Who uses it? How mature is it? What is the downstream blast radius of a vulnerability? Does it handle credentials, payments, or sensitive data? When unsure, start at Level 1 and climb.

## Edge Cases

- **Docs live on a separate site repo:** audit that repo too and link its published URLs.
- **Multi-repo project:** document build and dependency practices per repo, or link one shared doc from each repo.
- **No signing yet (Level 3):** DO-03 cannot be met with docs alone. Point the user to the `defender` skill for BR-06.01 release signing, then document the verification steps once signing exists.
- **Machine-readable end-of-life data:** consider [OpenEoX](https://openeox.org) or [endoflife.date](https://endoflife.date/recommendations) alongside the prose statement.
