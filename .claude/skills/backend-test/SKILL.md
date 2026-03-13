---
name: backend-test
description: 為後端 API 端點產生測試檔案。當使用者說「寫測試」、「新增測試」或「幫我測 xxx API」時使用。
argument-hint: "resource-name"
---

為 `$ARGUMENTS` 撰寫後端整合測試。

## 執行步驟

1. 從 `$ARGUMENTS` 推導命名：小寫單數作為檔案名稱與 URL path（例如 `User` → `user`），大寫底線作為錯誤代碼前綴（例如 `User` → `USER`）
2. 讀取 `src/controllers/$ARGUMENTS（小寫）.controller.ts` 了解所有端點
3. 讀取 `src/schemas/$ARGUMENTS（小寫）.schema.ts` 了解驗證規則
4. 在 `tests/$ARGUMENTS（小寫）.test.ts` 建立測試檔案，涵蓋：
   - 每個端點的成功情境
   - 驗證失敗（400）
   - 資源不存在（404）
   - 需要認證的端點加上 401 測試

## 測試規範

- 使用 Vitest + Supertest，不 mock 資料庫
- 每個 `describe` 對應一個端點（`METHOD /api/v1/resources`）
- 測試名稱用英文，清楚描述情境
- 需要認證時在 `beforeAll` 取得 token

## 範本

```typescript
import { describe, it, expect, beforeAll } from 'vitest'
import request from 'supertest'
import app from '../src/app'

// 以下用 user 為例，實際依 $ARGUMENTS 推導
describe('GET /api/v1/users/:id', () => {
  it('should return item when exists', async () => {
    const res = await request(app).get('/api/v1/users/1')
    expect(res.status).toBe(200)
    expect(res.body.success).toBe(true)
    expect(res.body.data).toHaveProperty('id')
  })

  it('should return 404 when not found', async () => {
    const res = await request(app).get('/api/v1/users/99999')
    expect(res.status).toBe(404)
    expect(res.body.error).toBe('USER_NOT_FOUND')
  })
})
```

5. 完成後輸出測試執行指令：`npx vitest run tests/<小寫資源名>.test.ts`
