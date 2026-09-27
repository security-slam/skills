# OSPS Self Assessment

Source: [ossf/security-assessments](https://github.com/ossf/security-assessments) and the Security Slam [Inspector badge page](https://securityslam.com/library/inspector). Before drafting, fetch the current template from that repository and follow its headings if they differ from this outline.

The CNCF TAG Security self-assessment ([example for k3s](https://github.com/cncf/toc/blob/e67e3e4e4396f131913cd226fc0fb0db04e7e67f/projects/k3s/self-assessment.md)) follows a similar shape and is a good worked reference.

## Outline

### 1. Project Overview and Scope

- What the project does, in two or three sentences
- Actors: users, operators, external systems, and how they interact
- Actions: the main flows, with a diagram if one exists
- In scope and out of scope for this assessment
- Security goals and non-goals

### 2. Development Practices and Governance

- Who has merge and release rights, and how they get them
- Code review requirements and branch protection
- CI checks that gate merges
- Contributor identity and DCO (Developer Certificate of Origin) or CLA (Contributor License Agreement) policy

### 3. Security Controls and Threat Considerations

- Authentication, authorization, and secrets handling
- Trust boundaries and how untrusted input is handled
- Top threats: for each, the attack, the impact, and the existing mitigation
- Known gaps and planned work

### 4. Incident Response and Vulnerability Management

- How to report a vulnerability (link SECURITY.md)
- Triage, fix, disclosure, and advisory process with timelines
- Past advisories and how they were handled

### 5. Dependencies and Supply Chain

- How dependencies are chosen, pinned, and updated
- SCA and SAST tooling
- Build and release pipeline, signing, SBOM, and provenance

## Drafting Rules

- Fill each section from repository evidence and cite the file or URL.
- Mark anything you cannot verify as `TODO(maintainer): ...`.
- Keep threat details at a level safe to publish. Report unfixed issues privately first.
- Add a "Last reviewed" date at the top so readers can judge freshness.
