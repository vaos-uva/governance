# governance

OZCO UVA governance. OZCO master-registry.

Public registry of the `vaos-uva` GitHub org's public repositories.

## Contents

| File | Purpose |
|---|---|
| `README.md` | This description |
| `vaos-repos-registry.yaml` | Machine-readable registry of the **public** repos (GOV, VAOS, ARCHMODELS) |
| `LICENSE` | Apache License 2.0 |

## Registry

`vaos-repos-registry.yaml` defines every public repo with its repo key (`GOV` / `VAOS` / `ARCHMODELS`),
spec (`[ @VAOS, public, … ]`), remote, role, languages, entrypoints, and validation gates.

- Source of truth: the private `vaos-uva/master-area` repo, `vaos-repos-master-config/build-config.yaml`.
- Scope here: public repos only (`spec[1] = public`). Private `/@UVA` entries of the master registry are deliberately excluded.
- Drift rule: change the master config first, then regenerate this file and commit both repos — never hand-edit this registry (r03 deterministic, r09 templates-not-manual).

## Git workflow

Per repo (any workspace clone): **find → fetch → analyze**. Fetch before analyzing; inspect the actual diff before commit/push. Docs-only repo, validation gate: `git diff --check`.
