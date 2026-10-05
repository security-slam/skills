# OSPS Templates

The OpenSSF ORBIT Definitions SIG maintains the OSPS Templates: policy templates that each cite the Baseline controls they satisfy. They are licensed CC BY 4.0, so copying with adaptation is fine as long as the drafted file keeps attribution.

The repository is expected to move. This file is the only place in these skills that names its location, so a move is a change to the two `repo=` lines here:

```bash
repo=eddie-knight/osps-templates # OSPS Templates (OpenSSF ORBIT Definitions SIG)
```

Home page: `https://github.com/$repo`. Templates live under `templates/`.

## Template to control map

| Template | File under `templates/` | Controls | Level | Drafted by |
| --- | --- | --- | --- | --- |
| Coordinated Vulnerability Disclosure (CVD) Policy | `Coordinated-Vulnerability-Disclosure-Policy.md` | VM-01.01 (policy and response timeframe), VM-02.01 (security contacts), VM-03.01 (private reporting channel) | 1 and 2 | `chronicler`; `cra` reuses it for checklist item 1 |
| Escalated Permissions Review Policy | `Escalated-Permissions-Review-Policy.md` | GV-04.01 | 3 | `chronicler` |
| Software Composition Analysis (SCA) Policy | `Software-Composition-Analysis-Policy.md` | VM-05.01, VM-05.02 (the policy); VM-05.03 is the enforced check | 3 | `chronicler` for the policy; `defender` for the blocking check |
| Static Application Security Testing (SAST) Policy | `Static-Application-Security-Testing-Policy.md` | VM-06.01 (the policy); VM-06.02 is the enforced check | 3 | `chronicler` for the policy; `defender` for the blocking check |
| Test Coverage Policy | `Test-Coverage-Policy.md` | QA-06.03 (with QA-06.02 test docs) | 3 | `defender` |
| Secrets and Credentials Management Policy | `Secrets-and-Credentials-Management-Policy.md` | BR-07.02 | 3 | `defender` |

The template headers cite the Baseline release they were written against. The CVD template mentions a "VM-01.03" timeframe note; in the 2026-08-28 release the response timeframe is part of VM-01.01. Trust the control text in the `defender` skill's Baseline reference.

## Drafting from a template

1. Fetch the template at drafting time. Never reproduce one from memory. Set `repo` from the top of this file in the same shell call:

   ```bash
   repo=eddie-knight/osps-templates
   gh api "repos/$repo/contents/templates/Coordinated-Vulnerability-Disclosure-Policy.md" -H "Accept: application/vnd.github.raw"
   ```

2. Drop the header notes: the blockquoted "This draft policy is inspired by" and "Before publishing" blocks, or the one-column header table in the SCA template. They are instructions to the drafter, not policy text.
3. Replace every bracketed placeholder (`[N] business days`, `[security email address]`, `[e.g., annually]`) with a value the maintainer gave you. Never leave a placeholder in, and never pick a number yourself. Delete a section the maintainer says does not apply rather than inventing content for it.
4. Put it where the template suggests, or where the project's existing docs already live: SECURITY.md for the CVD, SCA, SAST, and secrets policies, GOVERNANCE.md or MAINTAINERS.md for escalated permissions, CONTRIBUTING.md for test coverage. Extend an existing file before creating a new one.
5. Keep attribution in the drafted file. One HTML comment at the top of the section is enough:

   ```markdown
   <!-- Adapted from the Coordinated Vulnerability Disclosure Policy template in the
        OpenSSF OSPS Templates (ORBIT Definitions SIG), CC BY 4.0: https://github.com/OWNER/REPO -->
   ```

   Fill `OWNER/REPO` from `$repo` above. Mention the control IDs in the same comment or in the commit message so reviewers can trace them.
