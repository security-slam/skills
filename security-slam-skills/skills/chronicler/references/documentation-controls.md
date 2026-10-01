# Documentation Controls: Drafting Guide

Control text comes from the [OSPS Baseline 2026-08-28](https://baseline.openssf.org/versions/2026-08-28#documentation). The drafting notes are practical guidance, not normative text.

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

## Beyond DO: Other Documentation Controls

Other Baseline families also require documentation, for example GV-03.01 (contribution process), VM-02.01 (security contacts), and SA-01.01 (design docs). The Chronicler badge targets the DO family. The `defender` skill covers the rest.
