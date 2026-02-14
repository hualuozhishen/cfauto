# 在管理 Worker 界面添加修改子域名功能

在"📂 管理账号"弹窗的 Worker 列表中，为每个 Worker 增加子域名（`*.workers.dev`）的启用/禁用切换功能，以及显示当前子域名状态。

## 核心思路

利用 Cloudflare 现有 API：
- `GET /accounts/{id}/workers/scripts/{name}/subdomain` — 查询 Worker 子域名状态
- `POST /accounts/{id}/workers/scripts/{name}/subdomain` — 启用/禁用子域名
- `GET /accounts/{id}/workers/subdomain` — 获取账号的 subdomain 前缀

批量部署中已有类似逻辑（worker.js 第 465-477 行），本次为"管理 Worker"弹窗复用同一 API。

## 变更方案

### 后端

#### [MODIFY] [worker.js](file:///d:/DeskTop/GitHub/worker部署中控/worker.js)

**1. 新增 API 路由** `POST /api/toggle_subdomain`（约第 187 行后插入）

```js
if (url.pathname === "/api/toggle_subdomain" && request.method === "POST") {
    const { accountId, email, globalKey, workerName, enabled } = await request.json();
    return await handleToggleSubdomain(accountId, email, globalKey, workerName, enabled);
}
```

**2. 新增 `handleToggleSubdomain` 后端函数**（在 `handleDeleteWorker` 后面）

- 调用 `POST .../workers/scripts/{name}/subdomain` 设置 `{enabled: true/false}`
- 若启用成功，再查询 `GET .../workers/subdomain` 获取前缀，返回完整域名
- 返回 `{ success, domain?, msg? }`

**3. 扩展 `handleGetAllWorkers`**（第 647-660 行）

- 对每个 Worker 额外查询子域名状态 `GET /accounts/{id}/workers/scripts/{name}/subdomain`
- 查询账号 subdomain 前缀 `GET /accounts/{id}/workers/subdomain`
- 在返回数据中增加 `subdomain_enabled` 和 `subdomain_url` 字段

### 前端

#### [MODIFY] [worker.js](file:///d:/DeskTop/GitHub/worker部署中控/worker.js)

**4. 扩展管理弹窗表头**（第 1000 行附近）

- 在 "操作" 列前增加 "子域名" 列

**5. 修改 `openAccountManage` 函数**（第 1245-1289 行）

- 在 Worker 列表每行增加子域名状态展示 + 开关按钮
- 启用状态：显示完整 `https://{worker}.{prefix}.workers.dev` 链接 + 🚫禁用按钮
- 禁用状态：显示"已禁用" + ✅启用按钮

**6. 新增 `toggleWorkerSubdomain` 前端函数**

- 调用 `/api/toggle_subdomain`
- 成功后刷新管理弹窗

## 验证方案

### 手动验证

> [!IMPORTANT]
> 本项目为 Cloudflare Worker 单文件部署，无自动化测试框架。需要用户手动部署后验证。

1. 将修改后的 `worker.js` 部署到 Cloudflare Worker
2. 进入中控面板 → 账号列表 → 点击"📂 管理"
3. 确认 Worker 列表中新增"子域名"列，显示当前状态
4. 点击启用/禁用按钮，确认 Cloudflare API 响应正确
5. 刷新页面后确认状态持久化
