# Control Evidence Checks

Practical checks for GitHub-hosted projects, grouped by Baseline family. Replace `OWNER/REPO`. Some calls need admin rights; when a call returns 403 or 404, ask the user to check the setting in the UI instead of assuming a gap.

## AC: Access Control

| Control | Check |
| --- | --- |
| AC-01.01 MFA | `gh api orgs/OWNER --jq .two_factor_requirement_enabled` (org admin only) |
| AC-02.01 Least privilege for new collaborators | `gh api orgs/OWNER --jq .default_repository_permission` should be `read` or `none` |
| AC-03.01, 03.02 Protect primary branch | `gh api repos/OWNER/REPO/rules/branches/main` lists active rules. Look for `pull_request` or `update` restrictions and `deletion`. Also check classic protection: `gh api repos/OWNER/REPO/branches/main/protection` |
| AC-04.01 Default CI permissions | `gh api repos/OWNER/REPO/actions/permissions/workflow --jq .default_workflow_permissions` should be `read` |
| AC-04.02 Minimal job permissions | Every workflow sets top-level `permissions: {}` or read-only, with per-job grants. `grep -L '^permissions:' .github/workflows/*.y*ml` lists workflows missing it |

## BR: Build and Release

| Control | Check |
| --- | --- |
| BR-01.01, 01.04 Sanitize untrusted and collaborator input | Search workflows for `${{ github.event.` inside `run:` blocks. Pass values through `env:` instead. [zizmor](https://github.com/zizmorcore/zizmor) finds these |
| BR-01.03 No secrets for untrusted code | Flag `pull_request_target` or `workflow_run` jobs that check out PR head code |
| BR-02.01, 02.02 Unique release versions | `gh release list`; assets carry the version in name or path |
| BR-03.01, 03.02 Encrypted channels | Every official URL in README, Security Insights, and package metadata uses HTTPS |
| BR-04.01 Release log | Release notes or CHANGELOG list functional and security changes |
| BR-05.01 Standard dependency tooling | Build uses the ecosystem package manager with a lockfile |
| BR-06.01 Signed releases | Signatures, Sigstore bundles, or a signed checksum manifest on releases. `gh attestation verify` against a release asset |
| BR-07.01 No committed secrets | Secret scanning enabled: `gh api repos/OWNER/REPO --jq .security_and_analysis.secret_scanning.status`; also run gitleaks or trufflehog on history |
| BR-07.02 Secrets policy | Written policy for storing, accessing, rotating secrets |

## DO: Documentation

See the `chronicler` skill, which also drafts the documentation-shaped GV and VM controls below (GV-01, GV-03, GV-04, VM-01, VM-02, VM-03, VM-04.01, VM-05.01, VM-05.02, VM-06.01). The checks here confirm the documents exist.

## GV: Governance

| Control | Check |
| --- | --- |
| GV-01.01, 01.02 Members and roles | GOVERNANCE.md or MAINTAINERS lists who holds sensitive access and what each role does |
| GV-02.01 Public discussion | Issues or Discussions enabled: `gh api repos/OWNER/REPO --jq '.has_issues, .has_discussions'` |
| GV-03.01, 03.02 Contribution guide | CONTRIBUTING.md explains the process and acceptance requirements. GV-03.01 also passes if the docs clearly state that public contributions are not accepted. |
| GV-04.01 Review before escalated access | Governance doc states how people are vetted before getting write or admin |

## LE: Legal

| Control | Check |
| --- | --- |
| LE-01.01 Contributor assertion on every commit | DCO app or check enforced, or CLA bot required. count `Signed-off-by` lines in `git log -20 --format=%B` for a quick DCO signal |
| LE-02.01, 02.02 OSI or FSF license | `gh api repos/OWNER/REPO/license --jq .license.spdx_id` |
| LE-03.01, 03.02 License file present in repo and releases | LICENSE, COPYING, `LICENSES/` (REUSE layout), or `LICENSE/` at the root; release archives include it |

## QA: Quality

| Control | Check |
| --- | --- |
| QA-01.01, 01.02 Public repo with history | Repository is public; history is not rewritten on main |
| QA-02.01 Dependency list | Manifest and lockfile present (go.mod, package-lock.json, Cargo.lock, and so on) |
| QA-02.02 SBOM with compiled releases | Release assets include an SPDX or CycloneDX SBOM |
| QA-03.01 Status checks pass | Ruleset requires status checks on main |
| QA-04.01, 04.02 Subprojects | Security Insights `project.repositories` lists all repos; Level 3 audits each |
| QA-05.01, 05.02 No generated or opaque binaries | run `file` over `git ls-files` output and look for executables or archives; justify any hits |
| QA-06.01 Tests in CI | A workflow runs tests on pull requests |
| QA-06.02, 06.03 Test docs and policy | CONTRIBUTING.md says when and how tests run and that major changes need tests |
| QA-07.01 Non-author approval | Ruleset `pull_request` rule with `required_approving_review_count` of at least 1 |

## SA: Security Assessment

| Control | Check |
| --- | --- |
| SA-01.01 Design docs | Architecture doc showing actors and actions |
| SA-02.01 External interfaces | API, CLI, and config reference docs |
| SA-03.01, 03.02 Assessment and threat model | See the `inspector` skill |

## VM: Vulnerability Management

| Control | Check |
| --- | --- |
| VM-01.01 CVD policy with response time | SECURITY.md states a response timeframe |
| VM-02.01 Security contacts | SECURITY.md or Security Insights names a contact |
| VM-03.01 Private reporting | `gh api repos/OWNER/REPO/private-vulnerability-reporting --jq .enabled` or a private email |
| VM-04.01 Published vulnerability data | GitHub Security Advisories or an advisories page |
| VM-04.02 VEX for non-affecting vulns | OpenVEX or CSAF VEX documents published with releases |
| VM-05.01, 05.02, 05.03 SCA policy and enforcement | Written threshold policy plus a blocking SCA check on PRs (for example osv-scanner, dependency-review-action) |
| VM-06.01, 06.02 SAST policy and enforcement | Written threshold policy plus a blocking SAST check on PRs (for example CodeQL, Semgrep) |
