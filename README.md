# renovate-config

Organization-wide [Renovate](https://docs.renovatebot.com/) configuration preset for all `metio`
repositories. It exists so dependency-update policy lives in **one place** instead of being copied into
every repo's `renovate.json`.

## Using the preset

In any repository's `renovate.json`, extend this preset:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>metio/renovate-config"]
}
```

`github>` references this preset on GitHub explicitly — the metio repositories all live there, and the
explicit host reads clearly. The reference points at [`default.json`](default.json) on this repo's
default branch — that is the file Renovate reads. (`local>metio/renovate-config` is an equivalent,
platform-agnostic alternative that resolves through whatever platform Renovate is running on.)

## What the preset does

[`default.json`](default.json):

- **`config:recommended`** — Renovate's recommended baseline (dependency dashboard, sensible
  scheduling, auto-detected managers: Maven, GitHub Actions, Dockerfile/Containerfile, npm, …).
- **`automerge: true`** with **`major.automerge: true`** — every update, including major-version bumps,
  is auto-merged once required status checks pass. Auto-merge uses the platform's native mechanism
  (`platformAutomerge`), so a repo's branch protection still gates the merge.
- **`rebaseWhen: "behind-base-branch"`** — Renovate rebases any PR that falls
  behind its base branch on the next run, so when one PR merges its siblings are
  brought up to date automatically. This is what keeps auto-merge hands-off under
  a "require branches to be up to date before merging" branch-protection rule —
  without it, the second of two PRs touching the same file gets stuck
  "out-of-date" and never merges.
- **`assignees: ["sebhoss"]`** — update PRs are assigned to the maintainer.
- **`labels: ["dependencies"]`** — every update PR is labelled, so they can be
  filtered/automated uniformly. Note: Renovate does **not** create labels — the
  `dependencies` label must exist in each repo (and `security`, below). Set them
  once via the GitHub org's *Repository defaults → Labels* (applies to new repos)
  or sync them across existing repos with a label-management action.
- **`:semanticCommits`** — Conventional-Commits-style commit messages
  (`chore(deps): …`), uniform across the org.
- **`helpers:pinGitHubActionDigests`** — pins every `uses:` action to its commit
  SHA (`actions/checkout@<sha> # v6`). Supply-chain hardening: a moved tag can't
  silently change a workflow. Renovate keeps the `# vX` comment current and
  auto-merges digest bumps.
- **`docker:pinDigests`** — pins `FROM` base-image digests too. On its own this
  would churn hard (distroless/chainguard rebuild constantly), so the **container
  base images** group below collapses every base-image update into **one weekly
  PR** (`schedule: before 6am on monday`, `minimumReleaseAge: "0 days"`) — the
  weekly cadence is the control, not a per-rebuild PR.
- **`minimumReleaseAge: "3 days"`** with **`internalChecksFilter: "strict"`** — an
  update PR isn't even raised until the release is 3 days old, so a yanked or
  broken-but-CI-passing release is caught by the ecosystem before it auto-merges.
- **`prHourlyLimit: 0` + `prConcurrentLimit: 0`** — no throttle. Since everything
  auto-merges, updates land as fast as CI clears them rather than queueing.
- **`postUpdateOptions: ["gomodTidy"]`** — after a `go.mod` bump, `go mod tidy`
  runs so `go.sum` stays consistent and the PR merges cleanly (Go repos; a no-op
  elsewhere).
- **`osvVulnerabilityAlerts` + `vulnerabilityAlerts`** — OSV-based vulnerability
  detection; security-fix PRs get a `security` label and **skip the 3-day soak**
  (`minimumReleaseAge: "0 days"`) so fixes land immediately.
- **`timezone: "Europe/Berlin"`** — the dependency dashboard and schedules render
  in the maintainer's local time.
- **Per-ecosystem grouping** (`packageRules`) — updates are grouped by language /
  tooling so each lands as a single PR: `go modules`, `java dependencies`,
  `rust crates`, `javascript dependencies`, `python dependencies`, … plus
  `github actions`, `container base images`, and `helm charts`. Fewer PRs, and
  fewer same-file conflicts between sibling Renovate PRs. A repo that doesn't use
  an ecosystem simply never sees that group. (Trade-off: a breaking major inside
  a group blocks that group's auto-merge until resolved — restrict a group with
  `matchUpdateTypes` if you'd rather isolate majors.)

The policy is deliberately "pay in tests, not manual review": breaking changes are caught by each
repo's CI gate rather than by holding majors back for hand inspection. The only non-test safeguard is
the short release-age soak above — an automated wait for the ecosystem to flag bad releases, not a
manual gate; security fixes skip it. A consuming repo that wants a stronger safety net should
strengthen its tests, not gate the auto-merge.

## Per-repo overrides

Anything a single repository sets in its own `renovate.json` takes precedence over the preset, so a repo
can opt out of a specific behavior locally without changing the shared default — e.g. disable automerge
for one package:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>metio/renovate-config"],
  "packageRules": [
    { "matchPackageNames": ["some-risky-dep"], "automerge": false }
  ]
}
```

## Validation

[`validate.yml`](.github/workflows/validate.yml) runs `renovate-config-validator --strict` against
`default.json` on every push and pull request, so a malformed preset never reaches the repositories that
depend on it. [`reuse.yml`](.github/workflows/reuse.yml) enforces [REUSE](https://reuse.software/)
licensing compliance.

## License

[0BSD](LICENSES/0BSD.txt).
