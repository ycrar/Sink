# 專案記憶與部署手冊 (Project Memory & Deployment Runbook)

> 本文件記錄了本專案（Sink 短網址服務）的所有雲端架構、Cloudflare 資源配置、CI/CD 構建設定與恢復指南。
> 任何 AI 助理（如 Antigravity、Claude Code、Cursor 等）或開發者只需讀取此文件，即可無縫恢復專案記憶並繼續維護或擴展。

---

## 1. 專案基本資訊 (Basic Information)
- **專案名稱**: Sink 短網址與分析服務 (Link Shortener with Analytics)
- **GitHub 專案庫**: [ycrar/Sink](https://github.com/ycrar/Sink)
- **Fork 來源**: [miantiao-me/Sink](https://github.com/miantiao-me/Sink)
- **官方文檔**: [Sink Docs - Cloudflare Workers 部署](https://docs.sink.cool/deployment/workers)
- **服務網址 (Live Custom Domain)**: [https://url.ycrar.cc.cd](https://url.ycrar.cc.cd)
- **管理後台 (Dashboard)**: [https://url.ycrar.cc.cd/dashboard](https://url.ycrar.cc.cd/dashboard)
- **Cloudflare 預設 Worker 網址**: `https://sink.n7701428-2c1.workers.dev`

---

## 2. Cloudflare 架構與資源清單 (Infrastructure & Resources)

| 資源類別 | 名稱 / 識別碼 | 說明 |
| :--- | :--- | :--- |
| **Cloudflare Account ID** | `2c199b2b3fb9378ccb9472d84a9134b5` | 綁定帳號 (`n7701428@gmail.com`) |
| **Worker Script Name** | `sink` | Cloudflare Workers 服務主體 |
| **Script Tag** | `db9bd71629754054b2e771db866a392f` | Cloudflare 內部 Worker 識別碼 |
| **D1 Database** | `sink` | 關聯式資料庫（儲存短網址與統計） |
| **D1 Database ID** | `d9e49165-8949-49cc-a0b1-44eb5c6c8621` | Binding: `DB` |
| **KV Namespace** | `sink` | 高速快取命名空間 |
| **KV Namespace ID** | `c1f539f28955400ca374beb876a37ef8` | Binding: `KV` |
| **Zone Domain** | `ycrar.cc.cd` | 託管於 Cloudflare 的主網域 |
| **Zone ID** | `3b5472881b92079338204b52885a7405` | `ycrar.cc.cd` 區域識別碼 |
| **Worker Custom Domain** | `url.ycrar.cc.cd` | 綁定至 Worker 的自訂短網址子網域 |
| **Domain Binding ID** | `e3c76dbbab93ad07aba6b6001128c3754f362e17` | 自訂網域路由設定 |

---

## 3. Workers Builds CI/CD 設定 (Build & Deploy Settings)

Cloudflare Workers Builds 已與 GitHub Repo `ycrar/Sink` 的 `master` 分支深度整合，每次推動至 `master` 分支均會自動觸發構建與部署。

### 構建參數 (Build Parameters)
- **構建命令 (Build Command)**: `npm run build && npm run deploy:worker`
- **目標分支 (Branch)**: `master`
- **Trigger UUID**: `2823eab1-b2cb-4ba6-b797-4ddf620489a7`

### 環境變數 (Environment Variables in Cloudflare Dashboard)
- `DEPLOY_D1_DATABASE_NAME`: `sink`
- `DEPLOY_D1_DATABASE_ID`: `d9e49165-8949-49cc-a0b1-44eb5c6c8621`
- `DEPLOY_KV_NAMESPACE_ID`: `c1f539f28955400ca374beb876a37ef8`
- `NUXT_CF_ACCOUNT_ID`: `2c199b2b3fb9378ccb9472d84a9134b5`

### 密鑰與安全變數 (Secrets)
- `NUXT_SITE_TOKEN`: 設定於 Cloudflare Workers 設定中（後台登入與 API 認證 Token）。
  - *如需查看或變更 Token，可前往 Cloudflare Dashboard -> Compute (Workers & Pages) -> Workers -> sink -> Settings -> Variables & Secrets 進行管理。*

---

## 4. 關鍵修復與程式調整 (Code Modifications Applied)

在適應 Cloudflare 免費方案 (Free Tier) 時，本專案已套用以下優化（詳見 Commit 歷史）：
1. **Analytics Engine 彈性適配**:
   - 檔案: `scripts/cloudflare-deploy-config.mjs`
   - 修改: 預設免費帳號未開啟 Analytics Engine 時會導致部署報錯 (Error 10089)。已改為只有在環境變數提供 `DEPLOY_ANALYTICS_DATASET` 時才綁定 Analytics Engine。
2. **Cron Triggers 限制避開**:
   - 檔案: `scripts/cloudflare-deploy-config.mjs`
   - 修改: Cloudflare 免費方案全帳號上限為 5 個 Cron 觸發器。Sink 預設排程備份到 R2。在未設置 `DEPLOY_R2_BUCKET_NAME` 時，構建腳本會自動移除 `config.triggers`，避免超過 5 個 Cron 上限 (Error 10072)。

---

## 5. 快速恢復與後續開發指南 (How to Restore & Resume)

### 給 AI 助理（Prompt 指引）：
當你需要在新的對話或工作區接手此專案時，可以直接發送以下指令給 AI：
> 「請讀取 GitHub `https://github.com/ycrar/Sink` 根目錄下的 `PROJECT_MEMORY.md`，並基於這份專案記憶與 Cloudflare 資源繼續進行開發/維護。」

### 本地開發 (Local Development)
```bash
git clone https://github.com/ycrar/Sink.git
cd Sink
pnpm install
pnpm dev
```

### 資料庫遷移 (Remote Database Migrations)
若未來新增了資料庫綱要修改：
```bash
pnpm db:migrate:remote
```

### 手動重新部署 (Manual Redeployment)
只要將代碼 push 到 `ycrar/Sink` 的 `master` 分支，Cloudflare Workers Builds 就會自動拉取最新程式並發布至 `https://url.ycrar.cc.cd`。
```bash
git add .
git commit -m "feat: your new feature"
git push origin master
```
