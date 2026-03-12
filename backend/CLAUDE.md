# Backend — CLAUDE.md

> 後端開發指南，補充根目錄 CLAUDE.md 的後端細節。

---

## 目錄結構說明

```
backend/
├── src/
│   ├── config/
│   │   ├── database.ts      # Prisma client 初始化
│   │   ├── env.ts           # 環境變數驗證 (Zod)
│   │   └── swagger.ts       # Swagger 設定
│   ├── controllers/         # HTTP 層，只處理 req/res
│   ├── services/            # 業務邏輯層
│   ├── repositories/        # 資料存取層 (Prisma)
│   ├── routes/              # Express Router
│   ├── middlewares/
│   │   ├── auth.middleware.ts
│   │   ├── validate.middleware.ts
│   │   └── error.middleware.ts
│   ├── schemas/             # Zod 驗證 Schema
│   ├── utils/
│   │   ├── AppError.ts      # 自訂錯誤類別
│   │   ├── response.ts      # 統一回應工具
│   │   └── logger.ts        # Winston logger
│   └── app.ts               # Express 應用
├── prisma/
│   ├── schema.prisma
│   └── seed.ts              # 種子資料
├── tests/
├── .env.example
├── tsconfig.json
└── package.json
```

---

## 關鍵檔案範本

### `src/utils/AppError.ts`
```typescript
export class AppError extends Error {
  constructor(
    public code: string,
    public message: string,
    public statusCode: number = 500
  ) {
    super(message)
    this.name = 'AppError'
  }
}
```

### `src/middlewares/error.middleware.ts`
```typescript
import { Request, Response, NextFunction } from 'express'
import { AppError } from '../utils/AppError'
import { ZodError } from 'zod'

export const errorHandler = (
  err: Error,
  req: Request,
  res: Response,
  next: NextFunction
) => {
  if (err instanceof AppError) {
    return res.status(err.statusCode).json({
      success: false,
      error: err.code,
      message: err.message,
    })
  }

  if (err instanceof ZodError) {
    return res.status(400).json({
      success: false,
      error: 'VALIDATION_ERROR',
      message: err.errors[0].message,
      details: err.errors,
    })
  }

  console.error(err)
  res.status(500).json({
    success: false,
    error: 'INTERNAL_ERROR',
    message: '伺服器發生錯誤',
  })
}
```

### `src/middlewares/validate.middleware.ts`
```typescript
import { Request, Response, NextFunction } from 'express'
import { ZodSchema } from 'zod'

export const validate = (schema: ZodSchema) =>
  (req: Request, res: Response, next: NextFunction) => {
    try {
      req.body = schema.parse(req.body)
      next()
    } catch (error) {
      next(error)
    }
  }
```

---

## 新增 CRUD 模組的步驟

1. **Prisma Schema** → 在 `prisma/schema.prisma` 新增 Model
2. **執行 Migration** → `npx prisma migrate dev --name add_xxx`
3. **Zod Schema** → `src/schemas/xxx.schema.ts`
4. **Repository** → `src/repositories/xxx.repository.ts`
5. **Service** → `src/services/xxx.service.ts`
6. **Controller** → `src/controllers/xxx.controller.ts`
7. **Routes** → `src/routes/xxx.routes.ts`
8. **掛載路由** → 在 `src/app.ts` 中 import 並 `app.use()`
9. **Swagger 文檔** → 在 Controller 加 JSDoc 註解
10. **測試** → `tests/xxx.test.ts`

---

## 常用 Prisma 指令

```bash
npx prisma migrate dev --name <migration_name>   # 建立並執行 migration
npx prisma migrate reset                          # 重置資料庫（開發）
npx prisma generate                               # 重新生成 client
npx prisma studio                                 # 開啟資料庫 GUI
npx prisma db seed                                # 執行種子資料
npx prisma format                                 # 格式化 schema
```

---

## 環境變數清單

```env
NODE_ENV=development          # development | production | test
PORT=3001                     # 伺服器埠號
DATABASE_URL=                 # MySQL 連線字串
JWT_SECRET=                   # JWT 簽名密鑰（至少 32 字元）
JWT_EXPIRES_IN=7d             # JWT 有效期
CORS_ORIGIN=                  # 允許的前端來源
LOG_LEVEL=info                # error | warn | info | debug
```
