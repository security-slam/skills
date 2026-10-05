# CRA Readiness Items: Examples and Guidance

Source: the Security Slam [CRA Readiness Guide](https://securityslam.com/library/cra-readiness-guide). Example links are the guide's own picks.

## 1. Cybersecurity and Vulnerability Management Policy

A public file or docs section, usually SECURITY.md or security.txt. It must address all five points:

1. Secure development practices
2. How project risks are handled
3. Contact information for security and vulnerability questions
4. The process for vulnerability reporting, identification, remediation, patching, and coordinated disclosure
5. An end-of-life plan with the intended support period and the end-of-life process

Examples: [KubeVirt](https://github.com/kubevirt/kubevirt?tab=security-ov-file), [oneDNN](https://github.com/uxlfoundation/oneDNN?tab=security-ov-file), [Node.js](https://github.com/nodejs/node?tab=security-ov-file) (advanced), [TensorFlow](https://github.com/tensorflow/tensorflow?tab=security-ov-file) (advanced), [Django](https://docs.djangoproject.com/en/dev/internals/security/) (external page), [Python](https://www.python.org/dev/security/) (external page).

A published policy increases transparency. It does not create legal obligations for maintainers.

Skeleton for gaps:

```markdown
## Secure Development
All changes go through pull request review and CI (tests, SAST, dependency scanning).

## Risk Handling
Maintainers triage security-relevant issues and track them privately until fixed.

## Support Period and End of Life
Security fixes go to the latest minor release. Older releases are end of life.
```

Fill every value from the maintainers. The skeleton covers points 1, 2, and 5 only. Draft points 3 and 4, the security contact and the vulnerability process with its response timeframe, from the CVD Policy template: see the `chronicler` skill's [OSPS Templates reference](../../chronicler/references/osps-templates.md).

### Recommended additions from the CRA Stewards Playbook

The Linux Foundation [CRA Stewards Playbook](https://policy.openssf.org/CRA/stewards-playbook.html) lists more for a security policy. The Security Slam guide does not require these, so recommend them unless the steward is the Linux Foundation or one of its foundations, in which case they are required.

- **Scope:** which repositories, artifacts, and deployments the policy covers, and what it does not cover.
- **Response time expectations:** an honest statement. "We promise no response timeframe" counts for a volunteer project. Never push maintainers to commit to a number.
- **Bug bar:** what counts as a vulnerability and what is a normal bug to report publicly.
- **User notification:** where fixes are announced. Published GitHub Security Advisories reach the GitHub Advisory Database and OSV in machine-readable form.

Skeleton for gaps:

```markdown
## Scope
This policy covers the code in this repository, its release artifacts, and its
GitHub Actions workflows. It does not cover third-party dependencies.

## What Counts as a Vulnerability
Report it privately if it could let an attacker run code, read secrets, or
change a release. Wrong output and crashes without a security impact are bugs.
Report them as public issues.

Published advisories appear on the repository's Security tab and in the GitHub
Advisory Database, which feeds OSV.
```

Fill the scope and bug bar from the maintainers. Do not guess what is out of scope.

## 2. Contributing Guidance

CONTRIBUTING.md or a docs page. It must link explicitly to secure development practices (the policy from item 1 or a dedicated section). Examples: [PurpleBooth template](https://gist.github.com/PurpleBooth/b24679402957c63ec426), [oneDNN](https://github.com/uxlfoundation/oneDNN?tab=contributing-ov-file).

## 3. Release Documentation

CHANGELOG.md, GitHub release notes, or a release notes page. Each release lists new functionality and any security fixes. Examples: [KubeVirt v1.7.0](https://github.com/kubevirt/kubevirt/releases/tag/v1.7.0), [Kyverno CHANGELOG](https://github.com/kyverno/kyverno/blob/main/CHANGELOG.md), [oneDPL release notes](https://github.com/uxlfoundation/oneDPL/blob/main/documentation/release_notes.rst).

## 4. Bug Reporting Guide

Explain how to report non-security bugs, and point security reports elsewhere. It can live in CONTRIBUTING.md. Examples: [Kyverno CONTRIBUTING.md](https://github.com/kyverno/kyverno/blob/main/CONTRIBUTING.md), [Node.js issues guide](https://github.com/nodejs/node/blob/main/doc/contributing/issues.md#submitting-a-bug-report).

## 5. MFA Enforcement

Enable MFA for all contributors where the platform supports it. It is a must for admins. See [GitHub 2FA docs](https://docs.github.com/en/authentication/securing-your-account-with-two-factor-authentication-2fa/configuring-two-factor-authentication). On GitHub, an org owner can require it under organization settings. The artifact is a brief statement that it is enabled.

## 6. Branch Protection

Enable branch protection or rulesets on the default branch. See [GitHub branch protection docs](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/managing-a-branch-protection-rule). The artifact is a brief statement that it is enabled.

## 7. License File

A LICENSE or COPYING file, ideally from the [OSI-approved list](https://opensource.org/licenses). A clear license also clarifies liability and responsibility boundaries.

## 8. OSPS Baseline Level 1

Meet [OSPS Baseline](https://baseline.openssf.org) Level 1. Items 1 through 7 cover much of it already. Use the `defender` skill for the full gap analysis, and link the evidence: the project's grc.store target page, the scan workflow from the `mechanizer` skill, or a completed checklist.
