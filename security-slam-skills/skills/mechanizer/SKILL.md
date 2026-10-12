---
name: mechanizer
description: Earn the Security Slam Mechanizer badge by wiring a live, recurring OSPS Baseline scan into the default branch, as either the OSPS Baseline GitHub Action or the grc.store publish workflow (pvtr-publish-results), and recording the tooling in Security Insights. Use when the user mentions the Mechanizer badge, Security Slam, automated baseline evaluation, OSPS Baseline scanner, pvtr-github-repo-scanner, Privateer, osps-baseline-action, pvtr-publish-results, grc.store, grc.store namespaces or trusted publishers, or publishing Baseline results. Sets up the scan, walks through the grc.store prerequisites without performing them, and triages failures as a head start on Defender.
---

# Mechanizer Badge

The Mechanizer badge requires a live, recurring OSPS (Open Source Project Security) Baseline scan wired into the default branch and recorded in Security Insights. A passing score is not required. Two options qualify, and both run the same scanner, the OpenSSF-maintained [pvtr-github-repo-scanner](https://github.com/ossf/pvtr-github-repo-scanner) plugin for [Privateer](https://privateerproj.com):

- **Option 1: OSPS Baseline GitHub Action.** Results stay in the repository, as a workflow artifact, in the job log, or in the Security tab. The fastest start, with no rate limits, and it can run on same-repository pull requests.
- **Option 2: grc.store publish workflow.** The [pvtr-publish-results](https://github.com/revanite-io/pvtr-publish-results) reusable workflow runs the scan and publishes each signed result to a public target page on [grc.store](https://grc.store). It needs a grc.store namespace and a trusted-publisher binding first. The Defender badge requires this option, so steer anyone planning to go for Defender here.

Source: [securityslam.com/library/mechanizer](https://securityslam.com/library/mechanizer) and [Set up your grc.store namespace and targets](https://securityslam.com/library/grc-store-setup).

## Critical Rules

- **Never weaken a check to reach green.** Fix the underlying control or leave it failing and explain why. A failing scan still earns Mechanizer; a passing one is the `defender` skill's job.
- **Never change repository or org settings without explicit approval.** Branch protection, MFA enforcement, and default workflow permissions are outward-facing changes. Show the exact change and wait for a yes.
- **Pin every action to a full commit SHA** with a version comment. Resolve SHAs from the GitHub API at generation time. Never guess or reuse a SHA from memory. **One exception:** call the publish workflow by tag, `publish.yml@v1`. The hub accepts a result only when the Sigstore certificate names that exact tag; a SHA-pinned call puts the SHA in the certificate and the hub's identity check fails. Say so in a comment on the `uses:` line so a linter or reviewer does not "fix" it. When the repository has a zizmor config (`.github/zizmor.yml` or `zizmor.yml`), add the exception there as a scoped policy, `rules: unpinned-uses: config: policies: revanite-io/pvtr-publish-results/*: ref-pin`, with the reason as a comment, and leave the `uses:` line without an inline ignore. Otherwise keep the asset's `# zizmor: ignore[unpinned-uses]`.
- **Never trigger the publish workflow on pull requests or pushes.** A schedule, manual dispatch, or releases only. The hub stores one log per target and catalog every ten minutes and rejects the rest with `rate_limited`, which fails the run, so merges that land minutes apart turn the default branch red.
- **Never guess the grc.store namespace.** Ask for the slug and check it exists.
- **Never perform grc.store setup.** Enterprise access, namespaces, and trusted-publisher bindings are created in the grc.store UI by an enterprise or namespace admin. Instruct, then ask the user to confirm. The hub cannot tell a user who their enterprise admins are, and neither can you.
- **Keep scanner tokens away from untrusted code.** Never run a scan on `pull_request_target` or on fork PRs.
- **Start from a fresh scanner run.** Never infer a result the OSPS Baseline scanner reports. See [Scan First](#scan-first).

## Scan First

Before choosing an option, run the OSPS Baseline scanner locally as [the scanner reference](../mechanizer/references/local-scan.md) describes. It is the same `openssf/github-repo` plugin both CI options run, so the local table in step 2 is what the first CI run will report, minus the settings a repository-scoped job token cannot read.

## Workflow

### 1. Choose the option

Ask two questions:

1. **Is Defender a goal?** If yes, go for Option 2: Defender is judged from the grc.store result, so starting there saves a step.
2. **Is the maintainer in a grc.store enterprise?** Their steward's (a foundation) or their employer's. If they do not know, *My namespaces* on grc.store shows which enterprise manages their account. Without one, point them to the Slam organizers for onboarding, then keep going: Option 1 earns the badge today, and the Option 2 workflow, config, and Security Insights entry can be drafted now and committed once the account, namespace, and binding exist.

For a project not hosted on GitHub, tell the user to contact Slam organizers for an alternate evaluation path.

### 2. Run a local baseline scan

Summarize the results of the local run from [Scan First](#scan-first), one table per repository:

| Control | Result | Cause | Fix |
| --- | --- | --- | --- |
| OSPS-AC-03.01 | Failed | No branch protection on `main` | Add a ruleset blocking direct pushes (needs admin approval) |
| OSPS-VM-02.01 | Failed | No security contact found | Add SECURITY.md and link it in Security Insights |
| OSPS-DO-02.01 | Needs Review | Scanner cannot judge prose | Manual check |

Many failures are missing documentation or Security Insights fields. Hand those to the `cleaner` and `chronicler` skills.

Two failures are common on new projects:

| Control | Scanner message | Fix |
| --- | --- | --- |
| OSPS-BR-07.01 | Secret scanning and push protection are both disabled | Turn both on in repository settings. They are free for public repositories. This is a settings change, so get approval first. |
| OSPS-DO-01.01 | User guide was NOT specified in Security Insights data | Set `project.documentation.detailed-guide`. The scanner reads only that field; `quickstart-guide` alone fails. |

### 3. Triage failures (a head start on Defender)

Mechanizer does not require any of these fixed. Work through the table with the user anyway, because every fix here is one less for the `defender` skill. For each settings change, show the exact `gh api` call or UI steps and wait for approval. Re-scan after each batch.

### 4. Option 1: add the action workflow

Create `.github/workflows/osps-baseline.yml` from [assets/osps-baseline.yml](assets/osps-baseline.yml). Before writing it:

1. Resolve the latest release of each action and its commit SHA:

   ```bash
   tag=$(gh api repos/revanite-io/osps-baseline-action/releases/latest --jq .tag_name)
   gh api "repos/revanite-io/osps-baseline-action/commits/$tag" --jq .sha
   ```

   Repeat for `actions/checkout` and `actions/upload-artifact`.

2. Always set `catalog` explicitly to a Baseline catalog the pinned scanner release ships. Never rely on the default: the action's README and its `action.yml` have named different defaults.
3. Choose the scanner token. The action's README says the built-in `GITHUB_TOKEN` does not work. In Option 2 the same scanner accepted the job token and reported the settings it could not read as "Needs Review", so try `${{ github.token }}` first, and move to these options, in order, when controls the token should be able to read come back "Needs Review":
   1. **octo-sts**, when the org has the [octo-sts](https://github.com/octo-sts/app) GitHub App installed. The workflow trades its OIDC identity for a short-lived, read-only token. No secret is stored. See [references/octo-sts-token.md](references/octo-sts-token.md).
   2. **A fine-grained PAT** limited to this repository with read-only access. Store it as the `PVTR_GITHUB_TOKEN` secret.
   3. **A classic PAT** with `public_repo` (or `repo` for private repos), stored the same way. Use it only as a fallback: `public_repo` also grants write access to every public repository the user can push to.

   Never ask the user to paste a token into the conversation, and never create one yourself.

Correct:

```yaml
- uses: revanite-io/osps-baseline-action@0123456789abcdef0123456789abcdef01234567 # v1.5.0
```

Wrong - mutable tag:

```yaml
- uses: revanite-io/osps-baseline-action@v1
```

Wrong - token exposed to fork code:

```yaml
on:
  pull_request_target:
```

The action assesses Maturity Level 1 only. That is expected and still satisfies the badge.

Keep the workflow on a schedule so results stay fresh, and upload results as an artifact. Optionally set `upload-sarif: "true"` to surface failed controls in the Security tab, with osps-baseline-action v1.5.2 or later: through v1.5.1, `fail-on-error: "true"` exited before the SARIF upload, so failed controls reached only the workflow log and the results artifact. Add a status badge for the workflow to the README.

### 5. Option 2: add the publish workflow

#### Prerequisites you instruct, never perform

| Prerequisite | Who does it | What you do |
| --- | --- | --- |
| An account inside the steward's enterprise | An enterprise admin invites the maintainer, or the maintainer requests access through the enterprise's `https://grc.store/request-access?via=<enterprise>` link | Ask whether the maintainer has one. If not, point at the Slam organizers for onboarding and carry on drafting. |
| An enterprise-owned namespace for the project | An enterprise admin creates it with the enterprise under *Owned by*; a member asks an admin | Ask for the slug and check it: `curl -sf https://hub.grc.store/v1/namespaces/SLUG` (404 means it does not exist). Warn when the namespace is personal rather than enterprise-owned: the badge evidence then belongs to one person, and a hub admin has to move it into the enterprise on request. |
| The repository bound as a trusted publisher on that namespace | A namespace admin adds `owner/repo` under *Trusted publishers*, optionally restricted to a ref | State the exact binding: `OWNER/REPO`, with ref `refs/heads/main` when the workflow only runs from the default branch. Ask the user to confirm it exists before the workflow is committed. The public API exposes no binding listing; the check is "the first run succeeds". |
| The target registered and verified | The first successful publish run does it. Nothing manual for a GitHub repository | Tell the user the page to expect, `https://grc.store/targets/NAMESPACE/TARGET`, and check it resolves after the first run. Non-GitHub targets (manual registration plus a DNS or hosted-file challenge) are out of scope: say so and point at the organizers. |
| Target coordinate and license | The maintainer | Derive `NAMESPACE/TARGET@VERSION` from the namespace, the repository name, and the current release tag. Ask for the SPDX license the result is published under; suggest `CC0-1.0`. |

#### Write the workflow

Create `.github/workflows/publish-results.yml` from [assets/publish-results.yml](assets/publish-results.yml) and fill in the namespace, target, owner, repo, license, and level. Points to get right:

- **The scanner token goes inline.** The plugin, published on the hub as `openssf/github-repo`, reads its token only from its config `vars`, so the config is passed through `secrets.config` with `${{ secrets.GITHUB_TOKEN }}` rather than a committed `.pvtr/config.yml`, which could not carry it. The job token is read-only and scoped to the repository. Settings it cannot read (org MFA, some admin settings) report "Needs Review"; that is expected and does not block the badge. This worked on [revanite-io/pvtr-aws-s3](https://github.com/revanite-io/pvtr-aws-s3/blob/main/.github/workflows/publish-results.yaml) on 2026-10-04. octo-sts support is tracked in revanite-io/pvtr-publish-results#10.
- **`applicability` is the Baseline level the hub judges:** `maturity-1`, `maturity-2`, or `maturity-3`. Confirm it with the user; use `maturity-1` when unknown. The `defender` skill reads the published result at this level.
- **Triggers:** a weekly schedule and manual dispatch in the asset. The Defender loop publishes after each merged fix with `gh workflow run`, which keeps runs at least ten minutes apart. A release trigger is also allowed when releases are rare; drop it if releases often land within ten minutes of each other.
- **Version:** the `version` job reads the latest release tag and falls back to `0.0.0` for a project with no releases.

#### First run

```bash
gh workflow run publish-results.yml
gh run watch
curl -sf https://hub.grc.store/v1/targets/NAMESPACE/TARGET | jq '.versions[0].latest | {result, counts, run_at}'
```

The `publish` job's status reports publication, not the evaluation: a failing Baseline is an honest result and still exits 0. Read the verdict from the target page. When the job itself fails, map the error to its cause:

| Publish job error | Cause | Fix |
| --- | --- | --- |
| `forbidden`: bundle sync requires a write role or ownership of the target namespace | No trusted-publisher binding for this repository and ref, or the namespace does not exist | The namespace and binding rows above |
| `target_not_owned` or `results_caller_mismatch` | The `target:` namespace is not one this repository is bound to, or the run came from a ref the binding excludes | Fix `target:`, or the binding's ref |
| `rate_limited` | A second log for this target within ten minutes | Wait, and remove any push or pull-request trigger |
| `results_signer_untrusted` | The workflow was called by SHA or by a tag the hub does not accept | Call `publish.yml@v1` |
| The `run` job fails before publishing | The scanner aborted, usually a token or API problem | Read the run job log and re-run once |

### 6. Record the tooling in Security Insights

Add a `repository.security.tools` entry for the scanner. The `cleaner` skill's [field reference](../cleaner/references/security-insights-fields.md) has both shapes: for Option 1, `results.ci.location` is the workflow URL; for Option 2 it is the grc.store target page and `predicate-uri` is the publish workflow. Re-validate the file.

### 7. Submission checklist

- [ ] A workflow on the default branch runs the scan on a schedule (plus manual dispatch)
- [ ] At least one completed run: a workflow run (Option 1) or a log on the grc.store target page (Option 2)
- [ ] Actions pinned by SHA; the publish workflow called by tag (Option 2)
- [ ] Security Insights lists the scanner with the right `location` and validates
- [ ] Completion notification submitted on the Mechanizer badge page

## Edge Cases

- **Scanner flakes on API errors:** re-run once. If it keeps failing, check token scope and rate limits before blaming the repo.
- **Control fails because the scanner is behind the Baseline version:** note the version gap in the submission and ask a Slam advisor. Do not fake compliance.
- **Org-level controls (MFA):** the MFA check runs only with an org-scoped token. A repository-scoped token (octo-sts, a fine-grained PAT, or the job token in Option 2) reports AC-01.01 as "Needs Review". Ask the user whether an org admin will run that scan. Never request broader scopes than needed.
- **Multi-repo project:** add the workflow to every repository listed in Security Insights. For Option 2, each repository is bound separately and publishes its own target.
- **Personal namespace:** the workflow runs, but the evidence belongs to one person's account. Say so, and point at the hub-admin transfer path in the grc.store setup guide.
- **Alternative tooling:** OpenSSF [Minder](https://mindersec.dev/) publishes Baseline Level 1 rules and can auto-remediate some checks. It helps close gaps but does not replace the two options.
