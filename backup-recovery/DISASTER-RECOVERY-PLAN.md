# ZarishSphere Disaster Recovery Plan

> **Document:** DR Plan | **Version:** 2.0.0 | **Updated:** 2026-03-24
> **Classification:** Internal — operational staff only
> **Owner:** @DevOps-Ariful-Islam

---

## Recovery Objectives

| Metric | Target | Notes |
|--------|--------|-------|
| **RTO** (Recovery Time Objective) | 4 hours | Time from disaster to service restoration |
| **RPO** (Recovery Point Objective) | 1 hour | Maximum acceptable data loss window |
| **Backup Frequency** | Hourly (PostgreSQL WAL) + Daily (full dump) | |
| **Backup Retention** | 30 days daily, 12 months monthly | |
| **Backup Location** | Cloudflare R2 (encrypted AES-256) | |

---

## Disaster Classification

| Category | Examples | Response Time |
|----------|---------|---------------|
| **P0 — Data Loss** | Accidental DELETE, DB corruption | Immediate |
| **P0 — Complete Outage** | Kubernetes cluster down | Immediate |
| **P1 — Partial Outage** | One service down, degraded performance | < 1 hour |
| **P2 — Data Integrity Issue** | Incorrect data, sync failure | < 4 hours |

---

## Backup Architecture

```
PostgreSQL 18.3 (primary)
    │
    ├── WAL archiving (continuous) → Cloudflare R2: zarishsphere-backups/wal/
    │
    └── pg_dump (daily 02:00 UTC) → Cloudflare R2: zarishsphere-backups/daily/
                                          ↓
                                   AES-256 encryption
                                   (key in Vault)
```

---

## Recovery Scenarios

### Scenario 1: Accidental Data Deletion

**Steps:**
1. Stop writes to affected database immediately
   ```bash
   kubectl scale deployment zs-fhir-engine --replicas=0 -n zs-core
   ```
2. Identify the time of deletion from audit logs
   ```sql
   SELECT * FROM audit.events
   WHERE event_type = 'delete'
   ORDER BY recorded_at DESC LIMIT 100;
   ```
3. Download the last backup before deletion from R2
   ```bash
   rclone copy r2:zarishsphere-backups/daily/[DATE]/ /tmp/restore/
   ```
4. Restore to a test PostgreSQL instance first
   ```bash
   docker run -d --name pg-restore      -e POSTGRES_PASSWORD=restore      timescale/timescaledb:2.25-pg18
   pg_restore -h localhost -U postgres -d zarishsphere /tmp/restore/dump.pgc
   ```
5. Extract and re-import deleted records
6. Verify data integrity
7. Restart services
   ```bash
   kubectl scale deployment zs-fhir-engine --replicas=2 -n zs-core
   ```

### Scenario 2: Kubernetes Cluster Complete Failure

**Steps:**
1. Provision new cluster from IaC
   ```bash
   cd ~/zarishsphere/zs-infra-bgd
   tofu init && tofu apply
   ```
2. Install Argo CD
   ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f      https://raw.githubusercontent.com/argoproj/argo-cd/v3.3.x/manifests/install.yaml
   ```
3. Argo CD syncs all applications from GitOps repo automatically
4. Restore PostgreSQL from R2 backup (see Scenario 1 steps 3–6)
5. Verify all services healthy via smoke tests

### Scenario 3: Cloudflare R2 Backup Corruption

**Steps:**
1. Check all backup files for integrity
   ```bash
   # Each backup has a SHA-256 checksum file
   sha256sum -c zarishsphere-YYYY-MM-DD.pgc.sha256
   ```
2. If current backup corrupted, use previous day's backup
3. Accept RPO = up to 24 hours of data loss
4. Initiate P0 incident and notify owners

---

## Backup Verification Schedule

| Frequency | Test Type | Performed By |
|-----------|-----------|-------------|
| Weekly | Restore to staging and verify row counts | @DevOps-Ariful-Islam |
| Monthly | Full DR simulation (restore to test cluster) | @code-and-brain + @DevOps-Ariful-Islam |
| Quarterly | Table-top DR exercise with all owners | All owners |

---

## Contacts

| Role | Name | Contact |
|------|------|---------|
| Platform Lead | Ariful Islam | zarishsphere@gmail.com |
| Technical Lead | code-and-brain | code.and.brain@gmail.com |
| DevOps Lead | DevOps-Ariful-Islam | dev43ariful@gmail.com |
| Cloud (Cloudflare) | Support | https://support.cloudflare.com |

---

## Post-Recovery Checklist

- [ ] All services returning 200 on health checks
- [ ] FHIR CapabilityStatement endpoint responding
- [ ] AuditEvent logging active (spot-check)
- [ ] Keycloak authentication working (test login)
- [ ] Grafana dashboards showing data
- [ ] Incident report opened and timeline documented
- [ ] Postmortem scheduled within 48 hours
