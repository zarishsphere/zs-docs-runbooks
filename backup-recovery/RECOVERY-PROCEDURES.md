# Database Recovery Procedures

> **Runbook:** Restore ZarishSphere from Backup
> **Use when:** Database corruption, data loss, or disaster recovery required
> **Warning:** This procedure will cause downtime. Notify all active users first.
> **Last Updated:** 2026-03-24

---

## Before You Start

**Do NOT run this procedure without:**
- [ ] Confirmation from `@arwa-zarish` that recovery is needed
- [ ] List of all active users notified (send message to facility coordinators)
- [ ] Estimated downtime communicated (plan for 2–4 hours)
- [ ] PostgreSQL WAL replay considered first (see Step 0)

---

## Step 0: Try WAL Replay First (Less Downtime)

If the database is still running and only the last few hours are affected:

```bash
# Check WAL standby lag
kubectl exec -n zs-data postgres-standby-0 -- \
  psql -U zarish -c "SELECT now() - pg_last_xact_replay_timestamp() AS replication_delay;"

# If delay < 6 hours: promote standby instead of restoring from backup
kubectl exec -n zs-data postgres-standby-0 -- \
  pg_ctl promote -D /var/lib/postgresql/data
```

If WAL replay works: skip Steps 1–5, go directly to Step 6.

---

## Step 1: Take Services Offline

```bash
# Scale down all services that write to PostgreSQL
kubectl scale deployment -n zs-core --replicas=0 --all
kubectl scale deployment -n zs-clinical --replicas=0 --all

# Verify all pods are down
kubectl get pods -n zs-core
kubectl get pods -n zs-clinical
```

---

## Step 2: Download Backup from R2

```bash
# List available backups
rclone ls r2:zarishsphere-backups-{cc}/ | sort -k2 | tail -20

# Download the most recent full backup
rclone copy r2:zarishsphere-backups-{cc}/{backup-filename}.sql.enc /tmp/recovery/

# Decrypt the backup
vault kv get -field=key secret/zarishsphere/prod/backup-encryption-key > /tmp/backup.key
openssl enc -d -aes-256-gcm -in /tmp/recovery/{backup}.sql.enc \
  -out /tmp/recovery/{backup}.sql \
  -pass file:/tmp/backup.key
```

---

## Step 3: Drop and Recreate Database

```bash
kubectl exec -n zs-data postgres-0 -- \
  psql -U zarish postgres -c "
    SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname = 'zarishsphere';
    DROP DATABASE IF EXISTS zarishsphere;
    CREATE DATABASE zarishsphere OWNER zarish;
  "
```

---

## Step 4: Restore

```bash
kubectl cp /tmp/recovery/{backup}.sql zs-data/postgres-0:/tmp/restore.sql

kubectl exec -n zs-data postgres-0 -- \
  psql -U zarish zarishsphere < /tmp/restore.sql

# Verify row counts
kubectl exec -n zs-data postgres-0 -- \
  psql -U zarish zarishsphere -c "SELECT COUNT(*) FROM fhir.resources;"
```

---

## Step 5: Run Migrations

```bash
# Apply any migrations that postdate the backup
kubectl run migrate-job --image=ghcr.io/zarishsphere/zs-core-fhir-engine:latest \
  --env="DATABASE_URL=postgres://zarish:{password}@postgres:5432/zarishsphere" \
  --command -- migrate -path /migrations -database ${DATABASE_URL} up
```

---

## Step 6: Bring Services Back Online

```bash
kubectl scale deployment -n zs-core --replicas=2 --all
kubectl scale deployment -n zs-clinical --replicas=2 --all

# Run smoke tests
curl -s https://api.zarishsphere.com/fhir/R5/metadata | jq .resourceType
# Should return: "CapabilityStatement"
```

---

## Step 7: Communicate Resolution

1. Notify all facility coordinators: "ZarishSphere is restored and operational"
2. Specify any data loss window (time between last backup and incident)
3. Open a GitHub Issue for the postmortem
