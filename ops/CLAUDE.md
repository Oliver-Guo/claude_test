# Ops — CLAUDE.md

> 環境與部署指南

---

## 檔案說明

```
ops/
├── Dockerfile.backend       # 後端 Docker 映像
├── Dockerfile.frontend      # 前端 Docker 映像（Nginx）
├── docker-compose.yml       # 開發環境
├── docker-compose.prod.yml  # 生產環境
└── nginx.conf               # Nginx 設定（前端 + API Proxy）
```

---

## Dockerfile.backend

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npx prisma generate
RUN npm run build

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/prisma ./prisma
EXPOSE 3001
CMD ["node", "dist/app.js"]
```

---

## Dockerfile.frontend

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY ../ops/nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
```

---

## docker-compose.yml（開發）

```yaml
version: '3.8'

services:
  mysql:
    image: mysql:8.0
    container_name: myapp-mysql
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: myapp
      MYSQL_CHARSET: utf8mb4
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      timeout: 20s
      retries: 10

  backend:
    build:
      context: ../backend
      dockerfile: ../ops/Dockerfile.backend
    container_name: myapp-backend
    restart: unless-stopped
    ports:
      - "3001:3001"
    environment:
      NODE_ENV: development
      DATABASE_URL: mysql://root:rootpass@mysql:3306/myapp
      JWT_SECRET: dev-secret-change-in-production
      CORS_ORIGIN: http://localhost:5173
    depends_on:
      mysql:
        condition: service_healthy
    volumes:
      - ../backend:/app
      - /app/node_modules

  frontend:
    build:
      context: ../frontend
      dockerfile: ../ops/Dockerfile.frontend
    container_name: myapp-frontend
    restart: unless-stopped
    ports:
      - "80:80"
    depends_on:
      - backend

volumes:
  mysql_data:
```

---

## nginx.conf

```nginx
server {
    listen 80;
    root /usr/share/nginx/html;
    index index.html;

    # SPA fallback
    location / {
        try_files $uri $uri/ /index.html;
    }

    # API Proxy
    location /api/ {
        proxy_pass http://backend:3001;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # Gzip
    gzip on;
    gzip_types text/plain application/javascript text/css application/json;
}
```

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
