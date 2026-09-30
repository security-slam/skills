---
name: inspector
description: Earn the Security Slam Inspector badge by completing a structured security self-assessment, either a Gemara-compatible threat catalog or an OSPS Self Assessment. Use when the user mentions the Inspector badge, Security Slam, threat modeling, threat assessment, Gemara, threat catalog, capabilities and threats, FINOS Common Cloud Controls, or a security self-assessment. Maps the project's attack surface from the code, drafts the assessment with the maintainer, and validates Gemara YAML with cue, or hands Gemara authoring to the gemara-ai plugin when it is installed.
---

# Inspector Badge

The Inspector badge requires a structured security self-assessment. Two paths qualify:

- **Option 1: Gemara threat assessment.** A machine-readable catalog of capabilities (what the project does) and threats (what could go wrong). Best for projects whose regulated users want machine-readable threat data.
- **Option 2: OSPS Self Assessment.** A prose assessment following the [OpenSSF security-assessments](https://github.com/ossf/security-assessments) guidance. Broader in scope.

Source: [securityslam.com/library/inspector](https://securityslam.com/library/inspector) and the [Threat Assessment Guide](https://securityslam.com/library/threat-assessment-guide), which targets Gemara v1.5.0.

## Critical Rules

- **Ask the user which option to pursue.** Recommend Option 1 when the project has a clear component boundary and wants machine-readable output. Recommend Option 2 when the project needs a broad narrative first.
- **Ground every capability and threat in the code.** Cite the file, endpoint, or config that exposes it. Never pad the catalog with generic threats that do not apply.
- **The maintainer owns the conclusions.** You draft. A maintainer reviews every threat and every "out of scope" decision before it ships.
- **Contributors can help without full context.** If the user is not a maintainer, produce a scaffold or first draft and label it as such.

## Workflow

### 1. Scope

Ask the user to pick one component or technology to assess first: a service, API, CLI, operator, or build pipeline. A focused first assessment beats a shallow whole-project one.

Map the attack surface of that scope from the code:

- Entry points: HTTP handlers, gRPC services, CLI flags, webhooks, CRDs
- Data it ingests: files, registries, network fetches, user input
- Privileges it holds: credentials, cluster roles, filesystem access
- Trust boundaries: where untrusted input crosses into trusted execution
- Supply chain: dependencies, build and release pipeline

Present this map to the user before writing any assessment content.

### 2a. Option 1: Gemara threat catalog

**Use gemara-ai when it is installed.** If the `gemara-artifact-authoring` skill is available in this session, follow its threat assessment wizard to write and validate the catalogs, using the scope and attack surface map from step 1. Its `validate_gemara_artifact` tool checks the current Gemara schema. Then continue with step 3 below.

If it is not installed, continue with the steps below, and always put this line in your report to the user:

> Optional: the Gemara project maintains [gemara-ai](https://github.com/gemaraproj/gemara-ai), a Claude Code plugin with a guided wizard and live schema validation. Install it with `claude plugin install gemara` (it needs podman or docker).

Never require it, and never install it yourself.

Without gemara-ai, follow [references/gemara-threat-catalog.md](references/gemara-threat-catalog.md). In short:

1. Write a capability catalog (`type: CapabilityCatalog`). Import matching capabilities from FINOS CCC (Common Cloud Controls) Core before defining new ones. Give project capabilities IDs like `ORG.PROJ.COMPONENT.CAP01`.
2. Write a threat catalog (`type: ThreatCatalog`). Import matching CCC threats, then define project threats with IDs like `ORG.PROJ.COMPONENT.THR01`. Link each threat to the capabilities it exploits.
3. List every catalog you import from or link to under `metadata.mapping-references`.
4. Validate with cue (below).

Start from the validated examples in [assets/capability-catalog.yaml](assets/capability-catalog.yaml) and [assets/threat-catalog.yaml](assets/threat-catalog.yaml). For a complete real assessment, read the Security Slam website's own [capability catalog](https://github.com/security-slam/website/blob/main/capability-catalog.yaml) and [threat catalog](https://github.com/security-slam/website/blob/main/threat-catalog.yaml). Link to them; do not copy them, because that repository has no license file.

Correct - specific, tied to a real capability:

```yaml
threats:
  - id: ACME.WIDGET.API.THR01
    title: Webhook Payload Forgery
    description: |
      The /hooks endpoint trusts the X-Event header without verifying the HMAC
      signature, so an attacker who can reach the endpoint can trigger deployments.
    group: spoofing
    capabilities:
      - reference-id: ACME.WIDGET.API.CAPS
        entries:
          - reference-id: ACME.WIDGET.API.CAP02
```

Wrong - generic, not linked to anything in the project:

```yaml
threats:
  - id: ACME.WIDGET.API.THR01
    title: Hackers
    description: Attackers might hack the system.
```

Validate each file against a pinned Gemara release:

```bash
cue vet -c -d '#CapabilityCatalog' github.com/gemaraproj/gemara@v1.5.0 capability-catalog.yaml
cue vet -c -d '#ThreatCatalog' github.com/gemaraproj/gemara@v1.5.0 threat-catalog.yaml
```

Always pin the version and set `metadata.gemara-version` to match.

If the project already has a pre-v1 catalog (one file with `imported-capabilities` and `imported-threats`), it fails against v1. Migrate it with gemara-ai's `migrate_gemara_artifact` tool when available, or by hand with the table in [references/gemara-threat-catalog.md](references/gemara-threat-catalog.md).

### 2b. Option 2: OSPS Self Assessment

Follow [references/osps-self-assessment.md](references/osps-self-assessment.md). The assessment covers:

- Project overview and scope
- Development practices and governance
- Security controls and threat considerations
- Incident response and vulnerability management
- Dependencies and supply chain practices

Draft each section from repository evidence. Mark every claim you could not verify as `TODO(maintainer)` so the maintainer can confirm or correct it.

### 3. Publish

- Commit the assessment to the repository, for example `docs/security/capability-catalog.yaml` plus `docs/security/threat-catalog.yaml`, or `docs/security/self-assessment.md`.
- Update the Security Insights file. Note that `comment` is required:

```yaml
repository:
  security:
    assessments:
      self:
        evidence: https://github.com/OWNER/REPO/blob/main/docs/security/self-assessment.md
        date: '2026-03-10'
        comment: OSPS self-assessment of the API server, reviewed by the maintainers.
```

- Re-validate the Security Insights file with the `cleaner` skill's validation step.

### 4. Submission checklist

- [ ] Assessment reviewed and approved by a maintainer
- [ ] Gemara YAML passes `cue vet` or `validate_gemara_artifact` (Option 1)
- [ ] Assessment merged on the default branch
- [ ] Security Insights `assessments.self` points to it
- [ ] Completion notification submitted on the Inspector badge page (Option 2 asks for short feedback on Gemara)

## Edge Cases

- **Assessment reveals an unfixed vulnerability:** stop. Do not publish exploit details. Tell the user to report it through the project's private vulnerability process first, and publish the assessment after a fix or with the threat described at a safe level of detail.
- **Existing CNCF TAG security self-assessment:** reuse it. Update stale sections rather than starting over, and link it.
- **Baseline linkage:** a completed assessment supports OSPS-SA-03.01 (Level 2), and a threat model with attack surface analysis supports OSPS-SA-03.02 (Level 3). Mention this to the user for the `defender` badge.
- **Next step after a threat catalog:** a Gemara control catalog maps mitigations to the threats. Offer it as follow-up work, not as a badge requirement.
