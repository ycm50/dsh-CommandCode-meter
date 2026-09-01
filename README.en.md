# dsh-CommandCode-meter

<div align="center">

**DeepSeek Harness session cost tracking plugin · Command Code GOAT edition (bilingual UI)**

A GOAT-customized fork of [dsh-cost-meter](https://github.com/Han-1413141/dsh-cost-meter). All upstream features (per-conversation cost · official balance · budget box · custom provider balance · history · peak/off-peak pricing · official price sync · coding-plan quotas · token heat grid, …) are unchanged — only the **OpenCode Go quota card was replaced with the Command Code GOAT subscription quota**.

[![license](https://img.shields.io/badge/license-MIT-green)](LICENSE)
[![upstream](https://img.shields.io/badge/upstream-Han--1413141%2Fdsh--cost--meter-4176E6)](https://github.com/Han-1413141/dsh-cost-meter)

English | [中文](README.md)

</div>

---

## Upstream & acknowledgements

This plugin is a customized fork of [dsh-cost-meter](https://github.com/Han-1413141/dsh-cost-meter). The upstream `dsh-cost-meter` is a feature-complete **DeepSeek Harness session cost tracking plugin**: per-conversation cost, official balance query, budget box, custom provider balance, history, peak/off-peak pricing, official price sync, Coding Plan quotas, token heat grid, and more — with a built-in 90+ model price catalog and auto-matching, in a bilingual (zh/en) UI. It is maintained by [Han-1413141](https://github.com/Han-1413141) under the [MIT](LICENSE) license.

**Why this fork**: we use the **Command Code GOAT** subscription (rather than OpenCode Go) day to day. The upstream Go-quota card only talks to `opencode.ai/zen/go/v1/usage`, so it cannot query Command Code subscription quotas. This repo therefore forks the project and replaces the Go-quota card with the official Command Code endpoint `GET https://api.commandcode.ai/alpha/billing/credits` (Bearer auth), adding 5-hour / weekly / monthly-pool “used / total + progress bar” displays; credential resolution was also switched to the DSH credential store / `COMMANDCODE_API_KEY` environment variable. Everything except the Go-quota card code — features, structure, docs — is inherited from upstream unchanged.

**Acknowledgements**: thanks to [Han-1413141](https://github.com/Han-1413141) and all contributors of upstream dsh-cost-meter — the vast majority of this project's code and design comes from upstream; we only layered a targeted customization on top. Upstream repo: [Han-1413141/dsh-cost-meter](https://github.com/Han-1413141/dsh-cost-meter) (MIT).

---

## What changed in this fork

| Change | Details |
|---|---|
| Quota endpoint | Swapped from `opencode.ai/zen/go/v1/usage` to the **official Command Code endpoint** `GET https://api.commandcode.ai/alpha/billing/credits` (Bearer auth) |
| Three windows | Rolling **5h / weekly / monthly pool** all show “used / total” numbers + individual progress bars (upstream only showed the main window's percent) |
| Monthly pool math | GOAT monthly pool is **70 USD** = monthly credits pool; `used = 70 − monthlyCredits`, `cap = 70` (constant `GOAT_MONTHLY_POOL` in [lib/index.js](lib/index.js); update it when switching plans: Go=$10 / Pro=$80 / Max 10x=$150 / Max 20x=$300) |
| Sidebar | GOAT box renders the same three-window used/total + progress-bar style as the Settings page; the **budget box lost its budget progress bar** (budget, used %, today's cost & share, used/limit remain) |
| Credentials | Key resolution is now **DSH credential store `COMMANDCODE_API_KEY`** → env `COMMANDCODE_API_KEY` → legacy config `goQuota.apiKey` (the opencode login-state auto-detect was removed) |
| Copy | All “OpenCode Go” strings → “Command Code GOAT”; new `goQuotaUsedOf` (used/total) i18n key (zh/en) |

## Installation

> Requirements: Node.js ≥ 20 + DeepSeek Harness (a version with the `dsh plugin` command).

```sh
# Option 1: local directory (after cloning or extracting)
dsh plugin --profile web add link:./dsh-CommandCode-meter

# Option 2: git URL (once the repo is published)
dsh plugin --profile web add github:ycm50/dsh-CommandCode-meter

# Option 3: dshmarket (if submitted to the marketplace)
dsh plugin --profile web add dsh-CommandCode-meter
```

After installing, **restart** `dsh web` (plugin rows, the Typert manifest and the client bundle are all scanned at startup):

```sh
dsh web
```

> Note: the shipped package references `scripts/build.mjs` and `test/verify.mjs` in `scripts.build` / `scripts.test`, but those files are not shipped (`files` only includes `lib`, `cordis.patch.yml`, `docs/provider-pricing.json` — same as upstream). `npm run build` needs those files copied from the upstream repo; the plugin does not need them at runtime.

## Configuring COMMANDCODE_API_KEY

Enter the key in **Settings → Cost → Quota (GOAT card) → API key input** (write-only, never echoed; stored in the DSH credential store), or set `COMMANDCODE_API_KEY` in the DSH credential vault / environment beforehand. Resolution order:

1. DSH credential store (the Settings input saves here too)
2. Environment variable `COMMANDCODE_API_KEY`
3. Legacy config `goQuota.apiKey` (migration only)

> ⚠️ `GOAT_MONTHLY_POOL = 70` is a hardcoded constant in [lib/index.js](lib/index.js) — verify it against the Command Code plan you actually subscribe to. The endpoint is only contacted after you explicitly enable the GOAT quota.

## Relation to upstream

- Upstream: [Han-1413141/dsh-cost-meter](https://github.com/Han-1413141/dsh-cost-meter) (MIT)
- This repo is a modified fork; only the Go-quota card code changed (see the table above) — everything else inherits upstream;
- When the upstream plugin updates, this fork's edits get overwritten — to rebase, re-apply the corresponding changes in this repo's `lib/index.js` / `lib/store.js` / `lib/client.js` onto the new upstream (see [CHANGELOG.md](CHANGELOG.md) for the itemized diff).

## Development & verification

```sh
node --check lib/index.js && node --check lib/pricing.js \
  && node --check lib/store.js && node --check lib/typert.host.js \
  && node --check lib/client.js         # syntax checks
dsh --profile web --dump-config         # composition-tree check
dsh --profile web --port 3099           # real startup (watch logs and the UI)
```

## License

[MIT](LICENSE) © 2026 dsh-cost-meter contributors (GOAT fork © 2026)
