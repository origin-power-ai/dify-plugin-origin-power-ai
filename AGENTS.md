# AGENTS.md — origin_power_ai (Dify model provider plugin)

Context for coding agents working on this plugin. Read this before changing anything that
affects the Marketplace listing — several rules here are non-obvious and have cost us
rejected PRs.

## What this is

A Dify **model provider plugin** for the Origin Power AI gateway (an OpenAI-compatible
endpoint aggregating GLM / MiniMax models). Published on the Dify Marketplace as
`origin-power-ai/origin_power_ai` (Verified by Dify).

- Source repo (this one): `origin-power-ai/dify-plugin-origin-power-ai` (org-owned; moved
  from `Ikki6666/origin-power-ai`, which 301-redirects here)
- Marketplace page (active): https://marketplace.dify.ai/plugin/origin-power-ai/origin_power_ai
- Legacy listing `ikki6666/origin_power_ai` is **deprecated** (Dify added the notice + redirect);
  never publish new versions there
- Upstream registry repo (where PRs go): https://github.com/langgenius/dify-plugins

## Layout

| Path | What it is |
|---|---|
| `manifest.yaml` | Plugin identity (`author`/`name` = Marketplace ID), version, permissions, `repo`/`contact`/`privacy` |
| `provider/origin_power_ai.yaml` | Provider label/icon, credential forms, `predefined-model` + `customizable-model` |
| `provider/origin_power_ai.py` | Provider credential validation (`GET /models`, no token spend) |
| `models/llm/llm.py` | LLM implementation: subclasses SDK's `OAICompatLargeLanguageModel`; adds `_merge_credentials()` so **predefined** models inherit mode/tool/vision settings from their YAML |
| `models/llm/*.yaml` | Predefined model catalog (5 GLM models) + `_position.yaml` ordering |
| `_assets/icon.svg` | Brand mark (official logo, vector) |
| `README.md`, `readme/README_zh_Hans.md`, `PRIVACY.md` | Docs; PRIVACY.md is **required** for the Marketplace |
| `docs/migrate-marketplace-author-to-org.md` | Checklist for moving listing ownership to the org (plan B) |

## Develop & debug

```bash
uv sync                                   # create/refresh .venv (Python 3.12)
uv run python -m main                     # remote-debug against a Dify instance
```

Remote debugging needs `.env` (copy `.env.example`): `REMOTE_INSTALL_URL` + `REMOTE_INSTALL_KEY`
from the target Dify → Integrations → **Debug** panel. For a self-hosted instance port-forward the
daemon first — and pick a **free local port** (this Mac already has `5003` taken by a local Docker
Dify, use e.g. `15003`):

```bash
kubectl -n dify-system port-forward deploy/dify-plugin-daemon 15003:5003
```

Package a release build (CLI = `dify-plugin-<os>-<arch>` binary from the
`langgenius/dify-plugin-daemon` releases, renamed to `dify`):

```bash
dify plugin package ./ -o origin_power_ai-<version>.difypkg
```

## Rules that bite (learned the hard way)

1. **`.difyignore` must exclude `.git/`** — otherwise the whole git history is embedded in the
   `.difypkg` (a 325 KB plugin shipped as 602 KB). Verify after packaging.
2. **`author` must match `^[a-z0-9_-]{1,64}$`** (the daemon rejects uppercase at register time) and
   must be the GitHub handle that owns the `dify-plugins` fork used to open the PR.
3. **One `.difypkg` change per PR, nothing else.** `check-pkg-paths.py` fails a PR containing any
   second file (even a README edit). Never bundle source changes with a submission.
4. **Base the PR branch on current upstream `main`.** A stale base makes the 3-way diff pull in
   other people's files → the "only one file change" failure. Rebase (or use the
   `PUT /pulls/{n}/update-branch` API) before/after opening the PR.
5. **PR body must follow the official template and be English-only** (any CJK character fails the
   language check). Required sections: Plugin information (Author / Plugin name / Version / Source
   repository / Contact), Submission type (exactly one checkbox), What changed, **Local validation**,
   **Security and privacy notes** (mandatory whenever the scanner detects network calls — ours has
   two: `requests.get` in the provider, `requests.post` in the LLM), Risk level (exactly one).
6. **manifest required fields**: `repo` (URL), `contact` (valid email), `privacy` (relative path or
   URL). Recommended: `meta.minimum_dify_version`.
7. **Versions are keyed per author directory** — a new version must be higher than the sibling
   `.difypkg` files in `<author>/<plugin_name>/`. Same version under a *different* author is fine.
8. **Fork PRs need a maintainer to approve each workflow run** (`action_required`), which can take
   hours to days. A short, polite ping on the PR or a post in Dify Discussions is the practical
   remedy; escalation to an issue in `langgenius/dify-plugins` works for process questions.

## Publishing a release

1. Bump `version` in `manifest.yaml` (and any user-facing changelog/Release notes).
2. Commit, then in GitHub → Actions → **Publish plugin to Dify Marketplace (auto PR)** → *Run
   workflow*. It packages the plugin and opens the PR to `langgenius/dify-plugins` from the fork
   named after `manifest.author` — i.e. `origin-power-ai/dify-plugins` — so `PLUGIN_ACTION` must be
   a PAT that can push there.
3. Watch the PR checks (`apply-risk-label`, `pre-check-plugin`); fix per the rules above.
4. After merge, the Marketplace listing updates automatically. Optionally attach the `.difypkg` to
   a GitHub Release here as well.

## Do not

- Don't rename `name` or change `author` casually — that changes the Marketplace identity and means
  a **new listing** (existing users must reinstall). See the plan-B checklist for the sanctioned way.
- Don't commit `.env`, debug keys, or API keys (`.gitignore` covers `.env`).
- Don't submit a PR when the only change is a version bump caused by an unrelated doc edit.

## Current state (2026-09-14)

- **Marketplace (active)**: `origin-power-ai/origin_power_ai` **v1.0.0**, Verified by Dify —
  https://marketplace.dify.ai/plugin/origin-power-ai/origin_power_ai
- **Legacy listing**: `ikki6666/origin_power_ai` v0.0.5 now shows the deprecation notice
  ("deprecated due to ownership transferred") and points users to the new one.
- Migration answered and executed the same day (issue #3066 → PR **#3078**, single-file, checks green
  first try, merged in ~5 minutes). Maintainer guidance: ownership **cannot** be transferred; publish
  a new package under the org and the team adds the deprecation notice/redirect. Do **not** publish
  further versions under `ikki6666/...`.
- Executed procedure + rationale: `docs/migrate-marketplace-author-to-org.md`.
- **Open follow-up**: claim the listing at https://creators.dify.ai with an organization account
  (recommended by the maintainer) to track feedback/likes/stars.
- Note for self-hosted users (including the company instance): switching from the old listing to the
  new one means uninstalling and reinstalling the plugin — Dify does not migrate installed plugins
  between identities.

