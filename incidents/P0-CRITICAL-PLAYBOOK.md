# P0 Critical Incident Playbook

> **Runbook:** P0 Critical Incident Response
> **Use when:** System is completely down, PHI breach suspected, or data corruption detected
> **Time target:** Resolved within 4 hours

---

## The First 15 Minutes

### Minute 0–5: Detect and Declare

1. **Alert received** (Grafana, GitGuardian, user report)
2. **Verify it's real** — check Grafana → Is error rate > 50%? Is the FHIR API returning 500?
3. **Declare P0** — message all 4 owners immediately:
   ```
   🚨 P0 INCIDENT DECLARED
   Time: {HH:MM UTC}
   Issue: {one line}
   Impact: {clinics/users affected}
   Commander: @{your name}
   Next update: {HH:MM UTC — 30 min from now}
   ```

### Minute 5–10: Preserve Evidence

**Before making ANY changes:**

```bash
# Take a database snapshot immediately
kubectl exec -n zs-data postgres-0 -- \
  pg_dump -U zarish zarishsphere > /tmp/incident-snapshot-$(date +%Y%m%d-%H%M).sql

# Save current pod state
kubectl get pods --all-namespaces > /tmp/pod-state-$(date +%Y%m%d-%H%M).txt

# Save recent logs (last 1000 lines)
kubectl logs -n zs-core deployment/zs-core-fhir-engine --tail=1000 > /tmp/fhir-engine-logs.txt
```

### Minute 10–15: Contain

**If database breach suspected:**
```bash
# Immediately block external access
kubectl patch ingressroute zs-api-gateway -n zs-gateway \
  --type=json -p '[{"op":"add","path":"/spec/routes/0/middlewares","value":[{"name":"block-all"}]}]'
```

**If deployment caused the issue:**
→ Follow ROLLBACK-RUNBOOK.md immediately

**If infrastructure failure:**
→ Contact `@DevOps-Ariful-Islam` to switch to backup region

---

## Diagnosis Checklist

Work through these in order:

```
□ Is the FHIR API responding?
  curl -s https://api.zarishsphere.com/health | jq .status

□ Is PostgreSQL 18.3 healthy?
  kubectl exec -n zs-data postgres-0 -- pg_isready

□ Is Keycloak 26.5.6 responding?
  curl -s https://auth.zarishsphere.com/health/ready

□ Are NATS 2.12.5 connections established?
  curl -s http://nats:8222/healthz

□ Is Valkey 9.0.3 responding?
  kubectl exec -n zs-data valkey-0 -- valkey-cli ping

□ What changed in the last 2 hours?
  Check GitHub: recent merges to main in affected repos

□ Are pods crashing?
  kubectl get pods --all-namespaces | grep -v Running

□ What do logs show?
  kubectl logs -n zs-core deployment/zs-core-fhir-engine --tail=100
```

---

## Resolution Steps (Based on Root Cause)

| Root Cause | Resolution |
|-----------|-----------|
| Bad deployment | Follow ROLLBACK-RUNBOOK.md |
| Database out of disk | kubectl exec postgres-0 -- vacuum; expand PVC |
| Keycloak crashed | kubectl rollout restart deployment/keycloak -n zs-auth |
| NATS lost JetStream | kubectl rollout restart deployment/nats -n zs-data |
| Valkey OOM | kubectl rollout restart deployment/valkey -n zs-data |
| Secret expired | Rotate via Vault; restart affected deployments |
| PHI breach | See SECURITY-INCIDENT-PLAYBOOK.md |

---

## Resolution Declaration

When resolved:
1. Verify FHIR API responds to smoke test
2. Verify Grafana shows error rate < 0.1%
3. Post resolution message to all owners
4. Open a GitHub Issue for the postmortem
5. Write postmortem within 7 days (use POSTMORTEM-TEMPLATE.md)
