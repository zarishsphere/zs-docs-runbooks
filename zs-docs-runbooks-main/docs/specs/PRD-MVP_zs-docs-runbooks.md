# PRD-MVP — `zs-docs-runbooks`

> **Document:** Product Requirements (MVP) | **Version:** 1.0.0-mvp
> **Repository:** [https://github.com/zarishsphere/zs-docs-runbooks](https://github.com/zarishsphere/zs-docs-runbooks)
> **Layer:** Layer 0 — Governance | **Catalog #:** 3
> **Language:** Markdown | **License:** Apache 2.0

---

## Executive Summary

**Operational runbooks — deployment, incident response, backup/recovery, and on-call procedures.**

This document defines the **Minimum Viable Product (MVP)** scope for `zs-docs-runbooks` within the ZarishSphere sovereign digital health platform. It covers what must be built first, acceptance criteria, user stories, and the complete repository file structure.


### Platform Non-Negotiables (apply to every repository)

| Constraint | Rule |
|-----------|------|
| **Zero Cost** | All tooling, hosting, and services must use genuinely free tiers |
| **Open Source** | Apache 2.0 license; all code public |
| **FHIR R5 Native** | All clinical data modelled as FHIR R5 resources |
| **Offline-First** | Must function without network connectivity |
| **No-Coder Friendly** | GUI-first, template-driven, automatable |
| **Documentation as Code** | All decisions in GitHub via RFC/ADR |
| **Multi-tenant** | tenant_id scoping on all data operations |
| **HIPAA/GDPR** | AuditEvent on all PHI access; field-level encryption |

---

## Problem Statement

Without this repository, platform contributors and country program teams lack the governance documentation they need. Decisions become tribal knowledge, standards are inconsistently applied, and the platform cannot scale across countries.

## MVP Goals

1. P0 incident response playbook written
2. Deployment runbook written
3. Database backup and recovery documented

## MVP Functional Requirements

| ID | Requirement | Acceptance Criteria | Priority |
|----|------------|---------------------|---------|
| M-01 | Deployment runbook covers Argo CD sync + rollback | All steps tested manually | P0 |
| M-02 | Incident response P0/P1/P2 classification defined | Escalation matrix present | P0 |
| M-03 | Backup and recovery steps documented | Steps tested against dev environment | P0 |
| M-04 | On-call guide lists all critical services and contacts | All 4 owners listed | P1 |

## MVP Complete Repository Tree

```
zs-docs-runbooks/
├── README.md
├── LICENSE
├── .github/
│   ├── CODEOWNERS
│   └── workflows/
│       └── ci.yml
├── deployment/
│   ├── DEPLOY-RUNBOOK.md
│   └── ROLLBACK-RUNBOOK.md
├── incidents/
│   ├── INCIDENT-RESPONSE.md
│   ├── P0-CRITICAL-PLAYBOOK.md
│   └── POSTMORTEM-TEMPLATE.md
├── backup-recovery/
│   ├── BACKUP-PROCEDURES.md
│   └── RECOVERY-PROCEDURES.md
└── security/
    └── SECURITY-INCIDENT-PLAYBOOK.md
```

---


## Owners & Governance

| Role | GitHub Handle | Responsibility |
|------|--------------|----------------|
| Platform Lead | `@arwa-zarish` | Final approval, RFC votes |
| Technical Lead | `@code-and-brain` | Architecture, Go/TS review |
| DevOps Lead | `@DevOps-Ariful-Islam` | CI/CD, infra, deployment |
| Health Programs | `@BGD-Health-Program` | Clinical content, country programs |

**PR Policy:** All changes via Pull Request. Minimum 1 owner review. CI must pass. No direct commits to `main`.


---

## MVP Acceptance Checklist

- [ ] All MVP files exist in repository with real content (not placeholders)
- [ ] CI pipeline passes on `main` branch
- [ ] No secrets, credentials, or PHI committed
- [ ] README.md reflects current state with setup instructions
- [ ] CODEOWNERS file present
- [ ] All MVP functional requirements verified manually or via automated tests
- [ ] Linked to `CATALOGS.md` and `TODO.md` in `zs-docs-platform`

---

*This document is the authoritative MVP specification for `zs-docs-runbooks`.*
*Changes require a Pull Request with at least 1 owner approval.*
