# dsh-CommandCode-meter

<div align="center">

**DeepSeek Harness 会话费用统计插件 · Command Code GOAT 定制版(界面中英双语)**

[dsh-cost-meter](https://github.com/Han-1413141/dsh-cost-meter) 的 GOAT 定制 fork:原插件的全部功能(本会话费用 · 官方余额 · 预算图框 · 自定义 Provider 余额 · 历史记录 · 峰谷计价 · 官方价格同步 · Coding Plan 额度 · Token 热图等)保持不变,仅将 **OpenCode Go 额度卡替换为 Command Code GOAT 订阅额度查询**。

[![license](https://img.shields.io/badge/license-MIT-green)](LICENSE)
[![upstream](https://img.shields.io/badge/upstream-Han--1413141%2Fdsh--cost--meter-4176E6)](https://github.com/Han-1413141/dsh-cost-meter)

[English](README.en.md) | **中文**

</div>

---

## 源仓库与致谢

本插件是 [dsh-cost-meter](https://github.com/Han-1413141/dsh-cost-meter) 的定制 fork。上游 `dsh-cost-meter` 是一个功能完整的 **DeepSeek Harness 会话费用统计插件**:提供本会话费用、官方余额查询、预算图框、自定义 Provider 余额、历史记录、峰谷计价、官方价格同步、Coding Plan 额度、Token 热图等功能,内置 90+ 模型价格目录与自动匹配,界面中英双语,由 [Han-1413141](https://github.com/Han-1413141) 以 [MIT](LICENSE) 许可开源维护。

**改动原因**:我们日常使用 **Command Code GOAT** 订阅(而非 OpenCode Go)。上游插件的 Go 额度卡只对接 `opencode.ai/zen/go/v1/usage`,无法查询 Command Code 的订阅额度。因此 fork 本仓库,将 Go 额度卡整体替换为 Command Code 官方接口 `GET https://api.commandcode.ai/alpha/billing/credits`(Bearer 认证),并补充 5 小时 / 本周 / 月度池三档「已用 / 全部 + 进度条」展示;凭据解析也改为 DSH 凭据库 / 环境变量 `COMMANDCODE_API_KEY`。除 Go 额度卡相关代码外,其余功能、结构、文档均继承上游,未作改动。

**致谢**:感谢 [Han-1413141](https://github.com/Han-1413141) 及上游 dsh-cost-meter 的所有贡献者——本项目绝大部分代码与设计都来自上游,我们只是在其基础上做了一层针对性定制。上游仓库:[Han-1413141/dsh-cost-meter](https://github.com/Han-1413141/dsh-cost-meter)(MIT)。

---

## 本 fork 改了什么

| 改动 | 说明 |
|---|---|
| 额度接口 | 由 `opencode.ai/zen/go/v1/usage` 换成 **Command Code 官方端点** `GET https://api.commandcode.ai/alpha/billing/credits`(Bearer 认证) |
| 三档窗口 | 滚动 **5 小时 / 本周 / 月度池** 三档,全部显示「已用 / 全部」数值 + 各自进度条(原版仅主档位显示百分比) |
| 月度池口径 | GOAT 月额度池 **$70 等效用量**;`used = 池 − monthlyCredits`,`cap = 池`。**池可配置**(设置页「月度额度池」,缺省 $70),因为官方 `monthlyCredits` 是**当前计费模型那一档**的剩余额——各模型月度额度见下表 |
| 侧边栏 | GOAT 图框与设置页同款三窗口「已用/全部 + 进度条」样式;**预算图框去掉预算进度条**(保留预算、已用%、今日费用与占预算%、已用/额度) |
| 凭据 | Key 解析改为 **DSH 凭据库 `COMMANDCODE_API_KEY`** → 环境变量 `COMMANDCODE_API_KEY` → 旧配置 `goQuota.apiKey`(不再读取 opencode 登录态) |
| 文案 | 全部 "OpenCode Go" → "Command Code GOAT",新增 `goQuotaUsedOf`(已用/全部)i18n 文案(中/英) |

## 安装

> 需求:Node.js ≥ 20 + DeepSeek Harness(带 `dsh plugin` 命令)。

发布后(或本地路径):

```sh
# 方式一:本地目录(克隆或解压后)
dsh plugin --profile web add link:./dsh-CommandCode-meter

# 方式二:git URL(仓库发布后)
dsh plugin --profile web add github:ycm50/dsh-CommandCode-meter

# 方式三:dshmarket(若已提交到市场)
dsh plugin --profile web add dsh-CommandCode-meter
```

安装后**重启** `dsh web` 生效:

```sh
dsh web
```

> 注意:安装包内 `scripts.build` / `scripts.test` 引用的 `scripts/build.mjs` 与 `test/verify.mjs` 未随包发布(与上游一致,`files` 只含 `lib`、`cordis.patch.yml`、`docs/provider-pricing.json`),本地 git 克隆内不含这两个文件,`npm run build` 需要从上游仓库补齐;插件运行时不需要它们。

| 月额度池 | **$70 等效用量**(月度 Credits),已用 = 池 − 剩余 |
| 5 小时滚动窗上限 | **$14**(从本窗首次请求起算,非固定时钟) |
| 周滚动窗上限 | **$35** |

> 5 小时 / 周上限由官方接口 `windowLimits.fiveHour / weekly` 的 `used`/`cap` 直接返回,插件原样采信,不受「月度额度池」配置影响。

**各模型月度额度不同**(官方模型表):$70(GLM-5.2 / GPT-5.6 Sol / Tencent Hy3)、$60(DeepSeek V4 Flash 系列 / Kimi K2.7 Code)、$47(MiniMax M3)、$40(GLM-5.3 Flash / Gemini 3.8 Flash)、$33(Qwen 3.7 系列)、$30(MiMo V2.5)、$20(其余新模型)。接口返回的 `monthlyCredits` 是**当前计费那一个模型档位**的剩余额,所以「已用 = 池 − 剩余」只有在**池取成同一档的 allowance** 时才成立。插件因此把池做成配置项:留空 = GOAT 默认 $70;主力模型换成 DeepSeek V4 Flash 档就填 `60`。池填小了(剩余 > 池)面板会给出警告,不会算出虚高的已用。

**其他套餐**(换套餐时填对应值到「月度额度池」):

| 套餐 | 月费 | 月度 Credits | 5 小时上限 | 周上限 |
|---|---|---|---|---|
| Go | $1 | $10 | $3 | $6 |
| **GOAT** | **$10** | **$70** | **$14** | **$35** |
| Pro | $20 | $80 | $16 | $40 |
| Max 10× | $100 | $150 | $45 | $90 |
| Max 20× | $200 | $300 | $90 | $180 |
| Team Pro | $40 | $40 | $12 | $24 |

以上数值在 [lib/coding-plans.js](lib/coding-plans.js) 的 `COMMAND_CODE_PLANS` 中集中维护,来源为官方三页文档(2026-09 抓取复核):

- <https://commandcode.ai/docs/plans/goat>(GOAT:$10/月 → $70,7× 倍数)
- <https://commandcode.ai/docs/resources/pricing-limits>(全套餐对照表 + 各模型费率)
- <https://commandcode.ai/docs/resources/usage-limits>(逐套餐 5 小时 / 周上限)

## 模型计价(commandcode 路由)

如果你的模型路由指向 Command Code 的 Provider API(`provider: commandcode`,baseURL
`https://api.commandcode.ai/provider/v1`),价格表需要认识这个 provider,否则调用会
**只记 token、金额恒 0**(「今日费用」显示 ¥0)。本 fork 已内置 **`commandcode` provider
价格表(49 个模型)**,数据取自官方文档模型表,每条都带 `sourceUrl` / `checkedAt`。

- DeepSeek 系列按官方**峰谷两档**:谷 $0.15 / $0.60 / $0.003,峰 $0.30 / $1.20
  (峰时段 01–04 与 06–10 UTC 周一至周五,与 DeepSeek 官方同一窗口)。
- 其余模型按官方单档报价;官方 `-50% / -98% / -99%` 促销价按折后价录入。
- 模型 id 用 Provider API 写法(如 `deepseek/deepseek-v4.1-flash`、`zai-org/GLM-5.2`);
  大小写与连接符差异由归一化匹配兜住。

若要自行增改,在 **设置 → 费用 → 拓展价格表** 中挂载/编辑;官方调价后用
**同步官方价格** 只覆盖 DeepSeek 主表,provider 表需手动核对。

> 价格表更新后,启动时会自动为「有 token 但金额为 0」的历史桶补账(按谷价)。
> 若要把历史按真实峰谷时刻精确重算,切换一次「价格币种」会触发全量重放重定价。

## 配置 COMMANDCODE_API_KEY

在 **设置 → 费用 → 额度(Go 卡)→ 输入 API Key**(只写不回显,保存进 DSH 凭据库),或提前在 DSH 凭据库/环境变量中配置 `COMMANDCODE_API_KEY`。Key 解析优先级:

1. DSH 凭据库(设置页保存的 Key 也存这里)
2. 环境变量 `COMMANDCODE_API_KEY`
3. 旧配置 `goQuota.apiKey`(仅迁移用)

> ⚠️ 月度额度池不再是硬编码常量:默认 $70 来自 [lib/coding-plans.js](lib/coding-plans.js) 的 `COMMAND_CODE_PLANS.goat.monthlyCredits`,可在 **设置 → 费用 → 额度(Go 卡)→ 月度额度池** 覆盖。接口只在显式启用 GOAT 额度时才会出站请求。

## 与上游的关系

- 上游:[Han-1413141/dsh-cost-meter](https://github.com/Han-1413141/dsh-cost-meter)(MIT)
- 本仓库为其修改 fork,仅改动 Go 额度卡相关代码(见上方改动表),其余功能、结构、文档均继承上游;
- 上游插件更新时,本 fork 的改动会被覆盖——如有需要,请从上游新版重新应用本仓库 `lib/index.js` / `lib/store.js` / `lib/client.js` 中的对应改动(见 [CHANGELOG.md](CHANGELOG.md) 改动清单)。

## 开发与验证

```sh
node --check lib/index.js && node --check lib/pricing.js \
  && node --check lib/store.js && node --check lib/typert.host.js \
  && node --check lib/client.js         # 语法检查
dsh --profile web --dump-config         # 组合树校验
dsh --profile web --port 3099           # 真机启动(观察启动日志与 UI)
```

## License

[MIT](LICENSE) © 2026 dsh-cost-meter contributors(GOAT 定制 fork © 2026)
