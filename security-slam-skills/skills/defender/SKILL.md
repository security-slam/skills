---
name: defender
description: Earn the Security Slam Defender badge by reaching a passing OSPS Baseline result for the project's maturity level, published as a live, recurring scan on grc.store, with documented evidence for the controls the scanner cannot check linked from Security Insights. Use when the user mentions the Defender badge, Security Slam, OSPS Baseline compliance, baseline gap analysis, maturity level 1, 2, or 3, failed Baseline controls, needs-review controls, or grc.store results. Reads the latest published log, runs a control-by-control gap analysis with evidence, routes gaps to the other badge skills, and drafts the Security Insights evidence.
---

# Defender Badge

The Defender badge requires a passing OSPS (Open Source Project Security) Baseline status for the project's maturity level, judged from a live, recurring result on [grc.store](https://grc.store). The publish workflow from the `mechanizer` skill (Option 2) is required; the scanner action alone is not enough. Controls the scanner cannot evaluate need documented evidence a reviewer can follow, linked from Security Insights. The submission is the grc.store target page URL.

Defender is the capstone. Cleaner, Chronicler, Inspector, and Mechanizer are stepping stones toward it.

Source: [securityslam.com/library/defender](https://securityslam.com/library/defender) and the [OSPS Baseline 2026-08-28](https://baseline.openssf.org/versions/2026-08-28).

## Critical Rules

- **Confirm the maturity level with the user** using the mapping below. Never pick it silently.
- **The verdict is the hub's.** Never restate a published result as something other than what the target page shows. Fix the control, merge, and let the next run confirm it.
- **Every "Met" needs evidence:** a file path, URL, API response, or settings screenshot the user confirms. For controls the scanner never evaluates, the evidence goes into a Security Insights field, and evaluators follow the link. A bare "yes" does not count.
- **Never change repository or org settings without explicit approval.** Show the exact change first.
- **Answer honestly.** An unmet control stays unmet. Justify `N/A` only when the control's condition truly does not apply (for example, no releases yet).
- **Start from a fresh scanner run.** Never infer a result the OSPS Baseline scanner reports. See [Scan First](#scan-first).

## Scan First

Before reading the hub, run the OSPS Baseline scanner locally at the chosen level as [the scanner reference](../mechanizer/references/local-scan.md) describes. The hub's published result stays the verdict for the badge. The local run is the working picture between publishes: it runs the same plugin version the hub does, it reads the settings the maintainer's token can see and the job token cannot (branch protection, secret scanning), and rerunning it is how a fix is checked before it is merged and published.

## Maturity Level

The Slam evaluates Defender at a level set by the project's standing:

| Project | Level |
| --- | --- |
| CNCF or OpenSSF Sandbox | 1 |
| CNCF or OpenSSF Incubating | 2 |
| CNCF or OpenSSF Graduated | 3 |
| Non-stewarded, newly under construction | 1 |
| Non-stewarded, established, no known production users | 2 |
| Non-stewarded, known or suspected production users | 3 |
| Other stewarded project | Ask the steward |

Higher levels add controls, and the scanner covers fewer of them automatically, so Level 2 and 3 projects should expect most of the work in step 5.

## Workflow

### 1. Establish context

- Confirm the maturity level.
- Read the Security Insights file. If none exists, run the `cleaner` skill first. It becomes the evidence index.
- Note whether the project has releases, multiple repositories, and CI.
- Confirm the publish workflow is wired: `grep -rl pvtr-publish-results .github/workflows`. If nothing matches, hand off to the `mechanizer` skill, Option 2, and come back once the first run has published. A repository on the scanner action alone has Mechanizer evidence, not Defender evidence.
- Read the `target:` input and the `applicability` entry from that workflow. The applicability must match the chosen level (`maturity-1`, `maturity-2`, or `maturity-3`). If it does not, change it there first: the hub judges the level the scan was run at.

### 2. Read the latest published result

```bash
ns=NAMESPACE; target=TARGET; level=1
curl -sf "https://hub.grc.store/v1/targets/$ns/$target" \
  | jq '.versions[0].latest | {result, counts, run_at, log_namespace, log_id, log_version}'
```

`result` is `Passed` or `Failed`. `counts` span every control the scanner knows at every level, so they will not match the filtered list below; the list is the authority for the chosen level. Pull the log with the `log_namespace`, `log_id`, and `log_version` from that output:

```bash
curl -sf "https://hub.grc.store/v1/catalogs/LOG_NAMESPACE/LOG_ID/versions/LOG_VERSION" \
  | jq -r --arg l "maturity-$level" '.evaluations[]["assessment-logs"][]
           | select(.applicability | index($l))
           | "\(.requirement["entry-id"])\t\(.result)\t\(.message)"' | sort
```

The same log is linked from each run on the target page, `https://grc.store/targets/NAMESPACE/TARGET`. Sort the controls into three buckets:

- `Failed`: step 4.
- `Needs Review`: step 5. Read each `message` first: when it names a Security Insights declaration the scanner looked for, adding that field flips the result on the next run, and prose never does.
- `Passed`: record as Met, with the target page as evidence.

Skip `Not Run` entries whose message says the control identifier is retired; they are logged for visibility and are not in the 2026-08-28 reference. Other `Not Run` controls at the level go to step 5.

If the target page returns 404, the first run has not published yet. The `mechanizer` skill's first-run table maps the error to its cause.

### 3. Gap analysis

Load [references/osps-baseline-2026-08-28.md](references/osps-baseline-2026-08-28.md). For every control at and below the chosen level, start from the scanner's result, then check the rest with the commands in [references/control-evidence.md](references/control-evidence.md).

Report by family, with a summary line:

```
Level 2: 31 of 43 controls met, 4 N/A, 8 gaps

| Control | Status | Evidence / Gap | Route to |
| --- | --- | --- | --- |
| OSPS-AC-03.01 | Met | Scanner: Passed. Ruleset "main" blocks direct pushes | |
| OSPS-BR-06.01 | Gap | Releases are unsigned | defender (this skill) |
| OSPS-DO-06.01 | Gap | No dependency policy | chronicler |
| OSPS-GV-01.01 | Met (scanner: Needs Review) | MAINTAINERS.md; link from Security Insights | defender (this skill) |
| OSPS-SA-03.01 | Gap | No security assessment | inspector |
| OSPS-VM-01.01 | Met | SECURITY.md, 90-day disclosure window | |
```

Route gaps to the skill that owns them:

| Gap type | Skill |
| --- | --- |
| Security Insights fields | `cleaner` |
| Documentation and policies in the DO, GV, and VM families: user guides, defect and vulnerability reporting, governance, CVD, SCA and SAST policies | `chronicler` |
| OSPS-SA-03 assessments and threat models | `inspector` |
| Scan wiring, scanner failures, grc.store setup | `mechanizer` |
| Settings, CI hardening, release signing, SBOM, blocking SCA and SAST checks, test and secrets policies | this skill |

For QA-06.03 and BR-07.02, draft from the Test Coverage and Secrets and Credentials Management templates as the `chronicler` skill's [OSPS Templates reference](../chronicler/references/osps-templates.md) describes.

### 4. Drive the failed controls to zero

Work with the user in priority order: Level 1 first, then settings changes (fast, high impact), then docs, then release pipeline work. Fix one, merge it, then publish with `gh workflow run` to confirm it. To check a fix before it lands and publishes, rerun the local scan from [Scan First](#scan-first), or run the OSPS Baseline Action on the pull request; never trigger the publish workflow from a pull request. Run it on the workflow file step 1 found, and respect the hub's ten-minute window per target.

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

### 5. Cover what the scanner cannot see

For each `Needs Review` or `Not Run` control at the chosen level, write down where and how the project meets it, and link that from the Security Insights file. A link to a real document, a settings page, or a release that shows the practice in action counts. The `cleaner` skill's [field reference](../cleaner/references/security-insights-fields.md) maps controls to fields:

```yaml
repository:
  documentation:
    governance: https://github.com/OWNER/REPO/blob/main/GOVERNANCE.md # GV-01.01, GV-01.02, GV-04.01
    review-policy: https://github.com/OWNER/REPO/blob/main/CONTRIBUTING.md#review # QA-07.01
```

For a `Needs Review` whose message names a Security Insights declaration, add that field; the scanner reads it on the next run. When no field fits, for example a repository setting the token could not read and the scanner does not read from Security Insights, state it in SECURITY.md or GOVERNANCE.md and link that file from `repository.documentation.security-policy` or `.governance`.

Check every evidence URL before it goes into the file. Anything other than `200` gets fixed or replaced:

```bash
curl -sL -o /dev/null -w '%{http_code} %{url_effective}\n' URL
```

For a URL with a `#fragment`, also confirm the target page has that heading. Reviewers follow these links, and a dead one reads as an unmet control.

### 6. Show it (optional)

Add a plain link to the target page at the top of the README. A badge image for grc.store results is not available yet.

```markdown
[OSPS Baseline results](https://grc.store/targets/NAMESPACE/TARGET)
```

The `repository.security.tools` entry from the `mechanizer` skill, with `location` on the target page, already records the published result in Security Insights. Do not add a `third-party` assessment entry for it.

### 7. Submission checklist

- [ ] The grc.store target page shows a passing latest log at the chosen level
- [ ] Security Insights links evidence for every control at the level the scanner did not evaluate, and validates
- [ ] Optional: README links the target page
- [ ] The target page URL submitted on the Defender badge page

## Edge Cases

- **No releases yet:** release-conditioned controls (BR-02, BR-04, BR-06, DO-01, and others conditioned on an official release or on the project having made a release) are `N/A`. Say so explicitly. The publish workflow's version falls back to `0.0.0`.
- **Multiple repositories:** QA-04.01 requires listing them. At Level 3, QA-04.02 requires every subproject to meet the same bar, so audit each repo. Each repository publishes its own target.
- **Existing bestpractices.dev Baseline badge:** it is no longer Defender evidence. Keep it in the README if the project has one; it does not count toward the badge.
- **Scanner behind the Baseline version:** the log names the catalog only as `osps-baseline`; the scanner version is in the log's `metadata.author.version`. When a control in the log is missing from the 2026-08-28 reference or vice versa, note the gap in the submission and ask a Slam advisor. Do not fake compliance.
- **Level 2 or 3:** this skill has only been run at Level 1. Expect rougher edges, and most of the work in step 5.
- **Kusari Inspector** (free for CNCF and OpenSSF projects) helps with AC-04, BR-01, BR-07, QA-02, VM-05, and VM-06. Suggest it when those are gaps.
