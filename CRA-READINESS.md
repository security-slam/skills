# CRA Readiness

This project voluntarily documents its security practices using the Security Slam
[CRA Readiness checklist](https://securityslam.com/library/cra-readiness), aligned
with the EU [Cyber Resilience Act](https://openssf.org/public-policy/eu-cyber-resilience-act/).

## Disclaimer

> _This project voluntarily documents its security practices._
> _This information is provided "as is", without warranties or guarantees._
> _The maintainers and contributors:_
>
> - _have no obligations under the EU CRA,_
> - _are not Manufacturers, Importers, or Economic Operators,_
> - _assume no financial, contractual, or legal liability,_
> - _and do not provide CRA compliance assurances._
>
> _Entities incorporating this software into commercial products remain solely responsible for regulatory compliance, risk assessment, and vulnerability management._

For more context, see the ORC WG [maintainer transparency FAQ](https://cra.orcwg.org/faq/maintainers/transparency/).

## Checklist

Last reviewed: 2026-10-01

| Item | Description | Link to artifact |
| --- | --- | --- |
| Cybersecurity and Vulnerability Management Policy | Covers secure development practices, risk handling, security contact, the vulnerability reporting, remediation, and disclosure process, and the support period and end-of-life process. | [SECURITY.md](https://github.com/security-slam/skills/blob/main/SECURITY.md) |
| Contributing Guidance | Contributing guide links to secure development practices. | [CONTRIBUTING.md](https://github.com/security-slam/skills/blob/main/CONTRIBUTING.md#submitting-changes) |
| Release Documentation | Release notes describe new functionality and security fixes. | [GitHub releases](https://github.com/security-slam/skills/releases) |
| Bug Reporting Guide | Process for reporting non-security bugs, separate from security reporting. | [CONTRIBUTING.md](https://github.com/security-slam/skills/blob/main/CONTRIBUTING.md#reporting-bugs) |
| MFA Enforcement | MFA is enabled for all contributors and required for admins. | The `security-slam` GitHub organization requires two-factor authentication for all members. |
| Branch Protection | Branch protection is enabled on the default branch. | A ruleset on `main` requires pull requests, signed commits, and passing `pre-commit` and `version-check` checks, and blocks force pushes and deletion. |
| License File | The repository contains a clear, OSI-approved license. | [LICENSE](https://github.com/security-slam/skills/blob/main/LICENSE) (Apache-2.0) |
| OSPS Baseline | The project meets OSPS Baseline Level 1 or higher. | [OSPS Baseline workflow](https://github.com/security-slam/skills/actions/workflows/osps-baseline.yaml) |
