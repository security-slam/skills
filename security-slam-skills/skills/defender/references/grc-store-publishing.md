# Publishing OSPS Baseline Results to grc.store

Source: the Slam's [grc.store setup guide](https://securityslam.com/library/grc-store-setup) and the [`revanite-io/pvtr-publish-results`](https://github.com/revanite-io/pvtr-publish-results) README.

## How it works

The hub accepts an evaluation result only from the `revanite-io/pvtr-publish-results` reusable workflow, running on a GitHub-hosted runner in a repository the namespace trusts. The workflow installs a signed plugin from the hub, runs it, signs the log with Sigstore, and publishes it. No one publishes results by hand: `pvtr publish`, `grcli publish`, custom workflows, self-hosted runners, and forks are all refused.

A published result proves that an unmodified, hub-verified plugin ran on a GitHub-hosted runner. It does not prove the result describes the named target. The caller's config chooses what the plugin scans, so a correct config is the namespace owner's job.

## Prerequisites

1. **An account inside the steward's enterprise.** An enterprise admin invites the user, or the user requests access through the enterprise's `/request-access?via=<enterprise>` link.
2. **A namespace for the project.** An enterprise admin creates it, owned by the enterprise, and adds the maintainer as a namespace admin. The slug cannot change later. Ask the user for the namespace; never guess it.
3. **A trusted-publisher binding for each publishing repository.** On the namespace admin page, under *Trusted publishers*, add `owner/repo`. The optional git ref, for example `refs/heads/main`, restricts publishing to that ref. Recommend setting it to the default branch. The template's triggers run there.
4. **Nothing for a GitHub repository target.** The first successful run from the bound repository registers the target and marks it verified. Targets that are not GitHub repositories need manual registration and an ownership proof. They are out of scope for the Defender badge.

Check what already exists without an account:

```bash
curl -s "https://hub.grc.store/v1/targets?namespace=NAMESPACE" | jq '.items[] | {target_id, verified_at, evaluation_count}'
```

## Check the hub

The workflow installs the plugin from the hub it publishes to. Check that `openssf/github-repo` is there:

```bash
curl -s https://hub.grc.store/v1/plugins/openssf/github-repo | jq .latest_version
```

## Filling the template

Copy [../assets/grc-store-results.yml](../assets/grc-store-results.yml) to `.github/workflows/grc-store-results.yml` and replace:

| Placeholder | Value |
| --- | --- |
| `NAMESPACE` | The user's namespace |
| `TARGET_ID` | The repository name, lowercase |
| `CATALOG` | An OSPS Baseline catalog the plugin ships, matching the Baseline version the project targets, for example `osps-baseline-2026-08` |
| `LEVEL` | The confirmed maturity level |
| `OWNER`, `REPO` | The repository |

Keep `security-events: write` on the calling job even without `upload-sarif`. GitHub checks every job in the called workflow at startup, and the workflow fails without it.

A manual run defaults to a dry run: it does everything except sign and send, and uploads the bundles and SARIF as artifacts. Have the user run it once by hand, read the job summary, then run it again with the dry run off. Scheduled runs always publish.

## Pinning exception

Pin every other action to a full commit SHA. The `uses:` line for `pvtr-publish-results` is the one exception: the hub compares the signing certificate's workflow ref with one tag, byte for byte, and a SHA pin puts the SHA in the certificate instead. Keep the comment in the template that explains this. The hub records the workflow commit with every stored log, so a moved tag is auditable.

## Scanner token

The template passes the built-in `GITHUB_TOKEN`. It is read-only and scoped to the repository, so controls that read admin settings or org MFA report "needs review". An octo-sts token cannot reach this workflow: a job that calls a reusable workflow runs no steps, and GitHub drops secret outputs passed between jobs. [revanite-io/pvtr-publish-results#10](https://github.com/revanite-io/pvtr-publish-results/issues/10) tracks octo-sts support.

Cover the "needs review" controls with evidence (Workflow step 5) rather than handing the scanner a broader token. If the user still wants one, offer a fine-grained, read-only PAT for this repository, stored as a secret. Never ask the user to paste a token into the conversation.

## Reading the result

The job's color reports publication, not the evaluation. A failing baseline still publishes and exits 0. Red means no log landed. Read the verdict on the target page, `https://grc.store/targets/NAMESPACE/TARGET_ID`, or from the API:

```bash
curl -s "https://hub.grc.store/v1/targets?namespace=NAMESPACE" | jq '.items[] | select(.target_id=="TARGET_ID") | .latest'
```

No `latest` means no published evaluation. That is neither a pass nor a fail. Results older than 30 days show as stale.

## Errors

| Error | Cause | Fix |
| --- | --- | --- |
| 403 | The calling repository is not a trusted publisher for the namespace, or the binding's git ref does not match | Add or fix the binding on the namespace admin page. No secret fixes this. |
| `results_signer_untrusted` | SHA pin, wrong tag, fork, or self-hosted runner | Call `@v1` from the upstream repository on a GitHub-hosted runner |
| `results_caller_mismatch` | The OIDC token's repository or ref differs from the signing certificate's | Run from the bound repository, not a fork |
| `evaluator_unpublished` | The plugin is missing from that hub, unsigned, or yanked | Check the plugin on the hub, or pin a published version in the config |
| `rate_limited` | More than one log for the target and catalog within 10 minutes | Wait. Only that run's log was not stored. Never add a `pull_request` trigger. |

## Multi-repo projects

Each repository is its own target with its own workflow and trusted-publisher binding, all under the project's namespace.
