# Grafana Dashboard Reference

> **Runbook:** Monitoring and Alerting via Grafana 12.x
> **URL:** https://grafana.zarishsphere.com (or http://localhost:3000 locally)
> **Credentials:** admin / zarish_grafana (local dev)

---

## Dashboard Inventory

| Dashboard | URL Path | Purpose | Audience |
|-----------|---------|---------|----------|
| Platform Overview | /d/platform-overview | All services health at a glance | All owners |
| FHIR Engine | /d/fhir-engine | Request rates, latencies, error rates | Technical |
| Clinical Services | /d/clinical-services | Patient, encounter, observation services | Clinical leads |
| Database | /d/postgresql | PostgreSQL 18.3 query performance | DevOps |
| NATS JetStream | /d/nats | Message throughput, consumer lag | DevOps |
| Valkey | /d/valkey | Cache hit rate, memory usage | DevOps |
| Keycloak | /d/keycloak | Authentication rates, failed logins | Security |
| Kubernetes | /d/kubernetes | Pod health, resource usage | DevOps |
| Country Health BGD | /d/country-bgd | Bangladesh program indicators | Program managers |

---

## Key Metrics to Watch Daily

### Service Health (Platform Overview dashboard)
- All service panels should be **green**
- Any **red** panel = investigate immediately
- **Yellow** = degraded, plan intervention

### FHIR Engine Performance
| Metric | Healthy | Warning | Critical |
|--------|---------|---------|----------|
| P95 response time | < 200ms | 200–500ms | > 500ms |
| Error rate | < 0.1% | 0.1–1% | > 1% |
| Request rate | any | — | 0 (no traffic) |

### Database (PostgreSQL 18.3)
| Metric | Healthy | Warning | Critical |
|--------|---------|---------|----------|
| Active connections | < 80 | 80–100 | > 100 |
| Replication lag | < 100ms | 100ms–1s | > 1s |
| Disk usage | < 70% | 70–85% | > 85% |

---

## Alerts Setup

Alerts are configured in `zs-iac-observability`. Key alerts:

| Alert Name | Condition | Action |
|-----------|-----------|--------|
| FHIREngineDown | FHIR engine returns non-200 for 2 min | Page @DevOps-Ariful-Islam |
| DatabaseConnectionPoolExhausted | connections > 95 | Page @DevOps-Ariful-Islam |
| DiskUsageCritical | disk > 85% | Email all owners |
| NATSConsumerLag | consumer lag > 10,000 | Email @code-and-brain |
| HighErrorRate | error rate > 1% for 5 min | Email all owners |

---

## Adding a New Dashboard

1. Design in Grafana UI
2. Export as JSON (Dashboard → Share → Export)
3. Save to `zs-iac-observability/dashboards/[name].json`
4. Open PR

---

## Grafana Provisioning

Dashboards are provisioned from `zs-iac-observability` via GitOps.
Do not save dashboards only in Grafana UI — they will be lost on pod restart.
Always commit to the `zs-iac-observability` repository.
