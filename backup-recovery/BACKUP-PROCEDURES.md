# Backup Procedures

> **Runbook:** Verify and Manage Database Backups
> **Schedule:** Backups run automatically every 6 hours
> **Last Updated:** 2026-03-24

---

## Backup Architecture

```
PostgreSQL 18.3 (primary)
    │
    ├── WAL streaming → PostgreSQL Standby (hot standby, same region)
    │
    └── Daily full backup → Cloudflare R2 bucket (encrypted AES-256)
                                │
                                └── 7-year retention (HIPAA requirement)
```

---

## Verify Backups Are Running

**Check backup job status in Kubernetes:**

```bash
kubectl get cronjob -n zs-data | grep backup
# Output should show: zs-backup-postgres   0/6h   ACTIVE

kubectl get jobs -n zs-data | grep backup | tail -5
# Last 5 backup jobs should show "Complete"
```

**Check R2 bucket has recent files:**

Go to Cloudflare Dashboard → R2 → `zarishsphere-backups-{cc}` → Sort by date.
Most recent file should be < 6 hours old.

---

## Manual Backup (If Needed)

```bash
# Trigger an immediate backup
kubectl create job --from=cronjob/zs-backup-postgres manual-backup-$(date +%Y%m%d%H%M) -n zs-data

# Watch it complete
kubectl logs -n zs-data job/manual-backup-{timestamp} -f
```

---

## Backup Verification (Monthly Test)

**On the first Monday of each month:** `@DevOps-Ariful-Islam` performs a restore test:

1. Download the most recent backup from R2
2. Restore to a fresh PostgreSQL instance (dev environment)
3. Run validation queries:
   ```sql
   SELECT COUNT(*) FROM fhir.resources;
   SELECT MAX(updated_at) FROM fhir.resources;
   SELECT COUNT(DISTINCT tenant_id) FROM fhir.resources;
   ```
4. Record results in the execution log below

---

## Backup Contents

Each backup includes:
- All schemas: `fhir`, `clinical`, `audit`, `terminology`, `analytics`
- All TimescaleDB hypertables
- Sequence states
- Extensions and configuration

Excludes:
- Session data (Keycloak sessions — not needed for recovery)
- Temporary tables
- pg_toast data

---

## Execution Log

| Date | Operator | Type | Duration | Size | Verified | Notes |
|------|----------|------|----------|------|---------|-------|
| 2026-03-24 | @DevOps-Ariful-Islam | Initial | 5 min | 128MB | ✅ | Bootstrap |
