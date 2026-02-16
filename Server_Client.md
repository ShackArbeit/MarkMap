# Next.js Server / Client Components（重點複習版）

> 目的：快速掌握 **何時用 Server / Client**、**RSC Payload**、**Hydration**、**怎麼組合** ✅  
>（可直接用 markmap）

---

## 1) 預設規則
- `layout.tsx` / `page.tsx` **預設是 Server Components**
- 需要互動或瀏覽器 API 才使用 **Client Components（'use client'）**

---

## 2) 什麼時候用 Client Components？
需要下列任一功能就用 Client：
- 狀態 / 事件：`useState`, `onClick`, `onChange`
- Lifecycle：`useEffect`
- Browser APIs：`window`, `localStorage`, `geolocation`...
- 自訂 hooks
- React Context Provider（context 不支援在 Server Components 內宣告/使用）

---

## 3) 什麼時候用 Server Components？
適合放在 Server：
- 資料抓取（DB / API）靠近資料源
- 使用 **secrets**（API key / token）不暴露到 client
- 減少送到瀏覽器的 JS
- 提升 FCP、支援 Streaming（搭配 Suspense）

---

## 4) 'use client' 的重要概念（Client boundary）
- 在檔案頂部加 `'use client'` 代表建立 **Server/Client 分界**
- 這個檔案的：
  - imports
  - children
  都會被打進 **client bundle**
- 最佳實務：只把「真的需要互動」的元件標成 client，避免 bundle 變大

---

## 5) Server / Client 的渲染流程（必背）
### Server 端
- Server Components → 產生 **RSC Payload**
- Client Components + RSC Payload → 用來 **預渲染 HTML**

### Client 第一次載入
- HTML：先顯示「快速但不可互動」的畫面
- RSC Payload：對齊 server/client component tree
- JS：Hydrate Client Components → 變可互動

> Hydration：React 把事件處理器掛回 DOM，使靜態 HTML 可互動

### 後續導航（client-side navigation）
- RSC Payload 會被 prefetch + cache
- Client Components 在 client 端渲染（不再依賴 server-rendered HTML）

---

## 6) Server → Client 傳資料（Props）
- Server Component 抓資料後，用 props 傳給 Client Component
- 注意：傳給 Client 的 props 必須 **可序列化（serializable）**
- 進階：也可用 React `use()` + Suspense 來「串流」promise（略）

---

## 7) 交錯組合（Interleaving）
- Client Component 可以接收 **Server Component 當 children**
- 常見 pattern：
  - Client：控制 UI 狀態（Modal 開關）
  - Server：放資料抓取元件（Cart）

---

## 8) Context Provider（必考點）
- React context **不支援在 Server Components 內使用**
- 解法：
  - 建立 Client Provider（`'use client'`）
  - 在 Server 的 layout/page 把 Provider 包住 children
- 最佳實務：Provider 盡量 **包深一點**（不要包整個 `<html>`），讓靜態部分更好被最佳化

---

## 9) 共用資料：React.cache + Context（重點觀念）
- Server：用 `React.cache()` 包資料取得函式（同一 request 內去重）
- Layout：先拿到 promise（不要 await）丟進 Client Provider
- Client：從 context 拿 promise，用 `use()` 解析（搭配 Suspense）
- 注意：`React.cache` 只在 **同一次 request** 範圍有效（不跨 request）

---

## 10) 第三方元件（library 沒寫 'use client'）
- 若第三方元件其實用到 client-only features（useState 等）：
  - 用你自己的 Client wrapper 包起來（加 `'use client'`）
  - 之後 Server Component 就能安全使用該 wrapper

---

## 11) 防止「環境污染」（Server-only / Client-only）
- 風險：不小心把 server-only code（含 secrets）import 到 client
- 作法：
  - 在 server-only module 頂部加：`import 'server-only'`
  - 若 client-only module：`import 'client-only'`
- 效果：錯誤環境 import 會在 build-time 報錯（更安全）

---
