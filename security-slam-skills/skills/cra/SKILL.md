---
name: cra
description: Earn the Security Slam CRA Readiness badge by implementing voluntary EU Cyber Resilience Act (CRA) readiness practices and documenting them with a checklist and disclaimer. Use when the user mentions the CRA badge, the CRA Readiness badge, Security Slam, Cyber Resilience Act, CRA readiness, EU CRA for open source, vulnerability management policy, end-of-life or support period policy, or the project steward's CRA guidelines. Audits the eight readiness items, fills gaps, and publishes a CRA-READINESS.md with the required no-liability disclaimer.
---

# CRA Badge

The CRA badge requires implementing the CRA readiness guidelines provided by the project steward. The Slam has not yet confirmed what those guidelines are for each steward. Until it does, the eight-item checklist from the Slam's CRA Readiness Guide is the working definition.

"CRA readiness" is a voluntary transparency signal. It is not regulatory compliance. The CRA places no obligations on open source developers or volunteer maintainers merely for publishing or maintaining code. That covers almost every open source project.

Source: the Security Slam [CRA Readiness badge page](https://securityslam.com/library/cra-readiness), the [CRA Readiness Guide](https://securityslam.com/library/cra-readiness-guide) by Roman Zhukov (Red Hat) and Madalin Neag (Linux Foundation), and the OpenSSF [CRA Brief Guide for OSS Developers](https://best.openssf.org/CRA-Brief-Guide-for-OSS-Developers). Recommended additions to item 1 come from the Linux Foundation [CRA Stewards Playbook](https://policy.openssf.org/CRA/stewards-playbook.html).

## Critical Rules

- **Never describe the work as CRA compliance, certification, or legal assurance.** Use "CRA readiness" and "voluntary". You are not giving legal advice. Say so when the user asks legal questions and point them to the OpenSSF Global Cyber Policy channel.
- **Always include the disclaimer** from [assets/CRA-READINESS.md](assets/CRA-READINESS.md) verbatim in the published document.
- **Ask who the project's steward is, and whether it publishes CRA guidance.** A steward is a legal entity, not a manufacturer, that supports this specific project on a sustained basis, for example by hosting or governing it, for software intended for commercial use (CRA Article 3(14)). Individual maintainers and volunteer groups are never stewards. Many projects have none; record "none" and move on. If the steward publishes guidance (for example CNCF, the Linux Foundation, the Eclipse Foundation, or Apache), it wins where it differs from this skill.
- **Never let CRA-READINESS.md read as an attestation.** No signed declarations, no "certified", and no statement that the project makes a downstream product compliant. The CRA Stewards Playbook rules these out even for stewards.
- **Flag commercial context.** If the maintainers monetize the project or place it on the EU market as a product, or the steward acts as an open source steward under the CRA, tell the user real obligations may apply and that they need qualified legal advice. Continue with the readiness work only if they want to.
- **Never change settings (MFA, branch protection) without explicit approval.**

## Readiness Checklist

| # | Item | Done when |
| --- | --- | --- |
| 1 | Cybersecurity and vulnerability management policy | A public policy covers all five points: secure development practices, how project risks are handled, security contact, the vulnerability process (reporting, identification, remediation, patching, coordinated disclosure), and an end-of-life plan with the intended support period |
| 2 | Contributing guidance | Contributing docs link explicitly to secure development practices |
| 3 | Release documentation | Each release or tag has notes describing changes, including security fixes |
| 4 | Bug reporting guide | A documented process for non-security bugs, clearly separate from security reporting |
| 5 | MFA enforcement | MFA enabled for all contributors where the platform supports it; mandatory for admins |
| 6 | Branch protection | Enabled in org or repo settings |
| 7 | License file | A clear LICENSE or COPYING file, ideally an OSI-approved license |
| 8 | OSPS Baseline | Level 1 at minimum |

Items 1 through 4 and 7 overlap heavily with OSPS Baseline Level 1 and the `chronicler` skill. Item 8 is the `defender` skill at Level 1. Reuse that work rather than duplicating it. For item 1, draft points 3 and 4 (the security contact, and the vulnerability process with its response timeframe) from the CVD Policy template as the `chronicler` skill's [OSPS Templates reference](../chronicler/references/osps-templates.md) describes, and keep its attribution; the skeleton in this skill's reference covers the other points.

For examples and drafting guidance per item, see [references/cra-readiness-items.md](references/cra-readiness-items.md).

## Workflow

### 1. Context

- Ask about steward guidance and commercial context (see Critical Rules).
- Read SECURITY.md, CONTRIBUTING.md, LICENSE, the changelog or releases, and the Security Insights file.

### 2. Audit

Report each checklist item as `Met`, `Partial`, or `Gap` with evidence. For item 1, audit each of the five points separately. Policies most often miss the end-of-life plan.

Then check item 1 against the Stewards Playbook's recommended additions: scope, response time expectations, a bug bar (what counts as a vulnerability), and how users learn about fixes. See [references/cra-readiness-items.md](references/cra-readiness-items.md#recommended-additions-from-the-cra-stewards-playbook). Report each one as `Present` or `Recommended`, and never as a gap, because the Security Slam guide does not require them. They become required only when the steward is the Linux Foundation or one of its foundations.

If the project has a steward, also check that SECURITY.md says how maintainers escalate an actively exploited vulnerability to the steward. The steward's own CRA duties, such as reporting through ENISA's Single Reporting Platform and designating a CSIRT, apply from December 11, 2027 and belong to the steward. Never ask maintainers to take them on.

Check settings where the token allows:

```bash
gh api orgs/OWNER --jq .two_factor_requirement_enabled   # item 5, org admin only
gh api repos/OWNER/REPO/rules/branches/main              # item 6
gh api repos/OWNER/REPO/license --jq .license.spdx_id    # item 7
```

### 3. Close gaps

Draft missing content with the maintainer. Extend SECURITY.md for item 1 rather than creating a parallel policy. For its points 3 and 4, draft from the CVD Policy template as the `chronicler` skill's [OSPS Templates reference](../chronicler/references/osps-templates.md) describes, and keep the attribution comment. Ask the maintainer for the support period and the response timeframe; never invent either.

Correct - an end-of-life statement the maintainers confirmed:

```markdown
## Support Period

We provide security fixes for the latest minor release only. When a new minor
release ships, the previous one reaches end of life and receives no further fixes.
```

Wrong - implies legal responsibility:

```markdown
This project is CRA compliant and guarantees vulnerability fixes within 30 days.
```

### 4. Publish CRA-READINESS.md

1. Copy [assets/CRA-READINESS.md](assets/CRA-READINESS.md) to the repository root.
2. Fill the "Link to artifact" column with real URLs. For MFA and branch protection, write a brief statement that it is enabled.
3. Keep the disclaimer intact.
4. Link the file from the README.

### 5. Link each artifact from Security Insights

The badge page asks for every artifact in the checklist to be linked from `security-insights.yml`. Security Insights has no CRA-specific field, so record each one in the standard v2 field it belongs to: the policy in `repository.documentation.security-policy` and `project.vulnerability-reporting.policy`, the contributing guide and bug reporting guide in `repository.documentation.contributing-guide`, the support period in `project.documentation.support-policy`, release notes in `repository.release.changelog`, the license in `repository.license`, and the Baseline evidence in `repository.security.tools[]`. Validate with the `cleaner` skill.

Do not copy the Kyverno file the CRA guide links as its example. It declares schema 2.1.0 but uses v1 field names and fails v2 validation.

### 6. Submission checklist

- [ ] All eight items `Met`, with steward guidance applied if it exists
- [ ] CRA-READINESS.md published with the disclaimer and linked from the README
- [ ] Security Insights links every artifact and validates
- [ ] Completion notification submitted per the steward's instructions

## Next Steps to Suggest

These go beyond the badge. They are engineering improvements, not CRA requirements: generate SBOMs, adopt SLSA starting at Level 1, reach OSPS Level 2 or 3, automate with Gemara, Minder, and Scorecard.

Questions go to the [OpenSSF Global Cyber Policy Slack channel](https://openssf.slack.com/archives/C084A6XPX0F).
