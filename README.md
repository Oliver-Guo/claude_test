# Blog 網站

全端 Blog 系統，包含前台（文章瀏覽、會員留言）與後台（管理員 CRUD）。

## 技術棧

| 層級 | 技術 |
|------|------|
| 後端 | Node.js 20 + Express + TypeScript + Prisma |
| 資料庫 | MySQL 8.0 |
| 前端 | React 18 + Vite + TypeScript + shadcn/ui |
| 狀態管理 | Zustand + TanStack Query v5 |
| 部署 | Docker + Nginx |

---

## 🚀 本地開發啟動

### 前置需求

- Node.js 20+
- Docker（用於 MySQL）

### 1. 啟動 MySQL

```bash
cd ops
docker-compose up -d mysql
```

等待 MySQL 健康檢查通過（約 30 秒）：

```bash
docker-compose ps  # 確認 mysql Status 為 healthy
```

### 2. 後端初始化

```bash
cd backend

# 安裝依賴
npm install

# 建立 .env（首次需要）
cp .env.example .env

# 執行資料庫 Migration
npx prisma migrate dev --name init

# 建立初始資料（admin 帳號 + 示範分類 + 文章）
npm run seed

# 啟動開發伺服器（port 3001）
npm run dev
```

### 3. 前端啟動

```bash
cd frontend

# 安裝依賴
npm install

# 啟動開發伺服器（port 5173）
npm run dev
```

### 4. 開啟瀏覽器

| 服務 | 網址 |
|------|------|
| 前台首頁 | http://localhost:5173 |
| 管理後台 | http://localhost:5173/admin |
| API 文件 (Swagger) | http://localhost:3001/api/docs |

---

## 🔑 預設帳號

| 角色 | Email | 密碼 |
|------|-------|------|
| 管理員 | `admin@admin.com` | `admin5678` |

> 一般會員需透過 `/register` 頁面自行註冊

---

## 🌐 頁面路由

### 前台（公開）

| 頁面 | 路徑 | 說明 |
|------|------|------|
| 文章列表 | `/` | 支援分類篩選、分頁 |
| 文章內頁 | `/posts/:slug` | 顯示內文與留言 |
| 登入 | `/login` | 會員 / 管理員共用 |
| 註冊 | `/register` | 填寫 Email、密碼、暱稱 |

### 後台（需管理員登入）

| 頁面 | 路徑 | 說明 |
|------|------|------|
| 總覽 | `/admin` | 統計資料 |
| 文章管理 | `/admin/posts` | 列表、新增、編輯、刪除 |
| 新增文章 | `/admin/posts/create` | 填寫標題、內容、分類 |
| 編輯文章 | `/admin/posts/:id/edit` | 修改文章 |
| 分類管理 | `/admin/categories` | 新增、編輯、刪除分類 |
| 留言管理 | `/admin/comments` | 查看並刪除留言 |

---

## 🐳 Docker 一鍵啟動（正式環境）

```bash
cd ops

# 啟動所有服務（MySQL + 後端 + 前端）
docker-compose up -d

# 查看狀態
docker-compose ps

# 查看後端日誌
docker-compose logs -f backend
```

| 服務 | 網址 |
|------|------|
| 前台 | http://localhost |
| 後台 | http://localhost/admin |

### 停止服務

```bash
cd ops && docker-compose down

# 同時刪除資料庫資料
docker-compose down -v
```

---

## 📡 API 端點

### Auth

| 方法 | 路徑 | 說明 |
|------|------|------|
| POST | `/api/v1/auth/register` | 會員註冊 |
| POST | `/api/v1/auth/login` | 登入（含管理員）|
| GET | `/api/v1/auth/me` | 取得登入者資料 |

### Posts

| 方法 | 路徑 | 權限 | 說明 |
|------|------|------|------|
| GET | `/api/v1/posts` | 公開 | 已發布文章列表（分頁）|
| GET | `/api/v1/posts/:slug` | 公開 | 單篇文章（含留言）|
| GET | `/api/v1/posts/admin/all` | ADMIN | 所有文章（含草稿）|
| POST | `/api/v1/posts` | ADMIN | 建立文章 |
| PATCH | `/api/v1/posts/:id` | ADMIN | 更新文章 |
| DELETE | `/api/v1/posts/:id` | ADMIN | 刪除文章 |

### Categories

| 方法 | 路徑 | 權限 |
|------|------|------|
| GET | `/api/v1/categories` | 公開 |
| POST | `/api/v1/categories` | ADMIN |
| PATCH | `/api/v1/categories/:id` | ADMIN |
| DELETE | `/api/v1/categories/:id` | ADMIN |

### Comments

| 方法 | 路徑 | 權限 |
|------|------|------|
| POST | `/api/v1/posts/:postId/comments` | 已登入 |
| GET | `/api/v1/comments` | ADMIN |
| DELETE | `/api/v1/comments/:id` | ADMIN |

---

## 📁 專案結構

```
project/
├── backend/
│   ├── prisma/
│   │   ├── schema.prisma      # 資料庫模型
│   │   └── seed.ts            # 初始資料
│   ├── src/
│   │   ├── config/            # 環境變數、DB、Swagger
│   │   ├── controllers/       # HTTP 處理層
│   │   ├── services/          # 業務邏輯層
│   │   ├── repositories/      # 資料存取層
│   │   ├── routes/            # 路由設定
│   │   ├── middlewares/       # JWT 驗證、錯誤處理
│   │   ├── schemas/           # Zod 驗證 Schema
│   │   └── app.ts             # Express 入口
│   ├── .env.example
│   └── package.json
├── frontend/
│   ├── src/
│   │   ├── api/               # API 呼叫層
│   │   ├── components/        # UI 元件（shadcn + shared）
│   │   ├── hooks/             # Custom hooks
│   │   ├── pages/             # 頁面元件
│   │   │   ├── public/        # 前台頁面
│   │   │   └── admin/         # 後台頁面
│   │   ├── stores/            # Zustand 狀態
│   │   ├── types/             # TypeScript 型別
│   │   └── main.tsx           # 路由入口
│   └── package.json
└── ops/
    ├── docker-compose.yml
    ├── Dockerfile.backend
    ├── Dockerfile.frontend
    └── nginx.conf
```

---

## ⚙️ 環境變數

### Backend `.env`

```bash
cp backend/.env.example backend/.env
```

### Frontend `.env`

```bash
cp frontend/.env.example frontend/.env
```

---

## 🔧 常用指令

```bash
# 後端
npm run dev              # 開發模式（hot reload）
npm run build            # 編譯 TypeScript
npm run seed             # 執行 Seed 資料
npx prisma studio        # 開啟 Prisma GUI

# 前端
npm run dev              # 開發模式
npm run build            # 正式編譯

# Docker
docker-compose up -d mysql          # 只啟動 MySQL
docker-compose up -d                # 啟動所有服務
docker-compose logs -f backend      # 查看後端日誌
docker-compose exec backend sh      # 進入後端容器
```
