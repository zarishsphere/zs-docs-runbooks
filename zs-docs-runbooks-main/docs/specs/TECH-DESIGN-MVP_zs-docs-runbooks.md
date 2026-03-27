# TECH-DESIGN-MVP — `zs-docs-runbooks`

> **Document:** Technical Design (MVP) | **Version:** 1.0.0-mvp
> **Repository:** [https://github.com/zarishsphere/zs-docs-runbooks](https://github.com/zarishsphere/zs-docs-runbooks)
> **Layer:** Layer 0 — Governance | **Catalog #:** 3
> **Language:** Markdown / Docusaurus (future) | **License:** Apache 2.0

---

## Technical Summary

**Operational runbooks — deployment, incident response, backup/recovery, and on-call procedures.**

This document defines the **technical architecture, implementation design, complete repository tree, and acceptance criteria** for the MVP of `zs-docs-runbooks`.

---

## Technical Approach — MVP

All MVP content is **pure Markdown** — no build tooling, no dependencies. Files are browseable directly on GitHub. Docusaurus is a **post-MVP** addition.

## File Format Standards

- All files: UTF-8 encoding, Unix line endings (LF)
- Headings: ATX style (`#`, `##`, `###`)
- Tables: GFM pipe tables
- Code blocks: fenced with language tag
- Links: relative paths within repo; absolute for cross-repo

## CI Pipeline (`.github/workflows/ci.yml`)

```yaml
name: CI
on: [push, pull_request]
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: markdownlint
        run: npx markdownlint-cli "**/*.md" --ignore node_modules
      - name: Check links (MVP only internal)
        run: npx markdown-link-check docs/**/*.md --config .mlc.json
```

## Content Governance

All runbooks are written in plain Markdown, executable without tooling. Each runbook has a Prerequisites section, numbered Steps, and a Verification section.

## Directory Naming Convention

- `docs/` — user-facing documentation
- `templates/` — reusable templates for contributors
- `governance/` — process definitions (RFC, ADR, CAMM)
- `compliance/` — regulatory documentation

## MVP Complete Repository Tree

```
zs-docs-runbooks/
├── README.md
├── LICENSE
├── .github/
│   ├── CODEOWNERS
│   └── workflows/
│       └── ci.yml
├── deployment/
│   ├── DEPLOY-RUNBOOK.md
│   └── ROLLBACK-RUNBOOK.md
├── incidents/
│   ├── INCIDENT-RESPONSE.md
│   ├── P0-CRITICAL-PLAYBOOK.md
│   └── POSTMORTEM-TEMPLATE.md
├── backup-recovery/
│   ├── BACKUP-PROCEDURES.md
│   └── RECOVERY-PROCEDURES.md
└── security/
    └── SECURITY-INCIDENT-PLAYBOOK.md
```

---


## Owners & Governance

| Role | GitHub Handle | Responsibility |
|------|--------------|----------------|
| Platform Lead | `@arwa-zarish` | Final approval, RFC votes |
| Technical Lead | `@code-and-brain` | Architecture, Go/TS review |
| DevOps Lead | `@DevOps-Ariful-Islam` | CI/CD, infra, deployment |
| Health Programs | `@BGD-Health-Program` | Clinical content, country programs |

**PR Policy:** All changes via Pull Request. Minimum 1 owner review. CI must pass. No direct commits to `main`.


---

## MVP Acceptance Checklist

- [ ] All MVP files exist in repository with real content (not placeholders)
- [ ] CI pipeline passes on `main` branch
- [ ] No secrets, credentials, or PHI committed
- [ ] README.md reflects current state with setup instructions
- [ ] CODEOWNERS file present
- [ ] All MVP functional requirements verified manually or via automated tests
- [ ] Linked to `CATALOGS.md` and `TODO.md` in `zs-docs-platform`

---

*This document is the authoritative MVP specification for `zs-docs-runbooks`.*
*Changes require a Pull Request with at least 1 owner approval.*
