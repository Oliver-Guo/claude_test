# Ops — CLAUDE.md

> 環境與部署指南。設定檔內容請直接查看對應檔案。

---

## 檔案說明

| 檔案 | 說明 |
|------|------|
| `Dockerfile.backend` | 後端 Docker 映像（multi-stage build，Node 20 Alpine） |
| `Dockerfile.frontend` | 前端 Docker 映像（Vite build → Nginx Alpine） |
| `docker-compose.yml` | 開發環境（MySQL + Backend + Frontend） |
| `docker-compose.prod.yml` | 生產環境 |
| `nginx.conf` | Nginx 設定（SPA fallback + API Proxy + Gzip） |

---

## 常用指令

```bash
# 開發環境
docker-compose up -d                              # 啟動全部
docker-compose up -d --build backend             # 重建後端
docker-compose down                               # 停止全部
docker-compose logs -f backend                   # 查看後端日誌

# 資料庫操作
docker-compose exec backend npx prisma migrate dev
docker-compose exec backend npx prisma studio
docker-compose exec mysql mysql -u root -prootpass myapp

# 生產部署
docker-compose -f docker-compose.prod.yml up -d
```

---

## 環境說明

| 服務 | 開發 Port | 說明 |
|------|-----------|------|
| MySQL | 3306 | 資料庫 |
| Backend | 3001 | Express API |
| Frontend | 5173 (dev) / 80 (docker) | React 應用 |
| Docs | 5174 | VitePress |
