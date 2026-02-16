# Next.js Caching & Revalidating（重點複習版）

> 目標：記住 Next.js 常用快取/更新 API：  
> **fetch cache**、**tag**、**revalidateTag / updateTag**、**revalidatePath** ✅

---

## 1) 快取 vs 重新驗證（Revalidate）
- **Caching**：把資料/運算結果存起來，下次更快
- **Revalidation**：不用整個 rebuild，也能讓快取更新

---

## 2) fetch 的快取（單筆請求）
- `fetch` 預設 **不快取**
- 要快取單筆請求：
  - `{ cache: 'force-cache' }`
- 要定時更新（TTL）：
  - `{ next: { revalidate: 秒數 } }`
- fetch 也能加 tag：
  - `{ next: { tags: ['user'] } }`

> 補充：即使 fetch 不快取，Next.js 仍可能 prerender 並快取 HTML  
> 想「保證動態」可用 `connection()`（概念記住即可）

---

## 3) cacheTag（Cache Components + use cache）
- 用途：在 **'use cache'** 的快取結果上加 tag（不限 fetch）
- 可用在：
  - DB query / fs / 任意 server-side 計算
- 之後用 `revalidateTag` 或 `updateTag` 讓這個 tag 的快取失效/更新

---

## 4) revalidateTag（用 tag 更新快取）
- 事件後（mutation 後）依 tag 讓快取更新
- 推薦用法：
  - `revalidateTag('tag', 'max')`
  - 行為：stale-while-revalidate（先回舊的、背景抓新的）
- 可在：
  - Server Actions
  - Route Handlers

---

## 5) updateTag（立即更新，專給 Server Actions）
- 只能在 **Server Actions** 用
- 行為：**立刻讓快取過期**（read-your-own-writes）
- 適合：
  - 剛新增/修改資料後，馬上要在下一個畫面看到最新結果（例如 redirect 後）

### revalidateTag vs updateTag（一句話記法）
- `updateTag`：Server Action 專用、**立即過期**、強一致體感
- `revalidateTag`：Action/Handler 都可、可用 `'max'` 做 **SWR** 更新

---

## 6) revalidatePath（用路徑更新快取）
- 事件後針對「某個路由」做更新
- 可在：
  - Server Actions
  - Route Handlers
- 常用：更新 profile / list 頁面等

---

## 7) unstable_cache（Legacy / 不推薦）
- 實驗性舊 API，用來快取 async function（例如 DB query）
- 現在建議：
  - 改用 **Cache Components + 'use cache'**（優先）
- unstable_cache 可設定：
  - `tags`
  - `revalidate`（秒）

---
