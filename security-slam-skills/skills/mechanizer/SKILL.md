---
name: mechanizer
description: Earn the Security Slam Mechanizer badge by automating OSPS Baseline evaluation and publishing the results, through either LFX Insights (100% on Security and Best Practices) or the OSPS Baseline GitHub Action (zero failed controls). Use when the user mentions the Mechanizer badge, Security Slam, automated baseline evaluation, OSPS Baseline scanner, pvtr-github-repo-scanner, Privateer, osps-baseline-action, or LFX Insights security score. Sets up the scan, triages failures, and records the tooling in Security Insights.
---

# Mechanizer Badge

The Mechanizer badge requires automated OSPS Baseline evaluation with published results. Two paths qualify:

- **Option 1: LFX Insights (recommended by the Slam).** Reach 100% on the project's Security & Best Practices dashboard. LFX Insights curates a subset of controls. Onboarding takes longer, but 100% is easier once the project is in.
- **Option 2: OSPS Baseline GitHub Action.** Reach zero failed controls. It checks every control the scanner can verify automatically. It starts faster but demands more.

Both use the same scanner ([Privateer](https://privateerproj.com) with the [pvtr-github-repo-scanner](https://github.com/ossf/pvtr-github-repo-scanner) plugin).

Source: [securityslam.com/library/mechanizer](https://securityslam.com/library/mechanizer).

## Critical Rules

- **Never weaken a check to reach green.** Fix the underlying control or leave it failing and explain why.
- **Never change repository or org settings without explicit approval.** Branch protection, MFA enforcement, and default workflow permissions are outward-facing changes. Show the exact change and wait for a yes.
- **Pin every action to a full commit SHA** with a version comment. Resolve SHAs from the GitHub API at generation time. Never guess or reuse a SHA from memory.
- **Keep scanner tokens away from untrusted code.** Never run the scan on `pull_request_target` or on fork PRs.

## Workflow

### 1. Choose the path

Check LFX Insights first:

- Search [insights.linuxfoundation.org](https://insights.linuxfoundation.org/) for the project.
- If it is listed, open its Security & Best Practices page and note the score and failing checks. Prefer Option 1.
- If it is not listed, tell the user onboarding goes through a [project onboarding discussion](https://github.com/linuxfoundation/insights/discussions/categories/project-onboardings) and depends on community upvotes. Offer Option 2 in the meantime. Doing both is fine.

For a project not hosted on GitHub, tell the user to contact Slam organizers for an alternate evaluation path.

### 2. Run a local baseline scan

Get the current picture before touching CI. Follow the [pvtr-github-repo-scanner README](https://github.com/ossf/pvtr-github-repo-scanner) to run it locally against the repo with a read-only token. Summarize the results:

| Control | Result | Cause | Fix |
| --- | --- | --- | --- |
| OSPS-AC-03.01 | Failed | No branch protection on `main` | Add a ruleset blocking direct pushes (needs admin approval) |
| OSPS-VM-02.01 | Failed | No security contact found | Add SECURITY.md and link it in Security Insights |
| OSPS-DO-02.01 | Needs Review | Scanner cannot judge prose | Manual check |

Many failures are missing documentation or Security Insights fields. Hand those to the `cleaner` and `chronicler` skills.

### 3. Fix failures

Work through the table with the user. For each settings change, show the exact `gh api` call or UI steps and wait for approval. Re-scan after each batch.

### 4. Add the CI workflow (Option 2)

Create `.github/workflows/osps-baseline.yml` from [assets/osps-baseline.yml](assets/osps-baseline.yml). Before writing it:

1. Resolve the latest release of each action and its commit SHA:

   ```bash
   tag=$(gh api repos/revanite-io/osps-baseline-action/releases/latest --jq .tag_name)
   gh api "repos/revanite-io/osps-baseline-action/commits/$tag" --jq .sha
   ```

   Repeat for `actions/checkout` and `actions/upload-artifact`.

2. Set `catalog` to the Baseline catalog the scanner release supports. Use the action README's default unless the user needs an older catalog.
3. Tell the user to create a PAT with `public_repo` scope (or `repo` for private repos) and store it as the `PVTR_GITHUB_TOKEN` secret. The built-in `GITHUB_TOKEN` does not work. Never ask the user to paste the token into the conversation.

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

### 5. Publish results

- Keep the workflow on a schedule so results stay fresh, and upload results as an artifact.
- Optionally set `upload-sarif: "true"` to surface failed controls in the Security tab.
- Add a status badge for the workflow to the README.

### 6. Record the tooling in Security Insights

Add a `repository.security.tools` entry for the scanner. The `cleaner` skill's [field reference](../cleaner/references/security-insights-fields.md) has a complete example. Re-validate the file.

### 7. Submission checklist

- [ ] LFX Insights shows 100% Security & Best Practices, **or** the latest scheduled scan shows zero failed controls
- [ ] Workflow pinned by SHA and running on a schedule
- [ ] Results published (artifact, Security tab, or LFX dashboard link)
- [ ] Security Insights lists the scanner and validates
- [ ] Completion notification submitted on the Mechanizer badge page

## Edge Cases

- **Scanner flakes on API errors:** re-run once. If it keeps failing, check token scope and rate limits before blaming the repo.
- **Control fails because the scanner is behind the Baseline version:** note the version gap in the submission and ask a Slam advisor. Do not fake compliance.
- **Org-level controls (MFA):** the MFA check runs only when the token has `admin:org`. Ask the user whether an org admin will run that scan. Never request broader scopes than needed.
- **Multi-repo project:** add the workflow to every repository listed in Security Insights, or run it centrally with a matrix over the repos.
- **Alternative tooling:** OpenSSF [Minder](https://mindersec.dev/) publishes Baseline Level 1 rules and can auto-remediate some checks. It helps close gaps but does not replace the badge's scoring paths.
