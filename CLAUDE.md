# CLAUDE.md — 專案 AI 開發指南

> 本文件是 Claude AI 開發本專案的核心說明文件。
> 每次開始新的開發任務前，請先閱讀此文件。
> 各子目錄有對應的 CLAUDE.md，包含更詳細的實作細節。

---

## 📁 專案結構

```
project/
├── backend/          # Node.js + Express + TypeScript + Prisma
├── frontend/         # React 18 + Vite + TypeScript + shadcn/ui
├── docs/             # VitePress 文檔
└── ops/              # Docker / Nginx / CI 部署設定
```

---

## 🧠 開發原則（Claude 必讀）

1. **型別優先**：所有變數、函式參數、回傳值必須有 TypeScript 型別，禁止使用 `any`
2. **Schema 驅動**：資料結構以 Prisma Schema 為唯一真相來源，Zod Schema 對應同步
3. **三層架構**：後端嚴格遵守 Controller → Service → Repository 分層，禁止跨層直接呼叫
4. **單一職責**：每個檔案只做一件事，超過 200 行考慮拆分
5. **錯誤處理**：所有 async 函式必須有 try/catch，統一透過 errorHandler middleware 回應
6. **環境變數**：所有敏感資訊從 `.env` 讀取，禁止硬編碼在代碼中

---

## ⚙️ 技術棧總覽

### Backend
| 項目 | 技術 | 版本 |
|------|------|------|
| Runtime | Node.js | 20 LTS |
| 框架 | Express | ^4.18 |
| 語言 | TypeScript | ^5.3 |
| ORM | Prisma | ^5.x |
| 資料庫 | MySQL | 8.0 |
| 驗證 | Zod | ^3.x |
| 認證 | JWT + bcrypt | - |
| 測試 | Vitest + Supertest | - |
| 日誌 | Winston + Morgan | - |
| API 文檔 | Swagger UI (OpenAPI 3.0) | - |

### Frontend
| 項目 | 技術 | 版本 |
|------|------|------|
| 框架 | React | 18 |
| 建構工具 | Vite | ^5.x |
| 語言 | TypeScript | ^5.3 |
| UI 框架 | shadcn/ui + Tailwind CSS | - |
| 狀態管理 | Zustand | ^4.x |
| 資料請求 | TanStack Query v5 | - |
| 路由 | React Router v6 | - |
| 表單 | React Hook Form + Zod | - |
| HTTP | Axios | ^1.x |
| 圖示 | Lucide React | - |

---

## 📏 命名規範

### 檔案命名
```
# Backend（kebab-case + 層級後綴）
user.controller.ts / user.service.ts / user.repository.ts
user.routes.ts / user.schema.ts / auth.middleware.ts

# Frontend
UserCard.tsx       # PascalCase（React 元件）
useAuth.ts         # camelCase + use 前綴（hooks）
userStore.ts       # camelCase + Store 後綴
user.api.ts        # kebab-case（API 呼叫）
```

### 變數與函式
- 一般變數/函式 → `camelCase`（`getUserById`、`isLoading`）
- 常數 → `UPPER_SNAKE_CASE`（`MAX_RETRY_COUNT`、`API_BASE_URL`）
- React 元件 → `PascalCase`（`UserProfileCard`）
- TypeScript 型別/介面/enum → `PascalCase`（`UserResponse`、`UserRole`）

---

## 🌐 API 設計規範

### URL 結構
```
GET    /api/v1/users              # 列表（支援分頁 ?page=1&limit=20）
GET    /api/v1/users/:id          # 單筆
POST   /api/v1/users              # 建立
PATCH  /api/v1/users/:id          # 部分更新
DELETE /api/v1/users/:id          # 刪除
GET    /api/v1/users/:id/orders   # 關聯資源
```

### 統一回應格式
```typescript
// 成功
{ "success": true, "data": { ... }, "message": "操作成功" }

// 分頁
{ "success": true, "data": [ ... ], "pagination": { "page": 1, "limit": 20, "total": 100, "totalPages": 5 } }

// 錯誤
{ "success": false, "error": "RESOURCE_NOT_FOUND", "message": "用戶不存在" }
```

### HTTP 狀態碼
| 情況 | 狀態碼 |
|------|--------|
| 查詢成功 | 200 |
| 建立成功 | 201 |
| 無內容 | 204 |
| 驗證失敗 | 400 |
| 未授權 | 401 |
| 無權限 | 403 |
| 資源不存在 | 404 |
| 伺服器錯誤 | 500 |

---

## 📝 Git Commit 規範

遵循 [Conventional Commits](https://www.conventionalcommits.org/)，格式：`type(scope): 描述`

| 類型 | 用途 |
|------|------|
| feat | 新功能 |
| fix | 修復 bug |
| docs | 文檔更新 |
| refactor | 重構（不影響功能） |
| test | 測試相關 |
| chore | 建置/工具/依賴 |
| style | 樣式/格式（不影響邏輯） |

範例：`feat(user): 新增用戶列表 API`

---

## ⚠️ Claude 開發注意事項

1. **修改 Prisma Schema 後**，必須執行 `npx prisma migrate dev` 並更新對應的 Zod Schema
2. **新增 API 端點後**，必須在 Swagger 文件中補充對應的 JSDoc 註解
3. **新增環境變數後**，必須同步更新 `.env.example`
4. **前端新增 API 呼叫**，統一在 `src/api/` 目錄下建立對應檔案，不在元件內直接呼叫 axios
5. **禁止在 Repository 層寫業務邏輯**，禁止在 Controller 層直接呼叫 Prisma
6. **所有錯誤**透過 `throw new AppError(code, message, statusCode)` 拋出，由 errorHandler 統一處理
