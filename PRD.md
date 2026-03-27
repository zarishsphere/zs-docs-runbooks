# PRD — `zs-docs-runbooks`

> **Document Class:** PRD | **Version:** 1.0.0 | **Status:** Bootstrapping
> **Repository:** [https://github.com/zarishsphere/zs-docs-runbooks](https://github.com/zarishsphere/zs-docs-runbooks)
> **Layer:** Layer 0 — Governance | **Catalog #:** 3
> **License:** Apache 2.0 | **Governance:** RFC-0001

---

## 1. Overview

Operational runbooks repository. Contains step-by-step procedures for deployments, incident response, on-call rotations, backup/recovery, and routine operational tasks.

---

## 2. Repository Metadata

- **Name:** `zs-docs-runbooks`
- **Organization:** [https://github.com/zarishsphere](https://github.com/zarishsphere)
- **Language / Runtime:** Markdown
- **Visibility:** Public
- **License:** Apache 2.0
- **Default Branch:** `main`
- **Branch Protection:** Required (2-owner review for critical paths)

---

## 3. Platform Context

This repository is part of the **ZarishSphere** sovereign digital health operating platform — a free, open-source, FHIR R5-native system for South and Southeast Asia.

**Non-negotiable constraints:**
- Zero cost — all tooling must use genuinely free tiers
- FHIR R5 native — all clinical data modelled as FHIR R5 resources
- Offline-first — must work without network connectivity
- No-coder friendly — GUI-first, template-driven
- Documentation as Code — all decisions in GitHub

---

## 4. Goals & Objectives

- Provide actionable runbooks for all operational scenarios
- Ensure any owner can execute critical operations without tribal knowledge
- Document disaster recovery procedures for all critical systems

## 5. Functional Requirements

| ID | Requirement | Priority |
|----|------------|---------|
| F-01 | Deployment runbook (Argo CD sync, rollback procedures) | P0 |
| F-02 | Incident response playbook (P0–P3 severity tiers) | P0 |
| F-03 | Database backup and recovery procedures | P0 |
| F-04 | On-call rotation guide | P1 |
| F-05 | New country onboarding operational checklist | P1 |
| F-06 | Security incident response playbook | P0 |
| F-07 | Keycloak realm management procedures | P2 |
| F-08 | DNS and Cloudflare configuration guide | P2 |

## 6. Repository Tree

```
zs-docs-runbooks/
├── README.md
├── LICENSE
├── .github/
│   ├── CODEOWNERS
│   └── workflows/
│       └── ci.yml
├── deployment/
│   ├── DEPLOY-RUNBOOK.md              # Standard deployment procedure
│   ├── ROLLBACK-RUNBOOK.md            # How to rollback a failed deploy
│   ├── ARGOCD-OPERATIONS.md           # Argo CD sync, diff, app management
│   ├── HELM-UPGRADE-GUIDE.md          # Helm chart upgrade procedures
│   └── ZERO-DOWNTIME-DEPLOY.md        # Rolling update strategy
├── incidents/
│   ├── INCIDENT-RESPONSE.md           # P0–P3 response procedures
│   ├── P0-CRITICAL-PLAYBOOK.md        # Critical incident (data breach, outage)
│   ├── P1-HIGH-PLAYBOOK.md            # High severity playbook
│   ├── POSTMORTEM-TEMPLATE.md         # Standard postmortem format
│   └── ESCALATION-MATRIX.md           # Who to contact for what
├── backup-recovery/
│   ├── BACKUP-PROCEDURES.md           # PostgreSQL backup to Cloudflare R2
│   ├── RECOVERY-PROCEDURES.md         # Restore from backup steps
│   ├── DISASTER-RECOVERY-PLAN.md      # Full DR plan and RTO/RPO targets
│   └── BACKUP-VERIFICATION.md         # How to test backups monthly
├── security/
│   ├── SECURITY-INCIDENT-PLAYBOOK.md  # Security breach response
│   ├── SECRET-ROTATION.md             # Vault secret rotation procedures
│   ├── KEYCLOAK-ADMIN-GUIDE.md        # Realm, client, user management
│   └── CERTIFICATE-RENEWAL.md         # TLS cert renewal procedures
├── infrastructure/
│   ├── DNS-MANAGEMENT.md              # Cloudflare DNS change procedures
│   ├── KUBERNETES-OPS.md              # K8s node management, scaling
│   ├── SCALING-RUNBOOK.md             # How to scale services up/down
│   └── COST-MONITORING.md             # Free tier usage monitoring
├── onboarding/
│   ├── NEW-COUNTRY-ONBOARDING.md      # Full country onboarding checklist
│   ├── NEW-FACILITY-ONBOARDING.md     # Site-level onboarding steps
│   └── OWNER-OFFBOARDING.md           # Removing an org owner safely
└── monitoring/
    ├── GRAFANA-DASHBOARDS.md           # Dashboard links and interpretation
    ├── ALERTS-GUIDE.md                 # Alert meanings and responses
    └── SYNTHETIC-MONITORING.md         # Uptime check management
```

## 9. Ownership & Governance

| Role | GitHub User |
|------|-------------|
| Platform Lead | `@arwa-zarish` |
| Technical Lead | `@code-and-brain` |
| DevOps Lead | `@DevOps-Ariful-Islam` |
| Health Programs | `@BGD-Health-Program` |

All changes go through Pull Request → 1+ owner review → CI pass → merge.
Breaking changes require RFC + ADR.

---

## 10. Definition of Done

- [ ] All listed files exist with content
- [ ] CI pipeline passes (test + lint + security)
- [ ] README.md reflects current state
- [ ] OpenAPI / AsyncAPI spec present (services only)
- [ ] At least 1 integration test using testcontainers-go (Go) or Playwright (UI)
- [ ] No secrets committed (GitGuardian verified)
- [ ] CODEOWNERS file present
- [ ] Linked to CATALOGS.md and TODO.md

---

*This PRD is the canonical source of truth for this repository's purpose, structure, and requirements.*
*Changes require a PR against this file with owner approval.*
