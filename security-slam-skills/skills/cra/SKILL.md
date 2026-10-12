---
name: cra
description: Earn the Security Slam CRA Readiness badge by implementing the project steward's EU Cyber Resilience Act (CRA) guidance. Use when the user mentions the CRA badge, the CRA Readiness badge, Cyber Resilience Act, CRA readiness, EU CRA for open source, CRA stewardship, the Linux Foundation steward, or escalating an actively exploited vulnerability. Decides whether the CRA applies, then adds a stewardship statement to SECURITY.md, records the vulnerability reporting channel in Security Insights, and documents the steward escalation policy in CONTRIBUTING.md.
---

# CRA Readiness Badge

The CRA Readiness badge requires implementing the CRA guidelines provided by the project's steward. For Linux Foundation projects that guidance is [LF-Supported Projects and CRA Readiness](https://openssf.org/wp-content/uploads/2026/09/LF-Supported-Projects-and-CRA-Readiness-2026-09-10.pdf), and this skill implements it. Most projects finish after step 1.

Source: [securityslam.com/library/cra-readiness](https://securityslam.com/library/cra-readiness).

## Critical Rules

- **The steward's guidance wins.** This skill implements the Linux Foundation guidance. If the project has a different steward that publishes CRA guidance, follow that instead and tell the user the badge submission is that work. If the project has no steward, say the LF guidance does not apply and send the user to the Slam organizers.
- **Never answer the commercial-use question for the user.** Whether the project's releases are intended for commercial products is the maintainers' call. Ask, record the answer, and never infer it from download counts, dependents, or company names in the git log.
- **Never describe the work as CRA compliance, certification, or legal advice.** Say "CRA readiness". Point legal questions at the [OpenSSF Global Cyber Policy channel](https://openssf.slack.com/archives/C084A6XPX0F).
- **Use the statements verbatim.** The STOP sentence and the stewardship statement below are checked word for word. Edit nothing inside them.
- **Edit existing files in place.** Extend SECURITY.md, CONTRIBUTING.md, and the Security Insights file. Never create a parallel policy file.
- **Never invent contacts or channels.** Take the reporting address and channel from SECURITY.md, GitHub private vulnerability reporting, or the user. Follow the `cleaner` skill's rules for the Security Insights file.

## Scan First

Before the audit, run the OSPS Baseline scanner as [the scanner reference](../mechanizer/references/local-scan.md) describes. Controls OSPS-VM-02.01 and OSPS-VM-03.01 read the Security Insights fields step 4 writes, and their messages say which field is missing.

## Workflow

### 1. Context

Read SECURITY.md, CONTRIBUTING.md, and the Security Insights file (`find . -maxdepth 2 -iname 'security-insights.y*ml'`). Check whether GitHub private vulnerability reporting is on: `gh api repos/OWNER/REPO/private-vulnerability-reporting --jq .enabled`.

Ask the user two questions:

1. Who is the project's steward? The Linux Foundation and its foundations (CNCF, OpenSSF, and the rest) use this skill as written.
2. Does the project release software that is intended for use in commercial products?

### 2. Step 1: decide whether the CRA applies

If the answer to the second question is no, either because the project does not release software or because nothing it releases is meant for commercial products, add this sentence to SECURITY.md and stop. The badge is done.

```markdown
We do not anticipate this project being used in commercial products. See [LF-Supported Projects and CRA Readiness](https://openssf.org/wp-content/uploads/2026/09/LF-Supported-Projects-and-CRA-Readiness-2026-09-10.pdf) for more information.
```

The submission is the link to SECURITY.md. Skip to the checklist.

If the answer is yes, continue.

### 3. Step 2: publish the stewardship statement

Add this statement to SECURITY.md, verbatim, above the section that explains how to report a vulnerability:

```markdown
CRA stewardship: This project is supported under the Linux Foundation CRA stewardship framework, as described at https://www.linuxfoundation.org/security. Security vulnerabilities should be reported through the mechanisms described below, which we will coordinate with our CRA steward. For actively exploited vulnerabilities and severe incidents that may require CRA escalation, please use the project's emergency security reporting mechanisms as appropriate.
```

The text below it must say exactly how maintainers want to receive vulnerability reports: the channel, the contact, and what a reporter can expect. If SECURITY.md has no reporting process yet, route that to the `chronicler` skill (OSPS-VM-01.01 through VM-03.01) and come back. The statement says "described below", so it must sit above that process.

Correct - the statement, then the project's real process:

```markdown
## Reporting a Vulnerability

CRA stewardship: This project is supported under the Linux Foundation CRA stewardship framework, [...] as appropriate.

Report vulnerabilities through [GitHub private vulnerability reporting](https://github.com/example/widget/security/advisories/new). We acknowledge reports within 5 business days.
```

Wrong - paraphrased, and no process follows it:

```markdown
This project is covered by the LF's CRA framework. Email us about security bugs.
```

### 4. Step 3: record the reporting channel in Security Insights

Keep an accessible, private reporting mechanism open. GitHub private vulnerability reporting is recommended where supported. Then make the `project.vulnerability-reporting` block describe it:

```yaml
project:
  vulnerability-reporting:
    reports-accepted: true
    bug-bounty-available: false
    policy: https://github.com/OWNER/REPO/blob/main/SECURITY.md
    contact:
      name: <security contact or mailgroup from SECURITY.md>
      primary: true
      email: <address>
    comment: >-
      Report vulnerabilities through GitHub private vulnerability reporting:
      https://github.com/OWNER/REPO/security/advisories/new
```

Evaluators check that `reports-accepted` is `true`, that `policy` points at the SECURITY.md from step 3, and that `contact` or `comment` describes a working channel. If there is no Security Insights file, run the `cleaner` skill first. After editing, validate the file with `cue vet` as the `cleaner` skill's step 5 describes.

### 5. Step 4: document the steward escalation policy

Add a short section to CONTRIBUTING.md so maintainers know when to involve the steward. Keep the two triggers and the address; the wording is the project's.

```markdown
## Escalating to the CRA Steward

This project escalates two kinds of event to the Linux Foundation steward at steward@linuxfoundation.org:

- **Actively exploited vulnerabilities:** a vulnerability where the project has reliable evidence that a malicious actor has exploited it.
- **Severe incidents:** a security compromise of the project's own IT infrastructure.

Ordinary vulnerabilities with no evidence of exploitation are not escalation events. Remediation never waits on the escalation decision.
```

### 6. Submission checklist

STOP path:

- [ ] The opt-out sentence is in SECURITY.md on the default branch, with the link intact
- [ ] Completion notification submitted on the CRA Readiness badge page with the SECURITY.md link

Full path:

- [ ] Stewardship statement in SECURITY.md, verbatim, above a real reporting process
- [ ] `project.vulnerability-reporting` filled in and `cue vet` passes
- [ ] CONTRIBUTING.md escalation section names both triggers and the steward address
- [ ] Completion notification submitted with links to SECURITY.md, the Security Insights file, and CONTRIBUTING.md

## Edge Cases

- **Org-level SECURITY.md or CONTRIBUTING.md:** when the repository has none of its own, GitHub serves the copy from `OWNER/.github`. Make the change there once, and point `policy` at that file.
- **Multi-repository project:** the statement goes in each repository's SECURITY.md (or the org default). The Security Insights `project` block lives once, in the parent file that `header.project-si-source` points at.
- **The user wants the voluntary practices too:** the [CRA Readiness Guide](https://securityslam.com/library/cra-readiness-guide) checklist goes beyond the badge. Most of it is the `chronicler` and `defender` skills' work; offer those rather than drafting a separate checklist file.
