# New Health Facility Onboarding Runbook

> **Runbook:** Add a new health facility to ZarishSphere
> **Applies to:** Any health post, clinic, or hospital being added to an active country deployment
> **Estimated time:** 2–4 hours (browser only for governance steps)

---

## Overview

Adding a new facility to ZarishSphere involves:
1. Registering the facility in the FHIR Location registry
2. Creating a tenant in Keycloak
3. Adding the facility to the country distro
4. Training facility staff
5. Updating CAMM country status

---

## Step 1 — Gather Facility Information

Collect the following before starting:

```
Facility Name (official):
Facility Type: [ ] Health Post  [ ] Sub-district  [ ] District  [ ] Referral
Country Code: BGD / IND / MMR / PAK / THA
Administrative Division: [Division/State/Province]
District:
Upazila/Sub-district:
Village/Ward:
GPS Coordinates (Lat, Long):
Facility Code (official government code, if any):
Manager Name:
Manager Email:
Number of Staff (approx):
Internet connectivity: [ ] 4G  [ ] 3G  [ ] 2G  [ ] Satellite  [ ] None
Device availability: [ ] Android phones  [ ] Tablets  [ ] Desktop computers
```

---

## Step 2 — Register Facility in FHIR Location Registry

**Browser method (GitHub):**

1. Go to https://github.com/zarishsphere/zs-data-facility-registry
2. Click **"Add file"** → **"Create new file"**
3. Name: `facilities/bgd/[FACILITY-CODE].json`
4. Paste and fill in this template:

```json
{
  "resourceType": "Location",
  "id": "bgd-[FACILITY-CODE]",
  "meta": {
    "profile": ["https://zarishsphere.com/fhir/StructureDefinition/ZSLocation"]
  },
  "status": "active",
  "name": "[OFFICIAL FACILITY NAME]",
  "type": [{
    "coding": [{
      "system": "http://terminology.hl7.org/CodeSystem/v3-RoleCode",
      "code": "HOSP",
      "display": "Hospital"
    }]
  }],
  "address": {
    "country": "BD",
    "state": "[DIVISION]",
    "district": "[DISTRICT]",
    "city": "[UPAZILA]",
    "text": "[FULL ADDRESS]"
  },
  "position": {
    "longitude": [LONGITUDE],
    "latitude": [LATITUDE]
  },
  "identifier": [{
    "system": "https://zarishsphere.com/fhir/NamingSystem/bgd-facility-code",
    "value": "[OFFICIAL-GOVT-CODE]"
  }]
}
```

5. Commit with message: `content(facilities): add [FACILITY NAME] BGD`
6. Open PR → owner approves → merges

---

## Step 3 — Create Keycloak Tenant

**Via Keycloak Admin UI:**

1. Open https://auth.zarishsphere.com (or localhost:8443 for local)
2. Log in as admin
3. Go to **Clients** → **Create client**
4. Client ID: `fac-[COUNTRY]-[FACILITY-CODE]`
5. Set Home URL and Valid redirect URIs
6. Go to **Users** → create user accounts for facility manager
7. Assign role: `facility-admin` for the manager

**Via Keycloak CLI (if needed):**
```bash
kcadm.sh create clients -r zarishsphere   -s clientId="fac-bgd-[CODE]"   -s enabled=true   -s publicClient=false
```

---

## Step 4 — Update Country Distro

1. Go to https://github.com/zarishsphere/zs-distro-bgd
2. Open `facilities/facility-registry.json`
3. Click ✏️ edit
4. Add the new facility to the array
5. Commit and open PR

---

## Step 5 — Staff Training

| Training Module | Audience | Duration | Method |
|----------------|---------|----------|--------|
| ZarishSphere Overview | All staff | 30 min | Video / in-person |
| Patient Registration | Registration clerks | 2 hours | Hands-on practice |
| Clinical Data Entry | Nurses, clinicians | 4 hours | Guided session |
| Mobile App (if CHW) | Community health workers | 3 hours | Hands-on |
| System Admin | Facility manager | 2 hours | Walkthrough |

Training materials: https://docs.zarishsphere.com/training

---

## Step 6 — Update CAMM Status

1. Go to https://github.com/zarishsphere/zs-docs-camm
2. Open `countries/BGD-STATUS.md`
3. Update facility count and date
4. Open PR for owner approval

---

## Step 7 — Verify Onboarding Complete

```
[ ] Facility appears in FHIR Location registry
[ ] Facility users can log in to ZarishSphere
[ ] Test patient successfully registered
[ ] Test encounter successfully created
[ ] AuditEvent generated for test encounter
[ ] Facility visible in Grafana monitoring
[ ] Staff confirmed trained
[ ] CAMM country status updated
```

---

*Onboarding issues? Open an issue in zs-docs-runbooks or email platform@zarishsphere.com*
