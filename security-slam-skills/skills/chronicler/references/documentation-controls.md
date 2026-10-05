# Documentation Controls: Drafting Guide

Control text comes from the [OSPS Baseline 2026-08-28](https://baseline.openssf.org/versions/2026-08-28). The drafting notes are practical guidance, not normative text. Where a section names a template, draft from the OpenSSF OSPS Templates as [osps-templates.md](osps-templates.md) describes.

## OSPS-DO-01.01 User Guides (Level 1)

> When the project has made a release, the project documentation MUST include user guides for all basic functionality.

Cover install, configure, and the core tasks a new user performs. A README "Getting Started" section often suffices for small projects. Link a docs site for larger ones.

Check: can a new user go from zero to a working result using only the guide?

## OSPS-DO-02.01 Defect Reporting Guide (Level 1)

> When the project has made a release, the project documentation MUST include a guide for reporting defects.

State where to file bugs, what to include, and that security issues go through SECURITY.md instead. Issue templates help but do not replace a written guide.

## OSPS-DO-06.01 Dependency Management (Level 2)

> When the project has made a release, the project documentation MUST include a description of how the project selects, obtains, and tracks its dependencies.

Answer three questions:

1. **Selects:** What criteria must a new dependency meet (license, maintenance activity, security record)? Who approves it?
2. **Obtains:** Which package manager and registry? Are versions pinned or locked (lockfile, checksums)?
3. **Tracks:** Which tool watches for updates and vulnerabilities (Dependabot, Renovate, OSV-Scanner)?

Example:

```markdown
## Dependencies

We add a dependency only when it is actively maintained and uses an OSI-approved
license. A maintainer must approve every new direct dependency in review.

Go modules are pinned in `go.sum`. Dependabot opens weekly update PRs, and
`govulncheck` runs on every pull request.
```

## OSPS-DO-07.01 Build Instructions (Level 2)

> The project documentation MUST include instructions on how to build the software, including required libraries, frameworks, SDKs, and dependencies.

List toolchain versions, system packages, and the exact build command. Verify the instructions work in a clean environment before marking the control met.

## OSPS-DO-03.01 and 03.02 Release Verification (Level 3)

> When the project has made a release, the project documentation MUST contain instructions to verify the integrity and authenticity of the release assets.
>
> When the project has made a release, the project documentation MUST contain instructions to verify the expected identity of the person or process authoring the software release.

Give copy-paste commands. Name the expected signer identity explicitly. Example for keyless Sigstore signing from GitHub Actions:

```bash
cosign verify-blob \
  --certificate-identity-regexp '^https://github.com/OWNER/REPO/.github/workflows/release.yml@refs/tags/v' \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  --bundle artifact.tar.gz.sigstore.json \
  artifact.tar.gz
```

For SLSA provenance on GitHub, `gh attestation verify artifact.tar.gz --repo OWNER/REPO` also verifies the builder identity.

Match the commands to how the project actually signs. If it does not sign yet, this control is blocked until it does.

## OSPS-DO-04.01 and 05.01 Support and End of Security Updates (Level 3)

> When the project has made a release, the project documentation MUST include a descriptive statement about the scope and duration of support for each release.
>
> When the project has made a release, the project documentation MUST provide a descriptive statement when releases or versions will no longer receive security updates.

A support table works well:

```markdown
## Supported Versions

| Version | Status | Security fixes until |
| --- | --- | --- |
| 2.x | Active | Current |
| 1.x | Security fixes only | 2026-12-31 |
| < 1.0 | End of life | Ended |

Only the latest minor release of each supported major version receives fixes.
```

A rolling statement is also valid: "Only the most recent release receives security fixes."

## OSPS-GV-03.01 and 03.02 Contribution Process and Requirements (Levels 1 and 2)

> The project documentation MUST include an explanation of the contribution process, or clearly state that public contributions are not accepted
>
> The project documentation MUST include a guide for code contributors that includes requirements for acceptable contributions.

GV-03.01 is the process: fork or branch, open a pull request, what review looks like. GV-03.02 adds the bar a contribution has to clear: tests, sign-off or DCO, commit message style, which changes need a design discussion first. A project that accepts no public contributions meets GV-03.01 by saying so.

## OSPS-VM-02.01 Security Contacts (Level 1)

> The project documentation MUST contain security contacts.

Name the contact in SECURITY.md: an email alias, a GitHub private vulnerability reporting link, or named maintainers. Mirror it in `project.vulnerability-reporting.contact`. Section 2 of the CVD Policy template is a one-line placeholder; fill it with real contacts.

## OSPS-GV-01.01 and 01.02 Members and Roles (Level 2)

> The project documentation MUST include a list of project members with access to sensitive resources.
>
> The project documentation MUST include descriptions of the roles and responsibilities for members of the project

A MAINTAINERS.md with GitHub handles and roles covers both. Say which role holds admin, release, and secrets access. Keep the list the project will actually update; a stale list fails the spirit of the control.

Example:

```markdown
## Maintainers

| Name | GitHub | Role |
| --- | --- | --- |
| Ada Example | @ada | Maintainer: merge, release, repository admin |
| Bo Example | @bo | Reviewer: approve pull requests |

Maintainers hold admin access to the repository and the release secrets. Reviewers can approve but not merge.
```

## OSPS-VM-01.01 and 03.01 CVD Policy and Private Reporting (Level 2)

> The project documentation MUST include a policy for coordinated vulnerability disclosure (CVD), with a clear timeframe for response.
>
> The project documentation MUST provide a means for private vulnerability reporting directly to the security contacts within the project.

Draft from the CVD Policy template. The timeframe is the part most existing SECURITY.md files lack: acknowledgement, initial assessment, and disclosure windows, each a number the maintainers chose. "We promise no response timeframe" is honest but does not satisfy VM-01.01; ask the maintainers for a window they can keep. For the private channel, GitHub private vulnerability reporting (`gh api repos/OWNER/REPO/private-vulnerability-reporting --jq .enabled`) or a private email both qualify; the doc has to point at it.

## OSPS-VM-04.01 Published Vulnerability Data (Level 2)

> The project documentation MUST publicly publish data about discovered vulnerabilities.

Say in SECURITY.md where fixed vulnerabilities are announced. Published GitHub Security Advisories satisfy this and reach the GitHub Advisory Database and OSV in machine-readable form. A project with no advisories yet still states where they will appear.

## OSPS-GV-04.01 Review Before Escalated Permissions (Level 3)

> The project documentation MUST have a policy that code collaborators are reviewed prior to granting escalated permissions to sensitive resources.

Draft from the Escalated Permissions Review Policy template. Fill the contribution threshold, who nominates, how identity is checked, and the review cadence from the maintainers.

## OSPS-VM-05.01, 05.02, and 06.01 SCA and SAST Policies (Level 3)

> The project documentation MUST include a policy that defines a threshold for remediation of SCA findings related to vulnerabilities and licenses.
>
> The project documentation MUST include a policy to address SCA violations prior to any release.
>
> The project documentation MUST include a policy that defines a threshold for remediation of SAST findings.

Draft from the SCA Policy and SAST Policy templates. Name the tool the project really runs (Dependabot, OSV-Scanner, CodeQL, Semgrep) and remediation windows per severity the maintainers will hold to. The matching enforcement controls, VM-05.03 and VM-06.02, need a merge-blocking check in CI; route them to the `defender` skill.

## Beyond Chronicler

Other families also require documentation, for example SA-01.01 (design docs), SA-02.01 (external interfaces), QA-06.02 and 06.03 (test docs and policy), and BR-07.02 (secrets policy). The `defender` skill covers them, drafting QA-06.03 and BR-07.02 from the Test Coverage and Secrets and Credentials Management templates.
