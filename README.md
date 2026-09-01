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
| 月度池口径 | GOAT 套餐月额度池 **70 美元** = 月度余额池;`used = 70 − monthlyCredits`,`cap = 70`(常量 `GOAT_MONTHLY_POOL` 见 [lib/index.js](lib/index.js),换套餐需改:Go=$10 / Pro=$80 / Max 10x=$150 / Max 20x=$300) |
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

## 配置 COMMANDCODE_API_KEY

在 **设置 → 费用 → 额度(Go 卡)→ 输入 API Key**(只写不回显,保存进 DSH 凭据库),或提前在 DSH 凭据库/环境变量中配置 `COMMANDCODE_API_KEY`。Key 解析优先级:

1. DSH 凭据库(设置页保存的 Key 也存这里)
2. 环境变量 `COMMANDCODE_API_KEY`
3. 旧配置 `goQuota.apiKey`(仅迁移用)

> ⚠️ `GOAT_MONTHLY_POOL = 70` 是 [lib/index.js](lib/index.js) 中的硬编码常量,按你实际订阅的 Command Code 套餐核对;接口只在显式启用 GOAT 额度时才会出站请求。

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
