# felixgeelhaar/.github

Central, reusable GitHub Actions workflows and org defaults for the
`felixgeelhaar` OSS repositories. Bump a pinned tool/action version **here
once** and every caller repo picks it up.

## What lives here

| Reusable workflow | Purpose |
| --- | --- |
| [`js-ci.yml`](.github/workflows/js-ci.yml) | JS/TS CI — lint / typecheck / test / build (auto-detected) + a **nox** security scan with self-adjusting baseline gate. |
| [`nox-remediate.yml`](.github/workflows/nox-remediate.yml) | **Replaces dependabot.** Weekly + on-demand OSV dependency upgrades and GitHub Actions pin bumps, opened as one auto-merged PR. |

Caller templates: [`nox-remediate-caller-example.yml`](.github/workflows/nox-remediate-caller-example.yml),
[`js-ci-caller-example.yml`](.github/workflows/js-ci-caller-example.yml).

> A `go-ci.yml` reusable will be added when the rollout reaches the Go repos.

## The model

- **warden** — local git commit/push provenance gate (per repo, replaces
  husky + lint-staged). `warden init`; policy in `.warden.yaml`.
- **nox** — the single source of truth for dependency CVEs, secrets, and IaC
  findings. Runs in CI (`js-ci.yml` security job) and remediates on a schedule
  (`nox-remediate.yml`). Findings gate net-new critical/high once a repo
  commits a `.nox/baseline.json`.
- **nox-remediate** — the dependabot replacement. SHA-pins actions + bumps
  vulnerable deps, one PR, auto-merged on green.

## Adopting in a repo

1. **Dependency + action remediation** (dependabot replacement): copy
   `nox-remediate-caller-example.yml` to `.github/workflows/nox-remediate.yml`
   and delete `.github/dependabot.yml` + any dependabot auto-merge workflow.
2. **CI**: repos without bespoke CI copy `js-ci-caller-example.yml` to
   `.github/workflows/ci.yml`. Repos with richer required checks (E2E, CodeQL,
   cross-platform) keep their own `ci.yml` and just make their nox scan step
   blocking with the baseline gate.
3. **Local gate**: `warden init` and commit `.warden.yaml`.

## Pinned tool versions

- **nox** `1.7.1` (`sha256 3106ab3ce7197f43a45fc1128ea27112686421e8ec3eac9be356d4ca42ab4c89`)

Bump in the reusable workflow inputs; verify the sha256 of
`nox_<version>_linux_amd64.tar.gz` on every bump.
