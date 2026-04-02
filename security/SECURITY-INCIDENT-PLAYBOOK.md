# Security Incident Playbook

> **Runbook:** Responding to Security Incidents
> **Use when:** Suspected PHI breach, credential compromise, or active attack
> **Last Updated:** 2026-03-24

---

## Security Incident Triggers

Declare a security incident immediately if ANY of these occur:
- GitGuardian detected a committed secret
- Unusual access patterns in `audit.events` (access by unknown IP/user)
- Keycloak logs show mass login failures (> 50 failures in 5 min)
- Database accessed outside normal hours by an admin account
- Staff reports patient data shown to wrong person
- Trivy found a CRITICAL CVE in a deployed container

---

## First 30 Minutes

### Minute 0–5: Contain the Threat

```bash
# If credential is compromised — revoke immediately
# GitHub token: github.com/settings/tokens → Revoke
# Cloudflare token: dash.cloudflare.com → API Tokens → Revoke
# PostgreSQL: vault lease revoke {lease_id}
# Keycloak: Admin UI → Sessions → Revoke all sessions for affected user

# If active intrusion — block the source
# Cloudflare: Security → Firewall → Create rule: Block IP {suspicious_ip}
# Or: kubectl apply -f - <<EOF
# apiVersion: cilium.io/v2
# kind: CiliumNetworkPolicy
# spec:
#   egressDeny: [{ toCIDR: {suspicious_ip}/32 }]
# EOF
```

### Minute 5–15: Preserve Evidence

**Before making further changes**, capture:

```bash
# Keycloak access logs
kubectl logs -n zs-auth deployment/keycloak --tail=1000 > /tmp/keycloak-logs.txt

# FHIR audit events for last 2 hours
kubectl exec -n zs-data postgres-0 -- psql -U zarish zarishsphere -c \
  "SELECT * FROM audit.events WHERE recorded_at > NOW() - INTERVAL '2 hours' ORDER BY recorded_at;" \
  > /tmp/audit-events.txt

# Capture suspicious network traffic if possible
kubectl exec -n zs-core deployment/zs-core-fhir-engine -- ss -tulnp > /tmp/connections.txt
```

### Minute 15–30: Assess and Notify

**Determine scope:**
- Which tenant(s) are affected?
- Which resource types were accessed?
- Was data read only, or was data modified/deleted?
- Is the attacker still active?

**If PHI was accessed without authorisation:**
- Notify `@arwa-zarish` IMMEDIATELY (call, not message)
- GDPR notification required within 72 hours (Art. 33)
- Do not notify users until scope is determined

---

## Specific Scenarios

### Committed Secret

1. Revoke the secret immediately (GitHub token → revoke; API key → invalidate)
2. Generate a new secret and update in Vault/GitHub Org Secrets
3. Force-push git history rewrite to remove the secret from all branches:
   ```bash
   git filter-repo --sensitive-data-removal --path {file-containing-secret}
   ```
4. Notify all owners
5. Document in security incident log

### Mass Failed Authentication

1. Check Keycloak brute force protection is active (Settings → Security Defenses)
2. Block the attacking IP in Cloudflare firewall
3. Review if any accounts were successfully compromised
4. If any account compromised → follow "Credential Compromise" procedure

### Unexpected Data Export

1. Review `audit.events` for export events: `SELECT * FROM audit.events WHERE event_type = 'export'`
2. Identify user who initiated export
3. Suspend user account in Keycloak
4. Contact user's organization for explanation
5. If unauthorised: notify affected patients per GDPR Art. 34

---

## Notification Templates

**To country focal point (P0 breach):**
> "ZarishSphere Security Notice: We have detected a potential security incident affecting {country} data. We have taken immediate containment measures. We will provide a full report within 24 hours. Please change your ZarishSphere password immediately at {url}."

**To regulatory authority (GDPR Art. 33, within 72h):**
> Use the supervisory authority's official notification form. Include: nature of breach, categories of data subjects, approximate number affected, likely consequences, measures taken.
