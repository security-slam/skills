---
name: slam-status
description: Check a repository against all six Security Slam project badges at once (Cleaner, Chronicler, Inspector, Mechanizer, Defender, CRA Readiness) and recommend which badge to work on next. Use when the user asks where a project stands in the Security Slam, which badges it has or is missing, where to start, what to do next, or wants a Slam overview, scorecard, or readiness summary. Read-only. It reports evidence and hands off to the badge skill that does the work.
---

# Security Slam Status

Run a quick, read-only check of the current repository against every Security Slam project badge. Report what evidence exists, then name the one badge skill to run next.

This skill does not earn badges. The `cleaner`, `chronicler`, `inspector`, `mechanizer`, `defender`, and `cra` skills do that work.

## Critical Rules

- **Never write, edit, or delete files.** Never change repository settings. This skill only reads.
- **Never say a badge is earned.** Slam evaluators award badges. Say "evidence found" or "ready to submit".
- **Keep each check quick.** Look for the evidence listed below. Leave the full audit to the badge skill.
- **Mark what you cannot check as `Unknown`** and say why. Never guess a status.
- **End with exactly one recommended next skill.** Stop there. Never add an "after that" or a second suggestion.

## Workflow

### 1. Identify the repository

```bash
git remote get-url origin
gh repo view --json nameWithOwner,visibility,defaultBranchRef
```

Note whether the project has releases (`gh release list --limit 1`). Several checks depend on it.

Then find evidence the repository inherits from its organization:

- **Parent Security Insights file:** if the repository's Security Insights file sets `header.project-si-source`, fetch that URL with `curl -sfL` and read the `project` fields from it. The spec requires the URL to answer an unauthenticated GET. If the fetch fails, say so and mark the checks that depend on it `Unknown`.
- **Org default community files:** when the repository has no SECURITY.md, CONTRIBUTING.md, or CODE_OF_CONDUCT.md, GitHub serves the copy from the `OWNER/.github` repository. Check its root, `.github/`, and `docs/` folders, for example `gh api repos/OWNER/.github/contents/.github --jq '.[].name'`. Count a file found there as present.

Label inherited evidence in the report, for example "SECURITY.md (org default)" or "vulnerability reporting (from project-si-source)".

### 2. Run the quick checks

Check each badge in this order. Record a status and the evidence (a path, URL, or command result).

| Badge | Quick check | Evidence found when |
| --- | --- | --- |
| Cleaner | `find . -maxdepth 2 -iname 'security-insights.y*ml'` | The file exists, declares `schema-version` 2.x, and passes `cue vet` (see the `cleaner` skill). If `cue` is missing, report the file as "present, not validated". |
| Chronicler | Look for user guides and a defect reporting guide (README, CONTRIBUTING.md, docs). A section counts as a defect reporting guide when it tells people how to file a bug, whatever its heading says ("Reporting Bugs", "Issue Report Process", "Filing Issues"). | Both Level 1 docs exist. Higher levels need the maturity level, so report "Level 1 only checked" unless the user gave a level. |
| Inspector | Look for a Gemara threat catalog or a self-assessment document, and for `repository.security.assessments.self.evidence` in the Security Insights file. | An assessment document exists and the Security Insights file links it. |
| Mechanizer | `grep -rl 'osps-baseline-action' .github/workflows`, and check the project on [LFX Insights](https://insights.linuxfoundation.org/). | A scheduled Baseline scan workflow exists, or the project is on LFX Insights. Report the latest run result if you can read it. |
| Defender | Search the README for a `bestpractices.dev` badge. | A Baseline badge from bestpractices.dev is in the README. |
| CRA Readiness | Look for `CRA-READINESS.md` and its disclaimer. Check that SECURITY.md, CONTRIBUTING.md, and LICENSE exist. | The checklist file exists with the disclaimer, and the three supporting files exist. |

Use these status values only:

- `Evidence found`: the quick check passed. The badge skill can confirm and prepare the submission.
- `Partial`: some evidence exists. Say what is missing.
- `Not started`: no evidence.
- `Unknown`: the check could not run. Say why.

### 3. Report

Show one table, then the recommendation. Keep it short.

Correct:

```markdown
## Security Slam status: example/widget

| Badge | Status | Evidence |
| --- | --- | --- |
| Cleaner | Partial | `SECURITY-INSIGHTS.yml` exists; `cue vet` fails on missing `repository.core-team` |
| Chronicler | Evidence found | README "Usage"; CONTRIBUTING.md "Reporting bugs" (Level 1 only checked) |
| Inspector | Not started | No threat catalog or self-assessment found |
| Mechanizer | Not started | No Baseline scan workflow; not on LFX Insights |
| Defender | Not started | No bestpractices.dev badge in README |
| CRA Readiness | Partial | SECURITY.md and LICENSE exist; no CRA-READINESS.md |

**Next: run the `cleaner` skill.** Every other badge links its evidence from the Security Insights file, so fix it first.
```

Wrong - claims a badge, and gives several next steps:

```markdown
You have earned the Chronicler badge. Next you could do Cleaner, or Inspector, or maybe CRA.
```

### 4. Choose the next skill

Recommend the first badge in this order that is not `Evidence found`:

1. `cleaner`: always first. The other badges record their evidence in its file.
2. `chronicler`: documentation comes before automation and assessment.
3. `inspector`: the self-assessment feeds Baseline controls SA-03.01 and SA-03.02.
4. `mechanizer`: automated scans show what still fails.
5. `defender`: the capstone. It needs the four badges above.

Place `cra` by what the user asked for. If they asked about CRA readiness, recommend it first. Otherwise mention it after `chronicler`, because the two share most of their documents.

If every badge shows `Evidence found`, tell the user to run each badge skill's submission checklist.

## Edge Cases

- **Not a git repository, or no remote:** run the file checks only and mark the GitHub checks `Unknown`.
- **Project not hosted on GitHub:** mark Mechanizer `Unknown` and tell the user to ask Slam organizers about an alternate evaluation path.
- **Multi-repository project:** report on the current repository only, and say so. Offer to run again in the other repositories.
- **Org-level files:** a repository with only repository fields in its Security Insights file, and no SECURITY.md or CONTRIBUTING.md of its own, is not missing them. Check `header.project-si-source` and the `OWNER/.github` repository before reporting a gap.
- **No releases yet:** say so in the report. Many Baseline controls apply only after a first release, which makes Chronicler and Defender easier to reach.
