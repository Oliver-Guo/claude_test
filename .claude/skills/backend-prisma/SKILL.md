---
name: backend-prisma
description: 協助執行 Prisma 相關操作，包含 migration、schema 修改、seed、查詢優化。當使用者說「跑 migration」、「幫我寫 Prisma 查詢」、「更新 schema」時使用。
argument-hint: "migrate | seed | query | reset"
---

根據 `$ARGUMENTS` 協助處理 Prisma 相關任務。若未指定則分析當前需求決定動作。

## 可處理的任務

### `migrate`
1. 讀取 `prisma/schema.prisma` 確認最新異動
2. 提示執行：`npx prisma migrate dev --name <描述性名稱>`
3. 提醒執行後需同步更新對應的 Zod Schema

### `seed`
1. 讀取 `prisma/seed.ts`（若存在）
2. 確認 seed 資料合理，必要時協助補充
3. 提示執行：`npx prisma db seed`

### `query`
根據使用者描述的需求，產生符合以下規範的 Prisma 查詢程式碼：
- 分頁：`findMany` + `count` 搭配 `Promise.all`
- 關聯：使用 `include` 或 `select`，避免 N+1
- 條件：`where` 物件，不拼接字串
- 事務：`prisma.$transaction([...])`

### `reset`
警告使用者這會清空資料庫，確認後提示：`npx prisma migrate reset`

## 常用指令參考

```bash
npx prisma migrate dev --name <name>  # 建立並執行 migration
npx prisma migrate deploy             # 生產環境執行 pending migrations
npx prisma generate                   # 重新生成 Client（schema 變更後必跑）
npx prisma studio                     # 開啟 GUI（localhost:5555）
npx prisma format                     # 格式化 schema.prisma
npx prisma validate                   # 驗證 schema 語法
```

## 注意事項
- 修改 Prisma Schema 後必須執行 `prisma generate` 才能在程式碼中使用新型別
- `db push` 僅用於原型開發，正式開發一律使用 `migrate dev`
- 生產環境不執行 `migrate dev`，改用 `migrate deploy`
