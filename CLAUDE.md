# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This repository holds the **organization-wide Renovate preset** for all `metio` repositories. It is a
configuration-only repo — no application code, no build, no release. The whole point is that
dependency-update policy lives here once and every other repo extends it via
`"extends": ["local>metio/renovate-config"]`.

## The one file that matters: `default.json`

`default.json` is the preset Renovate actually reads. The name is **not arbitrary** — `local>metio/renovate-config`
(and `github>metio/renovate-config`) resolves to `default.json` at the repo root on the default branch.
**Do not rename or move it**, or every consuming repo's `extends` reference breaks.

A consuming repo extends this preset and may override any setting locally; the repo's own
`renovate.json` takes precedence over the preset.

## Changing the policy

Editing `default.json` changes dependency behavior across **every** metio repo on their next Renovate
run — there is no per-repo rollout. Treat changes here as org-wide. Current intent:

- `config:recommended` is the baseline.
- `automerge: true` + `major: { automerge: true }` auto-merge **all** updates, including majors, once
  required checks pass — this is deliberate ("pay in tests, not manual review"). Do **not** add an
  update-type gate to hold majors back; that has been explicitly rejected. Strengthen consuming repos'
  CI instead if correctness is a concern.
- `assignees: ["sebhoss"]` assigns update PRs to the maintainer.

## Conventions

- **License (REUSE):** the project is **0BSD**. Every file needs an `SPDX-FileCopyrightText`
  (`The renovate-config Authors`) and `SPDX-License-Identifier` (`0BSD`) declaration. YAML workflows
  carry these inline as `#` comments; files that cannot carry comments (`default.json`) and the Markdown
  docs are covered by globs in `REUSE.toml`. The `reuse.yml` workflow enforces this with `fsfe/reuse-action`.
- **Validation:** `validate.yml` runs `renovate-config-validator --strict default.json` on every push
  and PR. Run it locally with `npx --yes --package renovate -- renovate-config-validator --strict default.json`.
  Keep the preset valid — a broken `default.json` propagates to every dependent repo.
- **This repo dogfoods itself:** it does not need its own `renovate.json` to be useful, but if one is
  added it should `extends: ["local>metio/renovate-config"]` like everyone else.
