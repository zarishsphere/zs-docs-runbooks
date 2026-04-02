# Incident Response Guide

> **Runbook:** Incident Response Overview
> **Last Updated:** 2026-03-24

---

## Incident Severity Levels

| Level | Name | Description | Response Time | Examples |
|-------|------|-------------|---------------|---------|
| P0 | Critical | System completely down or PHI breach | Immediate — < 1 hour | Database down, security breach, complete outage |
| P1 | High | Major feature broken, some users affected | < 4 hours | FHIR API returning 500, auth failing for some users |
| P2 | Medium | Minor feature broken, workaround exists | < 24 hours | Slow search, PDF generation failing |
| P3 | Low | Cosmetic issue, no functional impact | < 7 days | UI typo, wrong timezone in report |

---

## First Steps for Any Incident

1. **Do not panic.** Read this runbook completely before acting.
2. **Determine severity** using the table above.
3. **Check if it's already known.** Look at GitHub Issues and Grafana alerts.
4. **Declare the incident** in the platform owner group (WhatsApp/Signal) with:
   ```
   INCIDENT DECLARED
   Severity: P{N}
   Description: {one line description}
   Impact: {who is affected}
   Declared by: @{your-name}
   ```

---

## Escalation Matrix

| Incident | Who to Contact | How |
|----------|---------------|-----|
| P0 — All | `@arwa-zarish` immediately | Call/WhatsApp |
| P0 — Technical | `@code-and-brain` immediately | Call/WhatsApp |
| P0 — Infrastructure | `@DevOps-Ariful-Islam` immediately | Call/WhatsApp |
| P0 — PHI breach | All 4 owners + legal counsel | Call |
| P1 — Technical | `@code-and-brain` within 2h | Message |
| P1 — Infrastructure | `@DevOps-Ariful-Islam` within 2h | Message |
| P2 | Assign GitHub Issue | GitHub |
| P3 | File GitHub Issue | GitHub |

---

## Communication During Incident

| Action | Who | How Often |
|--------|-----|-----------|
| Status update | Incident commander | Every 30 min (P0), every 2h (P1) |
| Country program notification | `@arwa-zarish` or `@BGD-Health-Program` | On P0/P1 affecting their data |
| Resolution notification | Incident commander | Once resolved |
| Postmortem | All owners | Within 7 days of resolution |

---

## For P0 Incidents: Go to P0-CRITICAL-PLAYBOOK.md

For all critical incidents, follow the dedicated P0 playbook.
