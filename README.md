# renovate-config

Organization-wide [Renovate](https://docs.renovatebot.com/) configuration preset for all `metio`
repositories. It exists so dependency-update policy lives in **one place** instead of being copied into
every repo's `renovate.json`.

## Using the preset

In any repository's `renovate.json`, extend this preset:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["local>metio/renovate-config"]
}
```

`local>` resolves the preset through the platform Renovate is already running on (GitHub), so it works
for private repositories too. The reference points at [`default.json`](default.json) on this repo's
default branch — that is the file Renovate reads.

## What the preset does

[`default.json`](default.json):

- **`config:recommended`** — Renovate's recommended baseline (dependency dashboard, sensible
  scheduling, auto-detected managers: Maven, GitHub Actions, Dockerfile/Containerfile, npm, …).
- **`automerge: true`** with **`major.automerge: true`** — every update, including major-version bumps,
  is auto-merged once required status checks pass. Auto-merge uses the platform's native mechanism
  (`platformAutomerge`), so a repo's branch protection still gates the merge.
- **`assignees: ["sebhoss"]`** — update PRs are assigned to the maintainer.

The policy is deliberately "pay in tests, not manual review": breaking changes are caught by each
repo's CI gate rather than by holding majors back for hand inspection. A consuming repo that wants a
stronger safety net should strengthen its tests, not gate the auto-merge.

## Per-repo overrides

Anything a single repository sets in its own `renovate.json` takes precedence over the preset, so a repo
can opt out of a specific behavior locally without changing the shared default — e.g. disable automerge
for one package:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["local>metio/renovate-config"],
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
