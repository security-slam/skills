---
name: defender
description: Earn the Security Slam Defender badge by reaching a passing OSPS Baseline status for the project's maturity level, published to grc.store. Use when the user mentions the Defender badge, Security Slam, OSPS Baseline compliance, baseline gap analysis, maturity level 1, 2, or 3, grc.store, pvtr-publish-results, publishing Baseline results, or the OpenSSF Best Practices Baseline badge on bestpractices.dev. Sets up the grc.store publish workflow, drives failed controls to zero, and documents evidence for the controls the scanner cannot check.
---

# Defender Badge

The Defender badge requires a passing OSPS (Open Source Project Security) Baseline status for the project's maturity level, published where anyone can check it. The evidence is the project's grc.store target page, `https://grc.store/targets/NAMESPACE/TARGET_ID`, with a passing scan behind it, plus Security Insights links for the controls the scanner cannot evaluate.

Defender is the capstone. Cleaner, Chronicler, Inspector, and Mechanizer are stepping stones toward it. Mechanizer accepts the OSPS Baseline scanner action or the grc.store publish workflow. Defender accepts only the publish workflow.

Source: [securityslam.com/library/defender](https://securityslam.com/library/defender), the [grc.store setup guide](https://securityslam.com/library/grc-store-setup), and the [OSPS Baseline 2026-08-28](https://baseline.openssf.org/versions/2026-08-28).

## Critical Rules

- **Confirm the maturity level with the user.** Never pick it silently. Use the Slam's criteria in [Maturity Level](#maturity-level).
- **Every "Met" needs evidence:** a file path, URL, API response, or settings screenshot the user confirms. A bare "yes" never counts. Evaluators follow the links.
- **Never change repository or org settings without explicit approval.** Show the exact change first.
- **The grc.store account, namespace, and trusted-publisher binding are the user's.** They are UI steps. Show where to go; never claim you did them. Ask for the namespace; never guess it.
- **Pin `pvtr-publish-results` by tag, not SHA.** It is the one exception to SHA pinning. The hub rejects a SHA-pinned call. Pin every other action to a full commit SHA.
- **Never weaken a check to reach a pass.** Fix the control, or leave it failing and say why.

## Maturity Level

The Slam evaluates CNCF and OpenSSF projects by their foundation stage:

| Stage | Level |
| --- | --- |
| Sandbox | 1 |
| Incubating | 2 |
| Graduated | 3 |

It evaluates projects without a steward by their adoption:

| Project | Level |
| --- | --- |
| Newly under construction | 1 |
| Established, with no known production users | 2 |
| Known or suspected production users | 3 |

For any other steward, ask the user what level the steward expects.

## Workflow

### 1. Establish context

- Confirm the maturity level.
- Read the Security Insights file. If none exists, run the `cleaner` skill first. It becomes the evidence index.
- Note whether the project has releases, multiple repositories, and CI.
- Check how the project earned Mechanizer. If it used the scanner action, setting up the publish workflow (step 2) comes first.
- Ask for the grc.store namespace, and check for an existing target:

  ```bash
  curl -s "https://hub.grc.store/v1/targets?namespace=NAMESPACE" | jq '.items[] | {target_id, verified_at, evaluation_count}'
  ```

### 2. Publish results to grc.store

Skip this step if the target already has recent published evaluations.

1. Walk the user through the prerequisites: an account in the steward's enterprise, a namespace, and a trusted-publisher binding for the repository. See [references/grc-store-publishing.md](references/grc-store-publishing.md#prerequisites).
2. Copy [assets/grc-store-results.yml](assets/grc-store-results.yml) to `.github/workflows/grc-store-results.yml` and fill the placeholders. Set the catalog to the Baseline version the project targets, and the applicability to the confirmed level.
3. After the merge, have the user run the workflow by hand. It defaults to a dry run. Read the job summary together, then have them run it with the dry run off.
4. Confirm the target page shows "Verified owner" and a result.

The job's color reports publication, not the evaluation. A failing baseline still publishes green. Red means nothing landed. For error codes, see [references/grc-store-publishing.md](references/grc-store-publishing.md#errors).

### 3. Gap analysis

Load [references/osps-baseline-2026-08-28.md](references/osps-baseline-2026-08-28.md). Only controls tagged with the project's level or below count. Split them into two groups:

- **Scanner-evaluated:** take the result from the latest published evaluation.
- **Not evaluated** (needs review, or no automated check): check the repo with the commands in [references/control-evidence.md](references/control-evidence.md).

Report by family, with a summary line:

```
Level 2: 31 of 43 controls met, 4 N/A, 8 gaps (5 failed in the scan, 3 need evidence)

| Control | Source | Status | Evidence / Gap | Route to |
| --- | --- | --- | --- | --- |
| OSPS-AC-03.01 | Scan | Met | Ruleset "main" blocks direct pushes | |
| OSPS-BR-06.01 | Scan | Failed | Releases are unsigned | defender (this skill) |
| OSPS-DO-06.01 | Scan | Failed | No dependency policy | chronicler |
| OSPS-SA-03.01 | Evidence | Gap | No security assessment | inspector |
| OSPS-VM-01.01 | Scan | Met | SECURITY.md, 90-day disclosure window | |
```

Route gaps to the skill that owns them:

| Gap type | Skill |
| --- | --- |
| Security Insights fields | `cleaner` |
| OSPS-DO documentation | `chronicler` |
| OSPS-SA-03 assessments and threat models | `inspector` |
| Scanner failures you cannot explain | `mechanizer` |
| Settings, CI hardening, release signing, SBOM, policies | this skill |

### 4. Drive the failed controls to zero

Work with the user in priority order: settings changes first (fast, high impact), then docs, then release pipeline work. Fix one, merge it, and let the next published run confirm it. The user can trigger the workflow by hand instead of waiting for the schedule.

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

### 5. Cover what the scanner can't see

For each control at the project's level that the scan does not evaluate, write down where and how the project meets it. Link that from Security Insights. Use the field that matches the control when one exists; the `cleaner` skill's [field reference](../cleaner/references/security-insights-fields.md) maps fields to controls. A link to a real document, a settings page, or a release that shows the practice counts.

Check every evidence URL before it goes in. Anything other than `200` gets fixed or replaced:

```bash
curl -sL -o /dev/null -w '%{http_code} %{url_effective}\n' URL
```

For a URL with a `#fragment`, also confirm the target page has that heading. A dead link reads as an unmet control.

### 6. Show it (optional)

There is no README badge image for grc.store results yet. A plain link at the top of the README is enough:

```markdown
[OSPS Baseline results](https://grc.store/targets/NAMESPACE/TARGET_ID)
```

Record the target page in Security Insights as an assessment:

```yaml
repository:
  security:
    assessments:
      third-party:
        - name: OSPS Baseline results on grc.store
          evidence: https://grc.store/targets/NAMESPACE/TARGET_ID
          comment: OSPS Baseline Level 1, evaluated weekly by the openssf/github-repo plugin and published through pvtr-publish-results.
```

Also add the workflow under `repository.security.tools` if Mechanizer has not already. Re-validate with the `cleaner` skill.

### 7. Submission checklist

- [ ] The grc.store target page shows "Verified owner" and a passing result at the project's level
- [ ] Every control the scan does not evaluate has evidence linked from Security Insights
- [ ] Security Insights validates
- [ ] Completion notification submitted on the Defender badge page, with the target page URL

## Optional: bestpractices.dev

bestpractices.dev still offers an OSPS Baseline questionnaire and badge. The Defender badge does not need it. Offer it only if the user wants one; the gap analysis above supplies the answers. See [references/bestpractices-questionnaire.md](references/bestpractices-questionnaire.md).

## Edge Cases

- **No releases yet:** release-conditioned controls (BR-02, BR-04, BR-06, DO-01, and others starting "When the project has made a release") are `N/A`. Say so explicitly. The template publishes as version `0.0.0` until the first release.
- **Multiple repositories:** QA-04.01 requires listing them. Each repository is its own target, with its own workflow and trusted-publisher binding, under one namespace. At Level 3, QA-04.02 requires every subproject to meet the same bar, so audit each repo.
- **Not hosted on GitHub:** the publish workflow runs only on GitHub. Tell the user to ask Slam organizers for an alternate evaluation path.
- **Stale result:** grc.store marks results older than 30 days as stale. Keep the weekly schedule.
- **Scanner lags the Baseline version:** the plugin may not ship the newest catalog. Use the newest catalog it ships, note the gap in the submission, and trust the control text in the reference file.
- **Kusari Inspector** (free for CNCF and OpenSSF projects) helps with AC-04, BR-01, BR-07, QA-02, VM-05, and VM-06. Suggest it when those are gaps.
