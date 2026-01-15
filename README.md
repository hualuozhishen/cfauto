<img width="1920" height="879" alt="image" src="https://github.com/user-attachments/assets/fef7e297-e455-41f9-b716-190ee68a59c7" />
---

# 🚀 Cloudflare Worker 部署中控

Cloudflare Worker部署管理工具，管理和维护 **CMliu (EdgeTunnel)** 与 **Joey (CFNew)** 两个项目。

它集成了**多账号管理**、**全局流量熔断**、**自动版本更新**、**双项目并行管理**以及**变量动态配置**等功能，拥有一个美观且功能强大的 Web 控制面板。

## ✨ 核心功能

* **🛡️ 全局流量熔断 (Global Fuse)**：
* 监控 Cloudflare 账号的总流量（Workers + Pages）。
* 当流量使用百分比超过设定阈值时，**自动同时轮换**该账号下所有项目的 UUID 并重新部署，防止被封号。


* **🔄 自动更新检测**：
* 定时检查 GitHub 上游仓库的最新 Commit。
* 发现新版本自动拉取代码并重新部署，保持项目最新。


* **📊 智能统计面板**：
* 直观展示各账号的今日流量使用情况。
* 自动按流量使用量**降序排列**，一眼识别流量大户。


* **⚡ 双项目并行管理**：
* 无需切换页面，同屏管理 CMliu 和 Joey 两个项目的变量与部署状态。


* **⚙️ 动态变量管理**：
* 支持在 UI 上直接添加、修改、删除 Worker 环境变量（如 UUID, PROXYIP 等）。
* 一键生成新的随机 UUID。



---

## 🛠️ 部署指南

### 1. 准备工作

* 一个 **Cloudflare** 账号。
* (可选) 一个 **GitHub Token**（用于提高 API 请求限额，避免检测更新失败）。

### 2. 创建 Cloudflare Worker

1. 登录 Cloudflare Dashboard，进入 **Workers & Pages**。
2. 点击 **Create Application** -> **Create Worker**。
3. 命名为 `worker-manager` (或任意名称)，点击 **Deploy**。
4. 点击 **Edit code**，将提供的 `worker.js` (V5.4) 完整代码粘贴进去，**Save and Deploy**。

### 3. 配置 KV Namespace (数据库)

此脚本需要 KV 来存储账号信息和配置。

1. 在 Cloudflare Dashboard 左侧菜单找到 **KV**。
2. 点击 **Create a Namespace**，命名为 `CONFIG_KV` (或其他名字，建议一致)。
3. 回到你的 Worker 设置页面 -> **Settings** -> **Variables**。
4. 在 **KV Namespace Bindings** 区域：
* **Variable name**: 必须填 `CONFIG_KV` (代码中写死的变量名)。
* **KV Namespace**: 选择你刚才创建的那个。


5. 点击 **Save and deploy**。

### 4. 设置环境变量

在 Worker 的 **Settings** -> **Variables** -> **Environment Variables** 中添加：

| 变量名 | 必填 | 说明 |
| --- | --- | --- |
| `ACCESS_CODE` | ✅ 是 | **访问密码**。用于登录中控面板，防止他人访问。 |
| `GITHUB_TOKEN` | ❌ 否 | **GitHub 个人访问令牌**。如果你的 Worker 频繁检测更新报错，建议配置此项以提高 API 限额。权限只需 `public_repo` 即可。 |

### 5. 设置触发器 (Cron Triggers)

为了让自动熔断和自动更新功能生效，必须设置定时任务。

1. 进入 Worker 的 **Settings** -> **Triggers**。
2. 点击 **Add Cron Trigger**。
3. **Cron Schedule**: 建议设置为 `*/30 * * * *` (每30分钟执行一次)。
* *注意：具体的检测频率逻辑（如每小时还是每分钟）是在 Web 面板的“自动检测设置”中配置的，这里的 Cron 只是为了唤醒 Worker 去检查配置。*



