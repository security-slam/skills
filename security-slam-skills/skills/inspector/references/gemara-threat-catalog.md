# Gemara Threat Catalog

Schema source: [gemaraproj/gemara v1.5.0](https://github.com/gemaraproj/gemara/tree/v1.5.0) (`capabilitycatalog.cue`, `threatcatalog.cue`, `collections.cue`, `metadata.cue`). Method source: the Security Slam [Threat Assessment Guide](https://securityslam.com/library/threat-assessment-guide) by Jennifer Power.

## The Idea

Think of the project as a house. **Capabilities** are what the house does ("allow entry", "store belongings"). **Threats** are how those capabilities get abused ("entry through an unlocked door"). Capabilities form the attack surface, because every intended function is a possible path for unintended use.

## v1 File Layout

Gemara v1 splits the work into two catalogs. Both share the `#Catalog` base.

| Field | Required | Notes |
| --- | --- | --- |
| `title` | Yes | Display name. |
| `metadata.id` | Yes | Unique ID, for example `ACME.WIDGET.API.CAPS`. |
| `metadata.type` | Yes | `CapabilityCatalog` or `ThreatCatalog`. |
| `metadata.gemara-version` | Yes | The Gemara release you validate against, quoted. |
| `metadata.description` | Yes | |
| `metadata.author` | Yes | `id`, `name`, `type` (`Human`, `Software`, or `Software Assisted`). |
| `metadata.mapping-references` | When importing | One entry (`id`, `title`, `version`) per external catalog you reference. |
| `groups` | When entries exist | Every capability or threat must name a `group` defined here. |
| `imports` | No | Pull whole entries from another catalog by `reference-id`. |

A capability requires `id`, `title`, `description`, and `group`.

A threat requires `id`, `title`, `description`, `group`, and `capabilities` (a list of `reference-id` plus `entries`). `vectors` and `actors` are optional.

Mark AI-drafted catalogs honestly: set `author.type: Software Assisted` until a human maintainer reviews it, then switch to `Human`.

## Step by Step

### Step 0: Scope

Pick one component. Check the [FINOS CCC Core catalog](https://github.com/finos/common-cloud-controls/releases) for capabilities and threats you can import instead of writing from scratch.

### Step 1: Capabilities

Ask "which common capabilities does this component have?" Import those from CCC. Then define the capabilities unique to the project.

Example: a container management tool pulls images from registries (CCC.Core.CP29, Active Ingestion) and resolves tags to versions (CCC.Core.CP18, Resource Versioning). It also retrieves images by mutable tag, which is project specific (CAP01).

### Step 2: Threats

For each capability, ask "how could this be misused?" Import linked CCC threats that fit. CCC.Core.TH14 (Older Resource Versions are Used) links to CP18 and applies because mutable tags can resolve to stale or compromised images. Then define project threats. Link each one to the exact capabilities it exploits, across catalogs.

Group threats by a scheme the team understands. STRIDE (Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege) works well.

### Step 3: Validate

```bash
cue vet -c -d '#CapabilityCatalog' github.com/gemaraproj/gemara@v1.5.0 capability-catalog.yaml
cue vet -c -d '#ThreatCatalog' github.com/gemaraproj/gemara@v1.5.0 threat-catalog.yaml
```

Silent output means valid. cue checks shape, unique IDs, and that each entry's `group` exists. It does not check that cross-catalog references resolve, so verify those by hand.

## Migrating a Pre-v1 Catalog

The Slam 2026 guide targeted Gemara v0.19.x, which used one file:

| Pre-v1 (single file) | v1 |
| --- | --- |
| `imported-capabilities` | `imports` in the capability catalog |
| `capabilities` | `capabilities` in the capability catalog, each with a `group` |
| `imported-threats` | `imports` in the threat catalog |
| `threats` | `threats` in the threat catalog, each with a `group` |
| No `metadata.type` | `metadata.type` required |
| No `metadata.gemara-version` | `metadata.gemara-version` required |
| Threat capability refs to own ID | Refs point at the capability catalog's `metadata.id` |

## Next Step

A Gemara control catalog maps mitigations to these threats. Offer it as follow-up work. It is not required for the Inspector badge.
