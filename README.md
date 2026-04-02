# zs-docs-runbooks — Operational Runbooks

> **Repository:** https://github.com/zarishsphere/zs-docs-runbooks
> **Layer:** Governance (Layer 0) | **Catalog #:** 3
> **License:** Apache 2.0 | **Status:** Active

---

## What Is This?

Step-by-step operational procedures for ZarishSphere. Every critical operation has a runbook so that any trained operator — regardless of prior experience with a specific system — can execute it correctly.

**Rule:** If you do something more than once, there must be a runbook for it.

---

## Runbook Index

| Runbook | When to Use |
|---------|------------|
| [deployment/DEPLOY-RUNBOOK.md](./deployment/DEPLOY-RUNBOOK.md) | Deploying a new service version |
| [deployment/ROLLBACK-RUNBOOK.md](./deployment/ROLLBACK-RUNBOOK.md) | Rolling back a failed deployment |
| [incidents/INCIDENT-RESPONSE.md](./incidents/INCIDENT-RESPONSE.md) | Responding to any incident |
| [incidents/P0-CRITICAL-PLAYBOOK.md](./incidents/P0-CRITICAL-PLAYBOOK.md) | Critical incident (outage, breach) |
| [incidents/POSTMORTEM-TEMPLATE.md](./incidents/POSTMORTEM-TEMPLATE.md) | Writing an incident postmortem |
| [backup-recovery/BACKUP-PROCEDURES.md](./backup-recovery/BACKUP-PROCEDURES.md) | Verifying backups are running |
| [backup-recovery/RECOVERY-PROCEDURES.md](./backup-recovery/RECOVERY-PROCEDURES.md) | Restoring from backup |
| [onboarding/NEW-COUNTRY-ONBOARDING.md](./onboarding/NEW-COUNTRY-ONBOARDING.md) | Onboarding a new country |
| [security/SECURITY-INCIDENT-PLAYBOOK.md](./security/SECURITY-INCIDENT-PLAYBOOK.md) | Responding to security incidents |

---

## How to Use Runbooks

1. Find the appropriate runbook from the index above
2. Read the **Prerequisites** section first
3. Follow steps **in order** — do not skip steps
4. After completing, add a log entry at the bottom of the runbook (date, what you did, outcome)
5. If a step fails: **STOP**, escalate per the escalation matrix in each runbook

---

## Owners

| Role | GitHub Handle | On-Call Responsibility |
|------|--------------|----------------------|
| Platform Lead | `@arwa-zarish` | P0 decisions |
| Technical Lead | `@code-and-brain` | Technical P0/P1 |
| DevOps Lead | `@DevOps-Ariful-Islam` | Infrastructure P0/P1/P2 |
