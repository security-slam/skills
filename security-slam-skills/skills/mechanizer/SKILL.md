---
name: mechanizer
description: Earn the Security Slam Mechanizer badge by wiring a live, recurring OSPS Baseline scan into the project's default branch, through either the OSPS Baseline GitHub Action or the grc.store publish workflow (pvtr-publish-results). Use when the user mentions the Mechanizer badge, Security Slam, automated baseline evaluation, OSPS Baseline scanner, pvtr-github-repo-scanner, Privateer, osps-baseline-action, pvtr-publish-results, or grc.store. Sets up the scan, triages failures, and records the tooling in Security Insights.
---

# Mechanizer Badge

The Mechanizer badge requires OSPS Baseline evaluation that runs on its own: a live, recurring scan on the default branch, noted in the Security Insights file. A passing score is not required. Two paths qualify, and both run the same scanner ([Privateer](https://privateerproj.com) with the OpenSSF [pvtr-github-repo-scanner](https://github.com/ossf/pvtr-github-repo-scanner) plugin):

- **Option 1: OSPS Baseline scanner action.** Results stay in the repository: a workflow artifact, the job log, or SARIF in the Security tab. It is the fastest start and has no rate limits.
- **Option 2: grc.store publish workflow.** The [`revanite-io/pvtr-publish-results`](https://github.com/revanite-io/pvtr-publish-results) reusable workflow runs the same scan and publishes each result to a public target page on grc.store. It needs a namespace and a trusted-publisher binding first, and the hub stores one result per target every ten minutes.

The Defender badge requires Option 2. If the project plans to go that far, recommend starting there.

Source: [securityslam.com/library/mechanizer](https://securityslam.com/library/mechanizer).

## Critical Rules

- **Never weaken a check to reach green.** Fix the underlying control or leave it failing and explain why.
- **Never change repository or org settings without explicit approval.** Branch protection, MFA enforcement, and default workflow permissions are outward-facing changes. Show the exact change and wait for a yes.
- **Pin every action to a full commit SHA** with a version comment. Resolve SHAs from the GitHub API at generation time. Never guess or reuse a SHA from memory. The one exception is the `pvtr-publish-results` call in Option 2, which the hub requires by tag.
- **Keep scanner tokens away from untrusted code.** Never run the scan on `pull_request_target` or on fork PRs.

## Workflow

### 1. Choose the path

Ask whether the project is going for the Defender badge. If yes, or the user wants public results, use Option 2. Otherwise Option 1 is faster. Doing both is fine.

For a project not hosted on GitHub, tell the user to contact Slam organizers for an alternate evaluation path.

### 2. Run a local baseline scan

Get the current picture before touching CI. Follow the [pvtr-github-repo-scanner README](https://github.com/ossf/pvtr-github-repo-scanner) to run it locally against the repo with a read-only token. Summarize the results:

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

### 3. Fix failures

The badge does not need a passing scan, so this step is optional. It is the bulk of the Defender badge, so offer it. Work through the table with the user. For each settings change, show the exact `gh api` call or UI steps and wait for approval. Re-scan after each batch.

### 4a. Add the scanner workflow (Option 1)

Create `.github/workflows/osps-baseline.yml` from [assets/osps-baseline.yml](assets/osps-baseline.yml). Before writing it:

1. Resolve the latest release of each action and its commit SHA:

   ```bash
   tag=$(gh api repos/revanite-io/osps-baseline-action/releases/latest --jq .tag_name)
   gh api "repos/revanite-io/osps-baseline-action/commits/$tag" --jq .sha
   ```

   Repeat for `actions/checkout` and `actions/upload-artifact`.

2. Always set `catalog` explicitly to a Baseline catalog the pinned scanner release ships. Never rely on the default: the action's README and its `action.yml` have named different defaults.
3. Choose the scanner token. The built-in `GITHUB_TOKEN` does not work. Offer these in order:
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

### 4b. Add the publish workflow (Option 2)

Use the `defender` skill's template, [../defender/assets/grc-store-results.yml](../defender/assets/grc-store-results.yml), and its [publishing reference](../defender/references/grc-store-publishing.md) for the grc.store prerequisites, the tag-pinning exception, the token, and error codes. The template runs weekly and on demand. Never add a `pull_request` trigger: the hub refuses more than one result per target every ten minutes.

### 5. Publish results

- Keep the workflow on a schedule, or on pushes to the default branch, so results stay fresh.
- **Option 1:** upload results as an artifact. Optionally set `upload-sarif: "true"` to surface failed controls in the Security tab. Use osps-baseline-action v1.5.2 or later. Through v1.5.1, `fail-on-error: "true"` exited before the SARIF upload, so failed controls reached only the workflow log and the results artifact (`pvtr/pvtr.sarif`). v1.5.1 also showed two-digit counts wrong in the workflow summary. The badge does not need a pass, so `fail-on-error: "false"` keeps a failing scan from turning every run red.
- **Option 2:** the target page on grc.store is the published result. Link it from the README.
- Optionally add a status badge for the workflow to the README.

### 6. Record the tooling in Security Insights

Add a `repository.security.tools` entry for the scanner. The `cleaner` skill's [field reference](../cleaner/references/security-insights-fields.md) has a complete example. Re-validate the file.

### 7. Submission checklist

- [ ] A scan workflow runs on the default branch on a schedule or on push, and its latest run completed
- [ ] Every action pinned by SHA, except the `pvtr-publish-results` tag (Option 2)
- [ ] Results published: artifact or Security tab (Option 1), or the grc.store target page (Option 2)
- [ ] Security Insights lists the scanner and validates
- [ ] Completion notification submitted on the Mechanizer badge page

## Edge Cases

- **Scanner flakes on API errors:** re-run once. If it keeps failing, check token scope and rate limits before blaming the repo.
- **Control fails because the scanner is behind the Baseline version:** note the version gap in the submission and ask a Slam advisor. Do not fake compliance.
- **Org-level controls (MFA):** the MFA check runs only when the token has `admin:org`. A repository-scoped octo-sts token or fine-grained PAT does not, so expect AC-01.01 to report "needs review". Ask the user whether an org admin will run that scan. Never request broader scopes than needed.
- **Multi-repo project:** add the workflow to every repository listed in Security Insights, or run it centrally with a matrix over the repos.
- **Alternative tooling:** OpenSSF [Minder](https://mindersec.dev/) publishes Baseline Level 1 rules and can auto-remediate some checks. It helps close gaps but does not replace either scan path.
