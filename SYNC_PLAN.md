# 记账小管家 · 跨设备同步 方案（待确认）

> 2026-09-10 | 目标：两台设备（手机/电脑）共用一个账本

## 现状
- 纯前端单 `index.html`，数据全在 `localStorage['acc_data']`
- 无任何云端代码 → 每台设备一个独立账本，天然不同步
- 页面本身可正常打开使用（已确认）

## 方案 A（推荐）：GitHub Gist 同步
- 用主人的 GitHub 细粒度 token（只勾 Gists: Read and write）
- 数据存成一个 secret gist，文件名 `accounting-data.json`
- 每台设备只需粘贴一次 token；App 自动查找/创建该 gist（无需手抄 ID）
- 无服务器、免费、秒级同步
- 缺点：token 存在浏览器 localStorage（自用可接受）；`api.github.com` 在国内偶有抽风

## 方案 B：自建小后端（Render）
- 仿 killer-boss-game，Flask + 文件存储，用「同步码」分区
- 优点：不需要 GitHub token，设备只需输一个同步码
- 缺点：要写 + 部署后端；Render 免费版会休眠（首次同步可能等 30~60s）

## 改动文件
- `记账小程序/index.html`（唯一改动）
- 改完 push 到 `mitanguse/accounting-app` → GitHub Pages 自动上线

## 实现步骤
1. 数据层加内部字段 `meta`：`updatedAt / recordsUpdated / rulesUpdated / budgetUpdated / deleted{}`
2. 新增记录打 `_u` 更新时间；删除记录写墓碑（防止被另一台设备"复活"）
3. 写合并算法 `mergeData(local, remote)`：records 按 id 取新、墓碑取最大、rules/budget 按更新时间取新
4. Gist API：GET 列表 / GET 内容 / POST 创建 / PATCH 更新
5. 设置页加「☁️ 云同步」区块：token 输入、自动同步开关、立即同步按钮、状态+最后同步时间
6. 触发时机：打开时拉取合并；改动后 3 秒防抖推送；切回前台时拉取
7. 综合验证：`node --check` + 本地双标签模拟

## 风险 / 待确认
- ⚠️ 选 A 还是 B？（蜜糖推荐 A）
- token 存浏览器本地，自用可否？
- 同步逻辑为"合并保底不丢数据"，冲突时按时间取新（不做花哨的冲突提示）
- 会往数据里加隐藏字段（界面不可见）
