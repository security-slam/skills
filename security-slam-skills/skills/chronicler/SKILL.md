---
name: chronicler
description: Earn the Security Slam Chronicler badge by completing every documentation-shaped OSPS Baseline control for a project's maturity level, across the DO (documentation), GV (governance), and VM (vulnerability management) families. Use when the user mentions the Chronicler badge, Security Slam, OSPS Baseline documentation, user guides, defect reporting guides, vulnerability reporting process, security policy, SECURITY.md, CVD policy, governance docs, MAINTAINERS or GOVERNANCE files, contributing guides, dependency policy, build instructions, release verification docs, or support and end-of-life statements. Audits existing docs, drafts only the missing pieces from the OpenSSF OSPS Templates, and links each one from the Security Insights file.
---

# Chronicler Badge

The Chronicler badge requires the documentation the OSPS (Open Source Project Security) Baseline expects at the project's maturity level: user guides, a defect and vulnerability reporting process, governance, and a description of how releases are built and published. Each document is linked from the Security Insights file.

Source: [securityslam.com/library/chronicler](https://securityslam.com/library/chronicler) and the [OSPS Baseline 2026-08-28](https://baseline.openssf.org/versions/2026-08-28).

## Critical Rules

- **Confirm the maturity level before auditing.** Ask the user, or read it from existing Slam notes. Never pick a level silently. See [Choosing a Maturity Level](#choosing-a-maturity-level).
- **Document what the project actually does.** Ask the maintainer how they build, release, support, govern, and triage. Never write a support window, response timeframe, review rule, or dependency policy the project has not agreed to.
- **Prefer short and accurate over long and stale.** A one-paragraph statement that stays true beats a handbook nobody maintains.
- **Put docs where users look.** README.md, CONTRIBUTING.md, SECURITY.md, GOVERNANCE.md, MAINTAINERS.md, SUPPORT.md, or the project's docs site.
- **Draft policies from the OSPS Templates, with attribution.** See [references/osps-templates.md](references/osps-templates.md). Fill every placeholder from the maintainer; never leave one in.
- **Link every doc from the Security Insights file.** If the file does not exist, run the `cleaner` skill first.

## Controls

Chronicler covers the DO, GV, and VM controls whose requirement falls on the project documentation. Controls in those families that require a mechanism, an enforced check, or an artifact generated per vulnerability (GV-02.01 public discussion channel, VM-05.03 and VM-06.02 blocking checks, VM-04.02 VEX documents) belong to the `defender` skill.

Complete every control at and below the chosen level. Level 2 includes Level 1. Level 3 includes Levels 1 and 2.

The Applies column copies each control's condition from the Baseline text. Read it from the table; never infer it. "After first release" controls are not applicable to a project that has never released. A project that deploys on every merge to main, such as a website, has effectively released.

| Level | Control | Applies | Requirement | Usual home | Security Insights field | Template |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | OSPS-DO-01.01 | After first release | User guides for all basic functionality | README, docs site | `project.documentation.detailed-guide` (required: the Baseline scanner reads only this field), plus `quickstart-guide` if one exists | |
| 1 | OSPS-DO-02.01 | After first release | A guide for reporting defects | CONTRIBUTING.md, issue templates | `repository.documentation.contributing-guide` | |
| 1 | OSPS-GV-03.01 | Always | An explanation of the contribution process, or a clear statement that public contributions are not accepted | CONTRIBUTING.md | `repository.documentation.contributing-guide` | |
| 1 | OSPS-VM-02.01 | Always | Security contacts | SECURITY.md | `project.vulnerability-reporting.contact` | CVD Policy, section 2 |
| 2 | OSPS-DO-06.01 | After first release | How the project selects, obtains, and tracks dependencies | CONTRIBUTING.md or a dependency policy doc | `repository.documentation.dependency-management-policy` | |
| 2 | OSPS-DO-07.01 | Always | How to build the software, including required libraries, frameworks, SDKs, and dependencies | CONTRIBUTING.md, BUILDING.md | `repository.documentation.contributing-guide` | |
| 2 | OSPS-GV-01.01 | Always | A list of project members with access to sensitive resources | MAINTAINERS.md, GOVERNANCE.md | `repository.documentation.governance` | |
| 2 | OSPS-GV-01.02 | Always | Descriptions of the roles and responsibilities for members of the project | GOVERNANCE.md | `repository.documentation.governance` | |
| 2 | OSPS-GV-03.02 | Always | A guide for code contributors that includes requirements for acceptable contributions | CONTRIBUTING.md | `repository.documentation.contributing-guide` | |
| 2 | OSPS-VM-01.01 | Always | A policy for coordinated vulnerability disclosure (CVD), with a clear timeframe for response | SECURITY.md | `project.vulnerability-reporting.policy` | CVD Policy |
| 2 | OSPS-VM-03.01 | Always | A means for private vulnerability reporting directly to the security contacts | SECURITY.md, GitHub private vulnerability reporting | `project.vulnerability-reporting.contact`, `.comment` | CVD Policy, section 3 |
| 2 | OSPS-VM-04.01 | Always | Publicly published data about discovered vulnerabilities | GitHub Security Advisories or an advisories page, linked from SECURITY.md | `repository.documentation.security-policy` | |
| 3 | OSPS-DO-03.01 | After first release | How to verify the integrity and authenticity of release assets | Release verification doc | `project.documentation.signature-verification` | |
| 3 | OSPS-DO-03.02 | After first release | How to verify the identity of the person or process that authored a release | Release verification doc | `project.documentation.signature-verification` | |
| 3 | OSPS-DO-04.01 | After first release | Scope and duration of support for each release | SUPPORT.md | `project.documentation.support-policy` | |
| 3 | OSPS-DO-05.01 | After first release | When releases stop receiving security updates | SUPPORT.md, SECURITY.md | `project.documentation.support-policy` | |
| 3 | OSPS-GV-04.01 | Always | A policy that code collaborators are reviewed prior to granting escalated permissions to sensitive resources | GOVERNANCE.md | `repository.documentation.governance` | Escalated Permissions Review Policy |
| 3 | OSPS-VM-05.01 | Always | A policy that defines a threshold for remediation of SCA findings related to vulnerabilities and licenses | SECURITY.md | `repository.documentation.security-policy` | SCA Policy |
| 3 | OSPS-VM-05.02 | Always | A policy to address SCA violations prior to any release | SECURITY.md | `repository.documentation.security-policy` | SCA Policy |
| 3 | OSPS-VM-06.01 | Always | A policy that defines a threshold for remediation of SAST findings | SECURITY.md | `repository.documentation.security-policy` | SAST Policy |

For drafting guidance and examples for each control, see [references/documentation-controls.md](references/documentation-controls.md).

## Workflow

### 1. Establish context

- Confirm the maturity level with the user.
- Check whether the project has made a release: `gh release list --limit 5` and `git tag --sort=-creatordate | head`.
- Read the Security Insights file if present.
- Check for org-level defaults: when the repository has no SECURITY.md, CONTRIBUTING.md, or GOVERNANCE.md of its own, GitHub serves the copy from the `OWNER/.github` repository. Audit that copy and label it "(org default)".

### 2. Audit

For each control in scope, search the repo and docs site for existing content. Record the result as a table:

| Control | Status | Evidence |
| --- | --- | --- |
| OSPS-DO-01.01 | Met | README.md "Usage" section covers install, configure, run |
| OSPS-DO-02.01 | Partial | Issue template exists; no written guide distinguishes bugs from security reports |
| OSPS-VM-01.01 | Partial | SECURITY.md explains how to report but names no response timeframe |
| OSPS-GV-01.01 | Gap | No MAINTAINERS or GOVERNANCE file |

Use `Met`, `Partial`, `Gap`, or `N/A`. Every `Met` needs a file path or URL. Every `N/A` needs a reason.

### 3. Close the gaps

Show the audit to the user and ask which gaps to draft. For each one:

1. Ask the maintainer for the facts you cannot find in the repo (support windows, response timeframes, signing process, dependency approval rules, who holds admin access).
2. Draft the smallest doc that satisfies the control. Extend an existing file before creating a new one. Where the table names a template, start from it as [references/osps-templates.md](references/osps-templates.md) describes, and keep the attribution comment.
3. Quote the control ID in a comment or commit message so reviewers can trace it.

Correct - extends CONTRIBUTING.md with a defect guide that separates security reports:

```markdown
## Reporting Bugs

Open a GitHub issue using the "Bug report" template. Include the version, your
platform, steps to reproduce, and what you expected to happen.

Do not report security vulnerabilities in public issues. Follow SECURITY.md instead.
```

Correct - a CVD timeframe the maintainers chose, adapted from the template:

```markdown
<!-- Adapted from the Coordinated Vulnerability Disclosure Policy template in the
     OpenSSF OSPS Templates (ORBIT Definitions SIG), CC BY 4.0: https://github.com/OWNER/REPO
     OSPS-VM-01.01 -->
## Coordinated Disclosure

We acknowledge reports within 5 business days and share an initial assessment
within 10 business days of acknowledgement. Details stay private until a fix is
released or 90 days have passed since confirmation, whichever comes first.
```

Wrong - invents a support policy the maintainers never agreed to, or leaves a template placeholder in:

```markdown
## Support

Each minor release receives security fixes for 24 months.

We acknowledge reports within [N] business days.
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

The Slam evaluates Defender at a level set by the project's standing. Use the same level here so the two badges line up:

| Project | Level |
| --- | --- |
| CNCF or OpenSSF Sandbox | 1 |
| CNCF or OpenSSF Incubating | 2 |
| CNCF or OpenSSF Graduated | 3 |
| Non-stewarded, newly under construction | 1 |
| Non-stewarded, established, no known production users | 2 |
| Non-stewarded, known or suspected production users | 3 |
| Other stewarded project | Ask the steward |

When unsure, start at Level 1 and climb.

## Edge Cases

- **Docs live on a separate site repo:** audit that repo too and link its published URLs.
- **Multi-repo project:** document build, dependency, and governance practices per repo, or link one shared doc from each repo.
- **Org-level files:** a SECURITY.md or CONTRIBUTING.md served from `OWNER/.github` counts. Make the change there once rather than copying it into each repo.
- **No signing yet (Level 3):** DO-03 cannot be met with docs alone. Point the user to the `defender` skill for BR-06.01 release signing, then document the verification steps once signing exists.
- **No SCA or SAST check yet (Level 3):** write the policy from the template, then route VM-05.03 and VM-06.02 to the `defender` skill. A policy without the blocking check does not satisfy those two.
- **Machine-readable end-of-life data:** consider [OpenEoX](https://openeox.org) or [endoflife.date](https://endoflife.date/recommendations) alongside the prose statement.
