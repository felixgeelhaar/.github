# felixgeelhaar/.github

Central, reusable GitHub Actions workflows and org defaults for the
`felixgeelhaar` OSS repositories. Bump a pinned tool/action version **here
once** and every caller repo picks it up.

## What lives here

| Reusable workflow | Purpose |
| --- | --- |
| [`js-ci.yml`](.github/workflows/js-ci.yml) | JS/TS CI — lint / typecheck / test / build (auto-detected) + a **nox** security scan with self-adjusting baseline gate. |
| [`go-ci.yml`](.github/workflows/go-ci.yml) | Go CI — gofmt / golangci-lint / race tests / optional coverage gate + a **nox** scan with taint SAST (cosign-verified plugin). |
| [`nox-remediate.yml`](.github/workflows/nox-remediate.yml) | **Replaces dependabot.** Weekly + on-demand OSV dependency upgrades and GitHub Actions pin bumps, opened as one auto-merged PR. |

Caller templates: [`nox-remediate-caller-example.yml`](.github/workflows/nox-remediate-caller-example.yml),
[`js-ci-caller-example.yml`](.github/workflows/js-ci-caller-example.yml).

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
   cross-platform) keep their own `ci.yml` but run no nox step of their own:
   add the shared scan as a separate workflow, copied from
   klarlabs-studio/.github's `workflow-templates/nox.yml`.
3. **Local gate**: `warden init` and commit `.warden.yaml`.

## Pinned tool versions

- **nox**: the scan in `go-ci` and `js-ci` runs
  [klarlabs-studio/.github's `nox.yml`](https://github.com/klarlabs-studio/.github/blob/main/.github/workflows/nox.yml),
  whose pin that repo's `scripts/nox-pin.sh` keeps current for the whole fleet.
  Never pin nox in a caller. `nox-remediate.yml` here still carries its own pin
  (1.63.0); bump it with the same signed checksum.
