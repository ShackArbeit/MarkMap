# Next.js Fetching Data（重點複習版）

> 目標：快速掌握 **Server/Client 抓資料方式**、**去重複請求與快取**、**Streaming（loading.tsx / Suspense）** ✅

---

## 1) Server Components：在哪裡抓資料？
Server Components 可用任何 async I/O：
- `fetch`
- ORM / Database
- Node.js `fs`

### fetch 重點
- 寫成 `async` component，`await fetch()`
- `fetch` 回應 **預設不 cache**
- 想強制每次都動態抓：`{ cache: 'no-store' }`

### ORM / DB 重點
- 因為在 server render → 可安全查 DB
- 同樣用 `async` + `await`

---

## 2) Client Components：怎麼抓資料？
兩條路：
1) React `use()`（從 Server 把 Promise 串到 Client）
2) SWR / React Query（套件自帶 cache/state 語意）

### (A) React `use()` + Suspense（串流）
- Server：先呼叫抓資料函式 **不要 await**，把 Promise 傳下去
- Client：`use(promise)` 讀資料
- 外層：用 `<Suspense fallback>` 顯示 loading

### (B) SWR / React Query
- Client 端直接抓
- 自行處理 `isLoading / error / cache`（依套件規則）

---

## 3) 去重複請求（Deduplicate）與快取（Cache）
### Request memoization（同一次 request 內）
- 同一 render pass 內相同 `GET/HEAD`（同 URL+options）會自動合併
- 範圍：只在 **單次 request 生命週期**
- 想退出：`fetch` 帶 `AbortSignal`

### Next.js Data Cache（可跨 requests）
- 例如 `fetch` 用 `cache: 'force-cache'`
- 能共享：同 render pass + 後續 incoming requests

### 不用 fetch（直接 DB/ORM）
- 用 `React.cache()` 包資料存取函式（避免同 request 內重複查）

---

## 4) Streaming（重點：避免慢資料卡整頁）
> 前提：文件假設你開了 `cacheComponents`（Next.js 15 canary）

### 為什麼需要？
- Server Components 抓資料慢 → 可能阻塞整條 route

### 兩種方式
1) `loading.tsx`
- 放在 route 資料夾
- 效果：切頁立刻看到 loading（整段 route streaming）

2) `<Suspense>`
- 更細粒度：只包住慢的區塊
- boundary 外內容先顯示，boundary 內再串流補上

### Loading UI 原則
- 要「有意義」：skeleton / spinner / 預覽區塊（提升體感）

---

## 5) 常見資料抓取模式（只記三個）
### Sequential（序列）
- 後一個請求依賴前一個結果（會卡）
- 用 Suspense 只能讓「後段」串流；第一段仍會先阻塞
- 改善：加 `loading.tsx` 或把第一段變快/可快取

### Parallel（平行）
- 先拿 promises，再 `Promise.all()`
- 注意：`Promise.all` 任一失敗會全失敗（需要可用 `Promise.allSettled`）

### Preload（預載）
- 先 `preload(id)`（用 `void getItem(id)` 提前發起請求）
- 再做其他阻塞工作
- 可搭配 `React.cache()` + `server-only` 做成 server-only 可重用工具

---
