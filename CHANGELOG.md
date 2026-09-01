# Changelog

本项目为 [dsh-cost-meter](https://github.com/Han-1413141/dsh-cost-meter) v1.6.12 的 **Command Code GOAT 定制 fork**。上游逐条开发记录见上游仓库 `CHANGELOG.md`;本文件只记录本 fork 相对上游 v1.6.12 的改动。

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
- `GOAT_MONTHLY_POOL = 70` 为硬编码常量,更换套餐需同步修改。
- 上游更新会覆盖本 fork 的全部改动;迁移方式见 README「与上游的关系」。
