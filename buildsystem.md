# vaos-uva build system

Definition of how each repo in the `vaos-uva` org is built, checked, and validated.
Rule set: r03 (deterministic), r09 (templates-not-manual). Git workflow per repo:
**find → fetch → analyze**, analyze before commit/push. Each repo has independent
branch/remote state — never assume a shared branch name.

## Build layers

Layer MASTER (private) owns the registry; public repos build from derived data.

1. **master-area** — master build. Input: `build-config.yaml` (single source of truth).
   Tool: `vaos-repos-master-config/masterbuild-cli.sh`
   (`bash <cli> build | check | list [repo-id]`, runtime python3 + PyYAML).
   `build` renders `repos/{id}.yaml` from config; `check` compares generated files
   against config (`MISS` / `DRIFT` / `ORPHAN`) and exits 1 on any drift. Output is
   deterministic: no timestamps, rebuild produces byte-identical files (r03).
2. **governance** — public registry mirror. `vaos-repos-registry.yaml` is generated
   from the master config, public repos only (`/@VAOS`, never `/@UVA`) — regenerate,
   never hand-edit.
3. **vaos** — portable bash/YAML tooling. Tools in `vaos-portable/vaos-common-tools/tools/`,
   registered in `base-tools.yaml`. Runtime policy: sh BASE, bash only for `<id>-cli.sh`
   entry mode, YAML = master data serialization.
4. **architecture-models-atlas** — pnpm-build TypeScript codebase. pnpm pinned
   `10.4.1`, patched `wouter` — npm/yarn break it. Bootstrapped into the chain here
   for completeness; its own `AGENTS.md` governs.

## Validation gates

Run from repo root, in order.

- `master-area`:   `bash vaos-repos-master-config/masterbuild-cli.sh check` → `git diff --check`
- `governance`:    `git diff --check`
- `vaos`:          `sh vaos-portable/vaos-portable-install.sh validate` →
                   `sh vaos-portable/vaos-common-tools/tools/glagol.sh selftest` →
                   `sh vaos-portable/vaos-common-tools/tools/kstar-tests.sh` → `git diff --check`
- `architecture-models-atlas`:  `pnpm test` → `pnpm run check` (tsc) →
                   `pnpm run build` → `git diff --check`;
                   schema changes additionally `pnpm db:push` (drizzle) with generated SQL review.

## Change flow

Registry/config change lives first in `master-area` (`build-config.yaml`), then
derived outputs are regenerated (`masterbuild-cli.sh build` → commit; gov mirror
regenerated → commit). Commits in every repo only after that repo's gates pass.

## Source of truth

- Repo registry: `master-area/vaos-repos-master-config/build-config.yaml`
- This document: `governance/buildsystem.md` (public definition)
