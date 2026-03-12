# Frontend — CLAUDE.md

> 前端開發指南，補充根目錄 CLAUDE.md 的前端細節。

---

## 目錄結構說明

```
frontend/src/
├── api/                 # API 呼叫函式（axios）
│   ├── user.api.ts
│   └── index.ts         # 統一匯出
├── components/
│   ├── ui/              # shadcn/ui 元件（從 shadcn CLI 生成）
│   └── shared/          # 跨頁面共用業務元件
├── hooks/               # 自訂 React Hooks
│   ├── useAuth.ts
│   └── useDebounce.ts
├── pages/               # 頁面元件（對應路由）
│   ├── HomePage.tsx
│   ├── LoginPage.tsx
│   └── users/
│       ├── UserListPage.tsx
│       └── UserDetailPage.tsx
├── stores/              # Zustand 狀態
│   └── authStore.ts
├── types/               # TypeScript 型別定義
│   ├── user.ts
│   └── api.ts           # API 回應共用型別
├── lib/
│   ├── axios.ts          # Axios 實例設定
│   ├── queryClient.ts    # TanStack Query 設定
│   └── utils.ts          # shadcn/ui cn() 工具
└── main.tsx
```

---

## 路由設定範例

```tsx
// src/main.tsx
import { BrowserRouter, Routes, Route } from 'react-router-dom'
import { QueryClientProvider } from '@tanstack/react-query'
import { queryClient } from './lib/queryClient'

// 受保護路由
const ProtectedRoute = ({ children }) => {
  const { token } = useAuthStore()
  if (!token) return <Navigate to="/login" />
  return children
}
```

---

## shadcn/ui 安裝元件

```bash
# 安裝常用元件
npx shadcn-ui@latest add button
npx shadcn-ui@latest add input
npx shadcn-ui@latest add card
npx shadcn-ui@latest add table
npx shadcn-ui@latest add dialog
npx shadcn-ui@latest add form
npx shadcn-ui@latest add toast
npx shadcn-ui@latest add badge
npx shadcn-ui@latest add dropdown-menu
npx shadcn-ui@latest add select
```

---

## TanStack Query 使用規範

```typescript
// ✅ Query Key 命名規範
['users']                        // 列表
['users', userId]                // 單筆
['users', userId, 'orders']      // 關聯資源

// ✅ 標準 useQuery
const { data, isLoading, error } = useQuery({
  queryKey: ['users', id],
  queryFn: () => fetchUser(id),
  enabled: !!id,                 // 條件查詢
  staleTime: 5 * 60 * 1000,     // 5 分鐘快取
})

// ✅ 標準 useMutation
const { mutate, isPending } = useMutation({
  mutationFn: createUser,
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['users'] })
    toast.success('建立成功')
  },
  onError: (error) => {
    toast.error(error.message)
  },
})
```

---

## Axios 設定

```typescript
// src/lib/axios.ts
import axios from 'axios'
import { useAuthStore } from '@/stores/authStore'

const instance = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  timeout: 10000,
})

// 請求攔截器：自動帶入 Token
instance.interceptors.request.use((config) => {
  const token = useAuthStore.getState().token
  if (token) config.headers.Authorization = `Bearer ${token}`
  return config
})

// 回應攔截器：統一錯誤處理
instance.interceptors.response.use(
  (response) => response.data,  // 自動解包 data
  (error) => {
    if (error.response?.status === 401) {
      useAuthStore.getState().logout()
      window.location.href = '/login'
    }
    return Promise.reject(error.response?.data || error)
  }
)

export default instance
```

---

## 表單規範（React Hook Form + Zod）

```tsx
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'
import { Form, FormField, FormItem, FormLabel, FormControl, FormMessage } from '@/components/ui/form'
import { Input } from '@/components/ui/input'
import { Button } from '@/components/ui/button'

const formSchema = z.object({
  name: z.string().min(2, '姓名至少 2 個字'),
  email: z.string().email('請輸入有效的 Email'),
})

type FormValues = z.infer<typeof formSchema>

export function UserForm({ onSubmit }: { onSubmit: (values: FormValues) => void }) {
  const form = useForm<FormValues>({
    resolver: zodResolver(formSchema),
    defaultValues: { name: '', email: '' },
  })

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
        <FormField
          control={form.control}
          name="name"
          render={({ field }) => (
            <FormItem>
              <FormLabel>姓名</FormLabel>
              <FormControl>
                <Input placeholder="請輸入姓名" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        <Button type="submit">送出</Button>
      </form>
    </Form>
  )
}
```

---

## 環境變數清單

```env
VITE_API_BASE_URL=http://localhost:3001/api/v1
VITE_APP_NAME=MyApp
VITE_APP_VERSION=1.0.0
```

---

## 新增頁面的步驟

1. `src/types/` → 定義相關型別
2. `src/api/xxx.api.ts` → 建立 API 呼叫函式
3. `src/pages/xxx/` → 建立頁面元件
4. `src/main.tsx` → 新增路由設定
5. 共用元件放至 `src/components/shared/`