---

## 📖 使用教程

### 1. 登录面板

访问你的 Worker URL (例如 `https://your-worker.workers.dev`)，输入你在环境变量中设置的 `ACCESS_CODE` 登录。

### 2. 添加 Cloudflare 账号

在左侧 **“📡 账号管理”** 面板中：

1. 点击 **“➕ 添加账号”**。
2. 填写信息：
* **备注**: 给账号起个名字（如：主力号、备用号）。
* **Account ID**: 在 Cloudflare 首页域名右下角可以找到。
* **API Token**: **必须申请一个具有以下权限的 Token**：
* `Account` - `Account Analytics` - `Read` (读取流量统计)
* `Account` - `Worker Scripts` - `Edit` (修改/部署 Worker)


* **Worker 分配**: 填写该账号下已经创建好的 Worker 名称。
* *🔴 CMliu Workers*: 填部署了 EdgeTunnel 的 Worker 名。
* *🔵 Joey Workers*: 填部署了 Joey 项目的 Worker 名。
* *(多个 Worker 用逗号 `,` 隔开)*




3. 点击 **保存账号**。

### 3. 配置自动熔断与更新

在页面顶部的 **配置栏** 中：

1. 勾选 **“自动检测”** 开关。
2. 设置 **频率** (建议与 Cron Trigger 保持一致或倍数关系，如 30 分钟)。
3. 设置 **熔断阈值%**：
* 例如设置为 `90`。
* 当账号总流量超过 90% (即 90,000 请求，免费版限额 10w) 时，系统会自动更换所有关联 Worker 的 UUID 并重新部署。
* 设置为 `0` 则关闭熔断功能。


4. 点击 **“保存设置”**。

### 4. 变量管理与手动部署

在右侧的项目面板中：

* **添加变量**: 点击虚线框 **“➕ 添加自定义变量”**，输入 Key (如 `TOKEN`) 和 Value。
* **修改变量**: 直接在输入框修改。
* **刷新 UUID**: 点击 **“🎲 刷 UUID”** 会随机生成新 ID（仅修改前端显示，需点击部署才生效）。
* **部署**: 点击 **“🚀 部署 ...”**，脚本会自动保存变量并上传代码到 Cloudflare。

---

## ⚠️ 注意事项

1. **权限问题**：
* 提供的 Cloudflare API Token 必须拥有 **编辑 Worker** 和 **读取账号分析(Analytics)** 的权限，否则无法统计流量或部署失败。


2. **流量统计延迟**：
* Cloudflare 的 GraphQL 流量统计可能有 5-15 分钟的延迟，熔断并非实时精确到秒，建议阈值不要设置得太极限（如建议 90% 甚至 80%）。


3. **项目名称匹配**：
* 在添加账号时，填写的 Worker 名称必须与你在 Cloudflare 后台实际创建的 Worker 名称**完全一致**，否则会提示找不到脚本。


4. **覆盖风险**：
* 部署功能会**覆盖**目标 Worker 的现有代码。请确保你只想运行 CMliu 或 Joey 的代码。


5. **GitHub API 限制**：
* 如果不配置 `GITHUB_TOKEN`，GitHub API 限制每小时 60 次请求。如果是多账号高频检测，可能会导致检测更新失败（面板显示 "Status Err"）。



---

## ❓ 常见问题

**Q: 为什么状态显示 "Status Err"?**
A: 通常是因为 GitHub API 请求受限。请在 Worker 环境变量中添加 `GITHUB_TOKEN`，或者稍后再试。

**Q: 熔断触发了，但是 Worker 没变？**
A: 请检查 Cron 是否配置正确，以及 API Token 是否有权限编辑 Worker Scripts。

**Q: 修改了变量没有生效？**
A: 修改变量后必须点击 **“🚀 部署”** 按钮，系统才会将新变量应用到 Cloudflare Worker 中。
