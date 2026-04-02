# Rollback Runbook

> **Runbook:** Roll Back a Failed Deployment
> **Applies to:** All ZarishSphere services (Kubernetes + Argo CD)
> **Last Updated:** 2026-03-24

---

## When to Roll Back

Roll back immediately if any of these are true:
- Error rate > 1% after deployment (check Grafana)
- P0 incident declared related to the deployment
- Smoke tests failed (check GitHub Actions)
- Clinical data is being corrupted
- Service is unreachable (503 responses)

**Do not wait** — patient safety depends on the system working.

---

## Prerequisites

- Access to Argo CD UI (https://argocd.zarishsphere.com)
- Access to GitHub repository for the service
- Knowledge of the previous working version tag

---

## Rollback Steps

### Step 1 — Identify Previous Version

In GitHub, navigate to the service repository → Tags → Find the last tag before the current one.

Example: If current is `v1.2.0`, previous is `v1.1.3`.

### Step 2 — Revert in Git

```bash
# Option A: Revert the merge commit (preferred)
git revert {merge-commit-hash}
git push origin main
# Argo CD will auto-deploy the reverted version

# Option B: If urgent, update the image tag directly in values.yaml
# Edit: deploy/helm/values.yaml
# Change: image.tag from "v1.2.0" to "v1.1.3"
# Commit and push to main
```

### Step 3 — Watch Argo CD

1. Open Argo CD → Find the service
2. Watch the sync — should complete in < 2 minutes
3. Verify the pods show the old version:
   ```bash
   kubectl get pods -n zs-clinical -o wide | grep {service-name}
   ```

### Step 4 — Verify Service Health

```bash
curl -s https://api.zarishsphere.com/{service}/health
# Should return: {"status": "ok"}
```

Check Grafana — error rate should return to < 0.1%.

### Step 5 — Notify

Post in the platform owner group: "Rollback of {service} v{X} complete. Now running v{Y}."
Open a GitHub Issue to track the root cause investigation.

---

## After Rollback

1. Open GitHub Issue: `[Rollback] {Service} v{X} rolled back`
2. Assign to `@code-and-brain` for root cause analysis
3. Do not re-deploy the failed version until root cause is identified
4. Write a postmortem within 7 days (use POSTMORTEM-TEMPLATE.md)
