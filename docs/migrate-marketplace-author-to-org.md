# 执行清单：把插件市场身份迁到公司组织（方案 B）

> 背景：插件 v0.0.5 已以 `ikki6666/origin_power_ai` 上架。源码仓库已迁至 `origin-power-ai/dify-plugin-origin-power-ai`（方案 A 已完成）。
> 本清单用于把市场身份（Plugin ID 前缀）也换成公司组织 `origin-power-ai`。
> 关键机制：市场身份 = manifest 的 `author` + `name`，与 GitHub 仓库位置无关；`author` 必须与推送 PR 的 fork 所有者一致（安装时校验 `plugin_unique_identifier`）。

## 0. 前置条件

- [ ] `origin-power-ai` 组织已存在，且发布责任人对其有仓库创建/fork 权限
- [ ] 组织内确定「发布责任人」（持有 PAT、触发发布 workflow 的人/机器账号）
- [ ] 决定新 listing 的起始版本号（建议 `1.0.0` 表示新身份；若要连续则 `0.0.6`）
- [ ] 确认 `origin-power-ai` 作为 GitHub handle 满足市场校验：全小写、`^[a-z0-9_-]{1,64}$`、不含 `langgenius`/`dify`

## 1. 准备 fork（org 身份）

- [ ] 在 GitHub 网页上 Fork `langgenius/dify-plugins`，**owner 选择 `origin-power-ai`**
- [ ] 确认 fork 名称 `origin-power-ai/dify-plugins` 可访问
- [ ] 为发布用的 PAT 确认权限：能向该 org fork 推送分支并创建 PR（classic PAT 需 `repo`；org 若启用 SSO 需授权）

## 2. 更新源码（`origin-power-ai/dify-plugin-origin-power-ai`）

- [x] `manifest.yaml`：`author: "origin-power-ai"`（原 `ikki6666`）
- [x] `manifest.yaml`：`version: 1.0.0`
- [x] 确认 `repo:` 已是 `https://github.com/origin-power-ai/dify-plugin-origin-power-ai`
- [x] provider YAML/代码中无 author 字段需改（已核查）
- [x] 重新打包：`dify plugin package ./ -o origin_power_ai-1.0.0.difypkg`
- [x] 本地校验：包内 author=`origin-power-ai` / version=`1.0.0` / repo 正确；无 `.git`；
      文件清单与 v0.0.5 完全一致（另在 `.difyignore` 排除内部文档 `AGENTS.md`、`docs/`）

## 3. 提交市场 PR

- [ ] fork 的 `main` 同步到上游最新（避免「单文件 diff」被旧基线污染——见 v0.0.5 的教训）
- [ ] 放置文件：`origin-power-ai/origin_power_ai/origin_power_ai-<version>.difypkg`（**PR 必须只含这一个文件变更**）
- [ ] 提交 PR 到 `langgenius/dify-plugins`
- [ ] PR 描述使用官方模板，且：
  - [ ] Submission type 勾 **New plugin**（目录不同，不算 version update）
  - [ ] 补 `## Local validation` 与 `## Security and privacy notes` 两章节
  - [ ] 正文全英文、风险等级勾 Medium
  - [ ] 主动说明 supersedes `ikki6666/origin_power_ai`（避免审核问重复）
- [ ] 过 pre-check（外部 fork 的 workflow 需维护者批准；必要时走 Discussions 求助，参考本次记录）

## 4. 上线与旧条目处置

> 2026-09-14 官方答复（issue #3066，crazywoola）：不支持转移归属，按本清单「新 listing」路线执行；
> **旧 listing 由官方加 deprecated 提示并自动跳转到新 listing**，因此下面「在旧目录再发一个版本」
> 这一步**不需要我们做**。

- [ ] 合并后确认新 listing：`https://marketplace.dify.ai/plugins/origin-power-ai/origin_power_ai`
- [ ] 旧 listing（`ikki6666/origin_power_ai`）：等官方加弃用提示 + 跳转（无需我方发版）
- [ ] 通知已安装用户：Dify 不会自动迁移插件身份，需卸载旧插件、从新 listing 重新安装（凭证需重填）

## 5. CI / 发布流程

- [ ] 源码仓库（org 下）的 `PLUGIN_ACTION` secret 指向可推送 org fork 的 PAT
- [ ] workflow 中 `repository: ${{ steps.meta.outputs.author }}/dify-plugins` 会自动解析为 `origin-power-ai/dify-plugins` ✓
- [ ] 发版流程：改 manifest 版本 → 手动 Run workflow（v0.0.5 起已改为 `workflow_dispatch`，避免误触发）

## 6. 完成检查点

- [ ] 市场页作者显示为公司组织；仓库/文档/README 链接均指向 org
- [ ] 旧 listing 已带弃用说明，且不再发布新版本（避免同版本双发）
- [ ] 团队可独立发布（不依赖个人账号）
