# octo-sts Token for the Baseline Scan

[octo-sts](https://github.com/octo-sts/app) is a GitHub App that trades a workflow's OIDC identity for a short-lived GitHub App token. The token is read-only here, expires in about an hour, and needs no stored secret.

This setup passed the OSPS Baseline scan with zero failed controls on [security-slam/skills](https://github.com/security-slam/skills/blob/main/.github/workflows/osps-baseline.yaml).

## Prerequisites

- The octo-sts GitHub App is installed on the org or repository. Check with `gh api orgs/ORG/installations --jq '.installations[].app_slug'` (needs org admin). Ask the user if you cannot check.
- The app's installation grants at least the permissions listed below. A trust policy can only request a subset of them.

## Trust Policy

Create `.github/chainguard/osps-baseline.sts.yaml` on the default branch. octo-sts reads trust policies from the default branch only, so the token exchange works only after this file merges.

```yaml
issuer: https://token.actions.githubusercontent.com
subject: repo:OWNER/REPO:ref:refs/heads/main
claim_pattern:
  job_workflow_ref: OWNER/REPO/.github/workflows/osps-baseline.yaml@refs/heads/main

# Read-only: the OSPS Baseline scanner only inspects repository settings and files
permissions:
  actions: read
  administration: read
  contents: read
  security_events: read
```

Repositories created after mid-2026 may use GitHub's immutable OIDC subject format, `repo:OWNER@OWNER_ID/REPO@REPO_ID:ref:refs/heads/main`. Get the IDs with `gh api orgs/OWNER --jq .id` and `gh api repos/OWNER/REPO --jq .id`. Match whatever format existing trust policies in the repository already use.

## Workflow Changes

Grant `id-token: write` to the job, add the octo-sts step before the scan, and pass its output as the token:

```yaml
    permissions:
      contents: read # Clone the repository
      id-token: write # Federate with octo-sts for a read-only scanner token
      security-events: write # Upload failed controls as SARIF
    steps:
      - name: Get a read-only token from octo-sts
        uses: octo-sts/action@SHA # vX.Y.Z
        id: octo-sts
        with:
          scope: ${{ github.repository }}
          identity: osps-baseline

      # ... checkout ...

      - name: OSPS Baseline scan
        uses: revanite-io/osps-baseline-action@SHA # vX.Y.Z
        with:
          token: ${{ steps.octo-sts.outputs.token }}
```

`identity` must match the trust policy file name without `.sts.yaml`. Resolve the octo-sts action SHA from its release tag. If the GitHub API refuses to read the `octo-sts` org (SAML enforcement), use `git ls-remote --tags https://github.com/octo-sts/action.git`.

## Limits

- The token has no org-level permissions, so the MFA control (AC-01.01) reports "needs review".
- The trust policy pins the workflow to the default branch. Pull requests cannot get a token, which keeps fork code away from it.
