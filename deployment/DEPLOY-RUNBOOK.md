# Deployment Runbook

> **Runbook:** Deploy a New Service Version
> **Applies to:** All ZarishSphere services in Kubernetes (Argo CD managed)
> **Last Updated:** 2026-03-24

---

## Overview

ZarishSphere uses GitOps via Argo CD 3.3.x. In normal operation, deployments are fully automated — merging to `main` triggers Argo CD to deploy. This runbook covers the automated process and how to intervene if needed.

---

## Prerequisites

- GitHub access to the target repository
- Argo CD UI access (ask `@DevOps-Ariful-Islam` for credentials)
- Understanding of what change is being deployed

---

## Standard Deployment (Automated — No Action Required)

```
Developer opens PR → CI passes → Owner approves → Merge to main
    ↓
GitHub Actions builds and pushes Docker image to GHCR
    ↓
Argo CD detects new image tag in values.yaml
    ↓
Argo CD syncs Kubernetes — rolling update (no downtime)
    ↓
Smoke tests run (zs-agent-smoke-tester)
    ↓
Deployment complete ✅
```

**You do not need to do anything for this to happen.**

---

## Manual Sync (If Argo CD Fails to Auto-Sync)

**When to use:** Argo CD shows "OutOfSync" but hasn't auto-synced within 5 minutes.

1. Open Argo CD UI: https://argocd.zarishsphere.com
2. Find the application (e.g., `zs-svc-patient`)
3. Click **"Sync"** button
4. Select **"Prune"** and **"Apply only"** (default settings)
5. Click **"Synchronize"**
6. Watch the sync progress — should complete in < 2 minutes

---

## Force Rollout (If Pod Is Stuck)

**When to use:** New image deployed but pods are stuck in `CrashLoopBackOff`.

```bash
# Force a rollout restart of stuck pods
kubectl rollout restart deployment/{service-name} -n zs-{namespace}

# Watch the rollout
kubectl rollout status deployment/{service-name} -n zs-{namespace}
```

If pods continue crashing: **STOP** → follow ROLLBACK-RUNBOOK.md

---

## Post-Deployment Verification

After any deployment:

1. Check Argo CD shows **"Healthy"** and **"Synced"**
2. Check Grafana dashboard for the service: https://grafana.zarishsphere.com
3. Look for error rate spike (should stay < 0.1%)
4. Check smoke test results in GitHub Actions run
5. Test a real FHIR request:

```bash
curl -s -H "Authorization: Bearer {test-token}" \
  https://api.zarishsphere.com/fhir/R5/metadata \
  | jq .resourceType
# Should return: "CapabilityStatement"
```

---

## Execution Log

| Date | Deployer | Service | Version | Outcome | Notes |
|------|----------|---------|---------|---------|-------|
| 2026-03-24 | @code-and-brain | Bootstrap | 1.0.0 | ✅ Success | Initial deployment |
