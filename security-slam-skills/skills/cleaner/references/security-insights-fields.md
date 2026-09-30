# Security Insights Fields

Source: [Security Insights v2.2.0 schema](https://github.com/ossf/security-insights/blob/v2.2.0/spec/schema.md). Re-check the schema when `header.schema-version` changes.

## Required Fields

| Path | Type | Notes |
| --- | --- | --- |
| `header.schema-version` | `X.Y.Z` | Must match a tag in `ossf/security-insights` (`2.2.0` means tag `v2.2.0`). |
| `header.last-updated` | date | Quoted `YYYY-MM-DD`. |
| `header.last-reviewed` | date | Quoted `YYYY-MM-DD`. |
| `header.url` | URL | Raw URL of this file on the default branch. |
| `project.name` | string | Required when a `project` section is present. |
| `project.administrators[]` | contact | At least one. |
| `project.repositories[]` | name, url, comment | All three required per entry. |
| `project.vulnerability-reporting.reports-accepted` | bool | |
| `project.vulnerability-reporting.bug-bounty-available` | bool | |
| `repository.url` | URL | |
| `repository.status` | string | A repostatus.org value. |
| `repository.accepts-change-request` | bool | |
| `repository.accepts-automated-change-request` | bool | |
| `repository.core-team[]` | contact | At least one. |
| `repository.license.url`, `.expression` | URL, SPDX | |
| `repository.security.assessments` | object | Required even with no assessment. Every assessment entry requires `comment`; `evidence` and `date` are optional. |
| `repository.release.automated-pipeline` | bool | Required only when `release` is present. |
| `repository.release.distribution-points[]` | link | Required only when `release` is present. |

A file may omit `project` and inherit it through `header.project-si-source`.

## Where Badge Evidence Goes

This table shows where Slam badge work usually lands. It is a practical guide, not an official mapping. The Baseline control IDs come from the 2026-02-19 catalog.

| Field | Evidence | Related Baseline controls | Badge |
| --- | --- | --- | --- |
| `project.vulnerability-reporting.contact`, `.policy` | SECURITY.md, CVD (coordinated vulnerability disclosure) policy | VM-01.01, VM-02.01, VM-03.01 | Cleaner, Chronicler |
| `project.documentation.detailed-guide` | User guides. The OSPS Baseline scanner reads only this field for DO-01.01, so always set it when a user guide exists. `quickstart-guide` is optional and does not satisfy the scanner alone. | DO-01.01 | Cleaner, Chronicler |
| `repository.documentation.contributing-guide` | CONTRIBUTING.md, including how to report defects | DO-02.01, GV-03.01, GV-03.02 | Chronicler |
| `repository.documentation.dependency-management-policy` | How dependencies are selected, obtained, tracked | DO-06.01 | Chronicler |
| `project.documentation.signature-verification` | How to verify release integrity and author identity | DO-03.01, DO-03.02 | Chronicler |
| `project.documentation.support-policy` | Support scope, duration, and end of security updates | DO-04.01, DO-05.01 | Chronicler, CRA |
| `project.documentation.design` | Design documentation of actors and actions | SA-01.01 | Defender |
| `repository.documentation.governance` | Roles, responsibilities, members with sensitive access | GV-01.01, GV-01.02 | Defender |
| `repository.documentation.review-policy` | Review requirements | QA-07.01, GV-04.01 | Defender |
| `repository.license` | LICENSE file | LE-02.01, LE-03.01 | Cleaner, CRA |
| `repository.release.changelog` | Release notes with security fixes | BR-04.01 | Defender, CRA |
| `repository.release.attestations[]` | SBOM, provenance, VEX | BR-06.01, QA-02.02, VM-04.02 | Defender |
| `repository.security.assessments.self` | Self-assessment or threat model | SA-03.01, SA-03.02 | Inspector |
| `repository.security.tools[]` | Baseline scanner, SCA, SAST | VM-05.03, VM-06.02 | Mechanizer |
| `project.repositories[]` | Every codebase in the project | QA-04.01 | Cleaner |

## Security Tool Entry

Use this shape when recording a tool, for example the OSPS Baseline scanner added for the Mechanizer badge:

```yaml
repository:
  security:
    tools:
      - name: OSPS Baseline Scanner
        type: other
        rulesets:
          - osps-baseline-2026-02
        integration:
          adhoc: false
          ci: true
          release: false
        results:
          ci:
            name: OSPS Baseline evaluation
            predicate-uri: https://github.com/revanite-io/osps-baseline-action
            location: https://github.com/OWNER/REPO/actions/workflows/osps-baseline.yml
```

`type` must be one of `fuzzing`, `container`, `secret`, `SCA`, `SAST`, or `other`. `rulesets`, `integration`, and `results` are required. `integration` requires all three booleans. `results.*` entries require `name`, `predicate-uri`, and `location`.
