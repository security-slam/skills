---
name: defender
description: Earn the Security Slam Defender badge by meeting every OSPS Baseline control for the project's maturity level and completing the OSPS Baseline questionnaire on bestpractices.dev. Use when the user mentions the Defender badge, Security Slam, OSPS Baseline compliance, baseline gap analysis, maturity level 1, 2, or 3, OpenSSF Best Practices Baseline badge, or bestpractices.dev. Runs a control-by-control gap analysis with evidence, routes gaps to the other badge skills, and prepares answers for the questionnaire.
---

# Defender Badge

The Defender badge requires robust completeness with the OSPS (Open Source Project Security) Baseline for the project's maturity level. Evidence is the OSPS Baseline badge from [bestpractices.dev](https://www.bestpractices.dev), displayed at the top of the README.

Defender is the capstone. Cleaner, Chronicler, Inspector, and Mechanizer are stepping stones toward it.

Source: [securityslam.com/library/defender](https://securityslam.com/library/defender) and the [OSPS Baseline 2026-08-28](https://baseline.openssf.org/versions/2026-08-28).

## Critical Rules

- **Confirm the maturity level with the user.** Never pick it silently. Use the level guidance in the `chronicler` skill.
- **Every "Met" needs evidence:** a file path, URL, API response, or settings screenshot the user confirms.
- **Never change repository or org settings without explicit approval.** Show the exact change first.
- **The questionnaire is the user's to submit.** bestpractices.dev requires the user to log in with repo admin access. Prepare the answers; never claim you submitted them.
- **Answer honestly.** An unmet control stays unmet on the questionnaire. Justify `N/A` only when the control's condition truly does not apply (for example, no releases yet).

## Workflow

### 1. Establish context

- Confirm the maturity level.
- Read the Security Insights file. If none exists, run the `cleaner` skill first. It becomes the evidence index.
- Note whether the project has releases, multiple repositories, and CI.
- Check for an existing bestpractices.dev entry: search the README for a `bestpractices.dev` badge.

### 2. Gap analysis

Load [references/osps-baseline-2026-08-28.md](references/osps-baseline-2026-08-28.md). For every control at and below the chosen level, check the repo and record the result. Use the commands in [references/control-evidence.md](references/control-evidence.md).

Report by family, with a summary line:

```
Level 2: 31 of 43 controls met, 4 N/A, 8 gaps

| Control | Status | Evidence / Gap | Route to |
| --- | --- | --- | --- |
| OSPS-AC-03.01 | Met | Ruleset "main" blocks direct pushes | |
| OSPS-BR-06.01 | Gap | Releases are unsigned | defender (this skill) |
| OSPS-DO-06.01 | Gap | No dependency policy | chronicler |
| OSPS-SA-03.01 | Gap | No security assessment | inspector |
| OSPS-VM-01.01 | Met | SECURITY.md, 90-day disclosure window | |
```

Route gaps to the skill that owns them:

| Gap type | Skill |
| --- | --- |
| Security Insights fields | `cleaner` |
| OSPS-DO documentation | `chronicler` |
| OSPS-SA-03 assessments and threat models | `inspector` |
| Automated checks, scanner failures | `mechanizer` |
| Settings, CI hardening, release signing, SBOM, policies | this skill |

### 3. Close gaps

Work with the user in priority order: Level 1 first, then settings changes (fast, high impact), then docs, then release pipeline work. Re-run the checks after each change.

Correct - a precise settings change shown for approval:

```bash
# OSPS-AC-03.01 / 03.02: block direct pushes and deletion of main
gh api -X POST repos/OWNER/REPO/rulesets --input - <<'JSON'
{"name":"main","target":"branch","enforcement":"active",
 "conditions":{"ref_name":{"include":["~DEFAULT_BRANCH"],"exclude":[]}},
 "rules":[{"type":"deletion"},{"type":"non_fast_forward"},
          {"type":"pull_request","parameters":{"required_approving_review_count":1,
           "dismiss_stale_reviews_on_push":true,"require_code_owner_review":false,
           "require_last_push_approval":false,"required_review_thread_resolution":false}}]}
JSON
```

Wrong - applying settings changes without asking, or marking a control met because a change is "planned".

### 4. Prepare the questionnaire

1. Tell the user to sign in at [bestpractices.dev](https://www.bestpractices.dev), add the project if needed, and open the OSPS Baseline section. The site autodetects some answers.
2. Produce a copy-ready answer sheet: one line per control with status and the evidence URL for the justification field.
   Check every evidence URL before it goes on the sheet. Anything other than `200` gets fixed or replaced, never submitted:

   ```bash
   curl -sL -o /dev/null -w '%{http_code} %{url_effective}\n' URL
   ```

   For a URL with a `#fragment`, also confirm the target page has that heading. Wrong: a justification that cites a docs page the project moved or deleted. Reviewers follow these links, and a dead one reads as an unmet control.
3. Once the project exists, also give the user a prefilled link that loads every answer into the form for review: `https://www.bestpractices.dev/en/projects/PROJECT_ID/baseline-1/edit?osps_ac_01_01_status=Met&osps_ac_01_01_justification=...`, one `osps_<family>_<nn>_<nn>_status` and `_justification` pair per control, URL-encoded. Open it in the user's browser with `open` (macOS) or `xdg-open` rather than asking them to copy a long URL from the terminal.
4. Warn that the site's autodetector marks OSPS-DO-01.01 Unmet when the user guide is a README instead of a docs folder, and can override a prefilled answer. Tell the user to set it to Met by hand with the README justification.
5. The user saves progress and returns as needed.

### 5. Display the badge

Once the questionnaire shows the level as met, add the badge to the top of the README. Link the badge to the level page, not the project root: `/projects/PROJECT_ID` redirects to the metal-series page, which shows "in progress" for a project that only holds a Baseline badge.

Correct:

```markdown
[![OpenSSF Best Practices](https://www.bestpractices.dev/projects/PROJECT_ID/baseline)](https://www.bestpractices.dev/projects/PROJECT_ID/baseline-1)
```

Wrong - the link lands on the metal-series page:

```markdown
[![OpenSSF Best Practices](https://www.bestpractices.dev/projects/PROJECT_ID/baseline)](https://www.bestpractices.dev/projects/PROJECT_ID)
```

Add the level page to Security Insights as an assessment:

```yaml
repository:
  security:
    assessments:
      third-party:
        - name: OpenSSF Best Practices OSPS Baseline
          evidence: https://www.bestpractices.dev/projects/PROJECT_ID/baseline-1
          comment: OSPS Baseline Level 1 questionnaire, self-reported and autodetected.
```

### 6. Submission checklist

- [ ] bestpractices.dev shows the Baseline level met
- [ ] Badge at the top of the README
- [ ] Security Insights links the evidence and validates
- [ ] Completion notification submitted on the Defender badge page

## Edge Cases

- **No releases yet:** release-conditioned controls (BR-02, BR-04, BR-06, DO-01, and others starting "When the project has made a release") are `N/A`. Say so explicitly.
- **Multiple repositories:** QA-04.01 requires listing them. At Level 3, QA-04.02 requires every subproject to meet the same bar, so audit each repo.
- **Tools lag the Baseline version:** automated tools may still use the October 2025 text. The questionnaire has complete coverage for all three levels. Trust the control text in the reference file.
- **Existing "metal" badge (passing, silver, gold):** keep it. The Baseline badge sits alongside it.
- **Kusari Inspector** (free for CNCF and OpenSSF projects) helps with AC-04, BR-01, BR-07, QA-02, VM-05, and VM-06. Suggest it when those are gaps.
