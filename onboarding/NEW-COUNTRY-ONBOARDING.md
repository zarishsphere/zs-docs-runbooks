# New Country Onboarding Runbook

> **Runbook:** Onboard a New Country to ZarishSphere
> **Applies to:** CAMM L0 to L1 transition
> **Time required:** 2–4 hours of operator time (spread over 1–2 weeks)
> **Last Updated:** 2026-03-24

---

## Prerequisites

- [ ] MOU signed (see `zs-docs-camm/templates/MOU-TEMPLATE.md`)
- [ ] Country focal point has a GitHub account
- [ ] Country 2-letter code determined (ISO 3166-1 alpha-2): e.g., BGD, IND, MMR
- [ ] Target cloud region or hardware determined
- [ ] `@arwa-zarish` approval to proceed

---

## Step 1: Create GitHub Repositories (15 minutes)

All steps done in GitHub browser:

1. Go to https://github.com/orgs/zarishsphere/repositories
2. Click **"New repository"**
3. Create `zs-distro-{cc}`:
   - Description: "ZarishSphere distribution: {country name}"
   - Public, Apache 2.0
4. Create `zs-infra-{cc}`:
   - Description: "ZarishSphere country infrastructure: {country name}"
   - Public, Apache 2.0

---

## Step 2: Fork Country Distro Template (10 minutes)

In `zs-distro-{cc}`:

1. Copy all files from `zs-distro-core` into `zs-distro-{cc}`
2. Edit `distro.yaml`:
   ```yaml
   metadata:
     name: zs-distro-{cc}
     country: {Country Name}
     countryCode: {CC}
     language: {language code}
     version: 1.0.0
   ```
3. Add country focal point to CODEOWNERS

---

## Step 3: Add Country Focal Point to GitHub Org (5 minutes)

1. Go to https://github.com/orgs/zarishsphere/people
2. Click **"Invite member"**
3. Enter their GitHub username
4. Role: Member (not Owner)
5. Add them to team: `country-{cc}-team` (create team if needed)

---

## Step 4: Add Country Status File (10 minutes)

In `zs-docs-camm`, edit `countries/{CC}-STATUS.md`:
- Update current level to L0
- Add focal point details
- Add MOU date

---

## Step 5: Set Up Country Infrastructure (1–2 hours)

Depending on deployment option:

**Option A: Oracle Cloud Free Tier (recommended)**
1. Sign up at cloud.oracle.com with country focal point email
2. Provision 2x ARM Ampere A1 Compute instances (free forever):
   - 2 OCPUs, 12GB RAM each
3. Install Docker 29.x on each
4. Document instance IPs in `zs-infra-{cc}/README.md`

**Option B: Raspberry Pi 5**
1. Flash Ubuntu Server 24.04 ARM64 to SD card
2. Set static IP on facility network
3. Install Docker 29.x

**Option C: Render.com (free tier, quick start)**
1. Create Render account
2. Deploy `zs-iac-dev-environment` Docker Compose to Render
3. Note: 750 hrs/month free — suitable for pilot only

---

## Step 6: Deploy ZarishSphere Stack (30 minutes)

SSH to the server, then:

```bash
git clone https://github.com/zarishsphere/zs-iac-dev-environment
cd zs-iac-dev-environment

# Edit .env with country-specific values
cp .env.example .env
nano .env

# Start the stack
docker compose up -d

# Verify all services running
docker compose ps
```

---

## Step 7: Configure Keycloak (30 minutes)

1. Open https://{server-ip}:8443
2. Login: admin / zarish_admin (change immediately!)
3. Create realm: `zarishsphere-{cc}`
4. Create clients: `zs-fhir-engine`, `zs-ui-clinical`
5. Create roles: `clinician`, `nurse`, `chw`, `supervisor`, `admin`
6. Create initial admin user for country focal point

---

## Step 8: Load Country Data (30 minutes)

```bash
# Load country facility registry
cd zs-distro-{cc}
./scripts/load-facilities.sh

# Load country forms
./scripts/load-forms.sh

# Load country terminology mappings
./scripts/load-terminology.sh
```

---

## Step 9: Verify Deployment (15 minutes)

```bash
# Test FHIR API
curl -s http://{server-ip}:8000/fhir/R5/metadata | jq .resourceType
# Expected: "CapabilityStatement"

# Test authentication
curl -s http://{server-ip}:8443/realms/zarishsphere-{cc}/.well-known/openid-configuration \
  | jq .issuer
```

---

## Step 10: Train Country Focal Point (2 hours)

Follow `zs-docs-camm/templates/TRAINING-CURRICULUM.md` L1 track.

---

## Post-Onboarding Checklist

- [ ] GitHub repositories created: `zs-distro-{cc}`, `zs-infra-{cc}`
- [ ] Country focal point added to GitHub org
- [ ] ZarishSphere stack deployed and responding
- [ ] Keycloak configured with country realm
- [ ] Country data loaded (facilities, forms, terminology)
- [ ] Country focal point trained (L1 curriculum complete)
- [ ] CAMM STATUS.md updated to L0 (will advance to L1 after first patient)
- [ ] MOU signed and filed in `zs-docs-donor/grants/`
