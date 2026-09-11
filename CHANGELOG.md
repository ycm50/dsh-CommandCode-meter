# Changelog

本项目为 [dsh-cost-meter](https://github.com/Han-1413141/dsh-cost-meter) v1.6.12 的 **Command Code GOAT 定制 fork**。上游逐条开发记录见上游仓库 `CHANGELOG.md`;本文件只记录本 fork 相对上游 v1.6.12 的改动。

## 1.6.12-goat.3 (2026-09-11)

### 修复:commandcode 路由的调用费用恒为 0

**现象**:`今日费用` 显示 ¥0,但账本里当天有 220 次调用、60 万输入 token、2240 万 cacheRead——token 都记下了,金额全是 0。

**根因**:调用走 provider `commandcode`(`https://api.commandcode.ai/provider/v1`),而价格表里**既没有 `commandcode` 这个 provider,主表也没有 `deepseek-v4.1-flash`**。`providerPriceEntryFor` 于是走到「未定价」分支返回 `priced: false` → `cost` 恒 0;而「今日费用」用的正是 `apiCost`(真金白银口径),于是显示 ¥0。这不是本轮之前两个版本引入的,而是本 fork 一直存在的缺口——`docs/provider-pricing.json` 里也从来没有 commandcode。

**改动**:

- **新增 `commandcode` provider 价格表**([lib/pricing.js](lib/pricing.js),49 个模型),数据取自官方文档 <https://commandcode.ai/docs/plans/goat> 的「Every model on the GOAT plan is listed below」表,每条都带 `sourceUrl` / `checkedAt`。模型 id 采用 Provider API 写法(与本机 `settings.yaml` 中 commandcode provider 的模型 id 一致)。
  - DeepSeek 系列按官方**峰谷两档**计价(谷 $0.15/$0.60/$0.003、峰 $0.30/$1.20,窗口 01–04 与 06–10 UTC 周一至周五,与 DeepSeek 官方同一窗口),复用既有 `billingMode: 'deepseek-peak'` 与周末全谷价规则。官方只公布了峰的输入/输出;缓存读峰值按官方每对价格恒为 2× 的规律推为 2× 并**显式写出**——若不写,`completeTier` 会用 `cacheMiss` 兜底,把 2240 万缓存读按未命中价计费($4.63,虚高 19 倍)。
  - 其余模型按官方单档报价;官方标注 `-50% / -98% / -99%` 的促销价按折后价录入并附挂牌价注释。
  - 免费模型(LongCat 2.0 / Laguna S 2.1)显式记零价而非「未定价」,语义是「确定不花钱」。
- **启动补账**([lib/store.js](lib/store.js)):`repairLedgerPricing` 此前**跳过**所有 `deepseek-peak` 桶(担心用正午近似重算会压低峰时已计费用)。改为:峰谷桶**有 token 但金额为 0** 时仍然补账,已有金额的保持不动——零金额桶不存在「误伤」问题,补一个谷价下界远好于恒 0。这一条保证修复对**已记录的历史**生效,而不只是对新调用生效。
- `PROVIDER_MODEL_FAMILIES` 补 `commandcode` 分组(设置页拓展价格表按家族展示)。

**实测**(本机 ledger,2026-09-11):修复后当天由 `$0` → `$0.2433`(≈¥1.75),与按谷价手算逐位一致(`(595137×0.15 + 144492×0.60 + 22428288×0.003) / 1e6`);历史 4 天金额一字未动。

> 注:当日补账按**谷价**计(日粒度桶无法还原峰谷拆分),实际账单在峰时段会略高;自修复起的新调用按真实时刻走峰谷两档,是准确的。若需把历史一次性按真实时刻重算,可切换一次「价格币种」(设置 → 费用)触发全量重放重定价。

## 1.6.12-goat.2 (2026-09)

### 计费口径修正:月度额度池不再硬编码

按官方文档复核 GOAT 定价与限额后,修正月度池的计费口径(来源见下)。

- **套餐表集中维护**:新增 `COMMAND_CODE_PLANS` / `COMMAND_CODE_PLAN_IDS` / `DEFAULT_COMMAND_CODE_PLAN` / `GOAT_WINDOW_CAPS` / `normalizeMonthlyPool`([lib/coding-plans.js](lib/coding-plans.js)),收录全部 6 个套餐的月费、月度 Credits 与 5 小时 / 周上限:

  | 套餐 | 月费 | 月度 Credits | 5h 上限 | 周上限 |
  |---|---|---|---|---|
  | Go | $1 | $10 | $3 | $6 |
  | **GOAT** | **$10** | **$70** | **$14** | **$35** |
  | Pro | $20 | $80 | $16 | $40 |
  | Max 10× | $100 | $150 | $45 | $90 |
  | Max 20× | $200 | $300 | $90 | $180 |
  | Team Pro | $40 | $40 | $12 | $24 |

- **月度池改为可配置**:新增 `goQuota.monthlyPool`(设置页「月度额度池」数字输入框,留空 = GOAT 默认 $70)。原因是官方 `monthlyCredits` 是**当前计费模型那一档**的剩余额,而非账户全局剩余——各模型月度额度从 $70(GLM-5.2 / GPT-5.6 Sol / Tencent Hy3)、$60(DeepSeek V4 Flash 系列 / Kimi K2.7 Code)、$47(MiniMax M3)、$40(GLM-5.3 Flash / Gemini 3.8 Flash)到 $20 不等。用固定 $70 去减 $40 档的模型会算出虚高甚至为负的已用。
- **一致性告警**:接口剩余额 > 配置池(池填小了 / 官方给了更高档位)时不再算出负数,而是把 `poolTooSmall` 透出;侧边栏与设置页 GOAT 卡片显示警告文案(`goQuotaPoolTooSmall`),提示把池改成主力模型对应额度。
- **文案修正**:`goQuotaNoUsage` 由「缺少 windowLimits 字段」改为「未解析出任何用量窗口(缺少 windowLimits 与 monthlyCredits)」,与实际判定逻辑一致——现在只要有月度 Credits 就能出数,不再因缺 `windowLimits` 直接报错。
- **默认值来源**:`GOAT_MONTHLY_POOL` 由字面量 70 改为取 `COMMAND_CODE_PLANS.goat.monthlyCredits`,避免套餐表与常量两处漂移。
- **未改动**:5 小时 / 周两档仍原样采信接口 `windowLimits.*.used/cap`(官方上限 $14 / $35 由官方返回,不依赖本地常量)。

**数据来源**(2026-09 抓取复核):

- <https://commandcode.ai/docs/plans/goat> — GOAT:$10/月 → $70 等效用量,7× 倍数
- <https://commandcode.ai/docs/resources/pricing-limits> — 全套餐对照表 + 各模型费率
- <https://commandcode.ai/docs/resources/usage-limits> — 逐套餐 5 小时 / 周上限;「1 credit = $1 用量」仅在满额模型成立

## 1.6.12-goat.1 (2026)

### 核心:Go 额度卡 → Command Code GOAT

- **接口替换**:`lib/index.js` 中 `GO_QUOTA_URL` 由 `https://opencode.ai/zen/go/v1/usage` 改为 `https://api.commandcode.ai/alpha/billing/credits`(Bearer 认证,`fetchWithRetry` 15s 超时,UA `dsh-cost-meter/1.6 (DeepSeek Harness plugin)`)。
- **响应解析**:新增 `windowOfCommandCode(raw)` 解析官方 `windowLimits.fiveHour / weekly`(used/cap/resetAt,`resetAt` 为 epoch **毫秒**),月度池按 `monthlyCredits` 计算:
  - `used = GOAT_MONTHLY_POOL − monthlyCredits`,`cap = GOAT_MONTHLY_POOL`;
  - 新增常量 `GOAT_MONTHLY_POOL = 70`(Go=$10 / Pro=$80 / Max 10x=$150 / Max 20x=$300,注释说明);
  - `emptyGoQuota()` 返回结构扩展为 `{status,message,fetchedAt,rolling,weekly,monthly}`。
- **错误处理**:401/403 → 软错误「未订阅」(goQuotaNoSub);缺 `windowLimits` → goQuotaNoUsage(新文案「缺少 windowLimits 字段」);全部消息文案 "OpenCode Go" → "Command Code GOAT"。

### 凭据

- **Key 解析重写**:`resolveGoKey` 优先级 = DSH 凭据库 `COMMANDCODE_API_KEY` → 环境变量 `COMMANDCODE_API_KEY` → 旧配置 `goQuota.apiKey`;删除 `findGoKeyInAuthJson`(不再读取 opencode 登录态)。
- `lib/store.js`:`SECRET_REF_MAP.goQuota` 由 `'OPENCODE_GO_API_KEY'` 改为 `'COMMANDCODE_API_KEY'`,设置页保存的 Key 与查询解析一致。

### 客户端 UI(lib/client.js)

- **设置页 Go 面板 + 侧边栏 GOAT 图框**:三档窗口(滚动 5 小时 / 本周 / 月度池)全部显示「已用 / 全部」数值(`goQuotaUsedOf` 新 i18n 键,`{used} / {cap}`)+ 各自进度条;月度池 percent = 已用/70×100。
- **侧边栏**:GOAT 图框主体改为三行 `cm-go-row`(短标签 + 进度条 + 已用/全部),与设置页同款样式;窄栏 rail 保留主窗口百分比。
- **预算图框**:移除预算进度条渲染(`cm-bbox-bar` 行删除),保留预算 / 已用% / 今日费用与占预算% / 已用/额度。
- **文案**:中英双语全部 "OpenCode Go" → "Command Code GOAT";新增 `goQuotaUsedOf`、`goQuotaKeyLabel`(Command Code API Key)等键。

### 已知差异

- 月度池显示为「已用 %」(池 − 剩余),因为 Go 卡片客户端只渲染基于 percent 的窗口;官方 CLI `/usage` 显示 5h/周百分比 + 剩余余额,口径略有不同。
- ~~`GOAT_MONTHLY_POOL = 70` 为硬编码常量,更换套餐需同步修改。~~ → 见 1.6.12-goat.2:已改为可配置(`goQuota.monthlyPool`)并集中到套餐表。
- 上游更新会覆盖本 fork 的全部改动;迁移方式见 README「与上游的关系」。
