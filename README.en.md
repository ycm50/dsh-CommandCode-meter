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
| Monthly pool math | GOAT monthly pool is **$70 of usage**; `used = pool − monthlyCredits`, `cap = pool`. The pool is **configurable** (Settings → “Monthly credit pool”, default $70), because the official `monthlyCredits` value is the remaining allowance of the **model tier you are currently billing against** — per-model allowances differ (see the table below) |
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

> ⚠️ The monthly pool is no longer a hardcoded constant: the $70 default comes from `COMMAND_CODE_PLANS.goat.monthlyCredits` in [lib/coding-plans.js](lib/coding-plans.js) and can be overridden in **Settings → Cost → Quota (GOAT card) → Monthly credit pool**. The endpoint is only contacted after you explicitly enable the GOAT quota.

## Verified GOAT pricing (from the official docs)

| Item | Value |
|---|---|
| Plan price | **$10 / month** |
| Monthly credits | **$70 of usage** (a 7× multiplier) |
| Rolling 5-hour cap | **$14** (opens on the first request of the window, not a fixed clock) |
| Weekly cap | **$35** |

> The 5-hour and weekly caps come straight from the API's `windowLimits.fiveHour / weekly` `used`/`cap` fields and are taken as-is; the “Monthly credit pool” setting does not affect them.

**Per-model monthly allowances differ** (official model table): $70 (GLM-5.2 / GPT-5.6 Sol / Tencent Hy3), $60 (DeepSeek V4 Flash family / Kimi K2.7 Code), $47 (MiniMax M3), $40 (GLM-5.3 Flash / Gemini 3.8 Flash), $33 (Qwen 3.7 family), $30 (MiMo V2.5), $20 (other new models). The API's `monthlyCredits` is the remaining allowance of the **model you are currently billing against**, so `used = pool − remaining` only holds when the pool is that same tier's allowance. That is why the pool is a setting: blank = GOAT default $70; switch your main model to DeepSeek V4 Flash and set `60`. If the pool is set too small (remaining > pool) the panel warns instead of reporting inflated usage.

**Other plans** (set the corresponding value in “Monthly credit pool” when you switch):

| Plan | Price/mo | Monthly credits | 5-hour cap | Weekly cap |
|---|---|---|---|---|
| Go | $1 | $10 | $3 | $6 |
| **GOAT** | **$10** | **$70** | **$14** | **$35** |
| Pro | $20 | $80 | $16 | $40 |
| Max 10× | $100 | $150 | $45 | $90 |
| Max 20× | $200 | $300 | $90 | $180 |
| Team Pro | $40 | $40 | $12 | $24 |

These values live in `COMMAND_CODE_PLANS` in [lib/coding-plans.js](lib/coding-plans.js), sourced from three official pages (fetched 2026-09):

- <https://commandcode.ai/docs/plans/goat> (GOAT: $10/mo → $70, the 7× multiplier)
- <https://commandcode.ai/docs/resources/pricing-limits> (full plan comparison + per-model rates)
- <https://commandcode.ai/docs/resources/usage-limits> (per-plan 5-hour / weekly caps)

## Model pricing (the `commandcode` route)

If your model route points at Command Code's Provider API (`provider: commandcode`, baseURL
`https://api.commandcode.ai/provider/v1`), the price table has to know that provider — otherwise
calls record **tokens only and a cost of 0** (so "Today's cost" reads ¥0). This fork ships a
built-in **`commandcode` provider table (49 models)** sourced from the official model table, with
`sourceUrl` / `checkedAt` on every entry.

- The DeepSeek family uses the official **peak / off-peak** tiers: off-peak $0.15 / $0.60 / $0.003,
  peak $0.30 / $1.20 (peak windows 01–04 and 06–10 UTC, Mon–Fri — the same windows DeepSeek itself uses).
- Other models use the official flat rates; official `-50% / -98% / -99%` promo prices are recorded at
  the discounted value with the list price noted.
- Model ids follow the Provider API (e.g. `deepseek/deepseek-v4.1-flash`, `zai-org/GLM-5.2`); case and
  separator differences are absorbed by canonical matching.

To add or change entries yourself, use **Settings → Cost → Extended price table**. **Sync official
prices** only overwrites the DeepSeek main table; provider tables must be checked by hand.

> After a price-table update, startup automatically re-costs historical buckets that have tokens but a
> zero amount (billed at off-peak). To re-price history precisely at its real peak/off-peak instants,
> toggling the **pricing currency** once triggers a full replay recompute.

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
