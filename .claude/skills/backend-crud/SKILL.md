---
name: backend-crud
description: 產生完整的後端 CRUD 模組（Controller/Service/Repository/Routes/Schema）。當使用者說「新增 xxx 模組」、「幫我建立 xxx CRUD」時使用。
argument-hint: "ResourceName"
---

為資源 `$ARGUMENTS` 產生完整的 CRUD 模組，遵循專案三層架構規範。

## 執行步驟

### 1. 推導命名

從 `$ARGUMENTS`（PascalCase，例如 `Product`）推導：
- 小寫單數（kebab-case 檔案名稱）：`product`
- 小寫複數（URL path + DB 表名）：`products`
- 大寫底線（錯誤代碼前綴）：`PRODUCT`

### 2. 更新 Prisma Schema
在 `prisma/schema.prisma` 新增 Model，參考現有 Model 的欄位慣例（id、createdAt、updatedAt、@@map 用小寫複數表名）。

### 3. 建立 Zod Schema
檔案：`src/schemas/<小寫單數>.schema.ts`
定義 `create` 和 `update`（partial）Schema，欄位根據 Prisma Model 決定。

### 4. 依序建立以下檔案

| 檔案路徑（使用小寫單數） | 職責 |
|------------------------|------|
| `src/repositories/<小寫單數>.repository.ts` | 只呼叫 Prisma，不含業務邏輯 |
| `src/services/<小寫單數>.service.ts` | 業務邏輯，找不到時 throw AppError |
| `src/controllers/<小寫單數>.controller.ts` | 只處理 req/res，catch 後 next(error) |
| `src/routes/<小寫單數>.routes.ts` | 掛 validate middleware |

### 5. 掛載路由
在 `src/app.ts` import 並加入 `app.use('/api/v1/<小寫複數>', ...)`

### 6. 提示後續動作
完成後告知使用者：
- 執行 `npx prisma migrate dev --name add_<小寫單數>`
- 執行 `/backend-test` 補充測試
- 在 Controller 補 Swagger JSDoc 註解

## 架構限制（必須遵守）
- Repository 禁止業務邏輯
- Controller 禁止直接呼叫 Prisma
- 所有錯誤透過 `throw new AppError(code, message, statusCode)` 拋出
- 所有 TypeScript 型別明確定義，禁止 `any`
