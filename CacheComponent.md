# Next.js Cache Components（重點速讀版）

> 用來複習：抓住 **Cache Components + PPR + use cache + Suspense + runtime data** 的核心即可 ✅

---

## 1) 什麼是 Cache Components？
- **Opt-in**：`next.config.ts` → `cacheComponents: true`
- 目的：同一條 route 混合
  - **Static（快）**
  - **Cached（可快取）**
  - **Dynamic（請求時才算）**
- 核心成果：
  - 先產出 **Static HTML Shell**（秒出畫面）
  - 動態區塊再用 **Streaming / request time** 補上

---

## 2) 渲染模型：Partial Prerendering（PPR）
- 啟用後預設就是 **PPR**
- Build time 先 prerender：
  - 能完成的 → 進 **static shell**
  - 不能完成的 → 你必須明確選擇怎麼處理

---

## 3) 遇到「不能 prerender」的內容：你一定要做 2 選 1
### A) 用 `<Suspense>` 延後到 request time
- **fallback** 會進 static shell
- 真正內容在 request time **streaming** 進來
- boundary 盡量貼近慢元件（讓 shell 最大化）

### B) 用 `'use cache'` 把結果快取進 shell
- 適用：**不依賴 runtime data**、資料不需每次都最新
- 搭配 `cacheLife()` 定義快取時間

> 沒做 A 或 B → 會看到 `Uncached data was accessed outside of <Suspense>` 錯誤

---

## 4) 哪些東西「一定」是 request time（不能 use cache）
### Runtime data（需要 request context）
- `cookies()`, `headers()`, `searchParams`, `params`（除非提供 `generateStaticParams`）
- 規則：
  - 讀 runtime APIs 的元件 **必須包在 `<Suspense>`**
  - runtime data **不能** 跟 `'use cache'` 在同一 scope

✅ 正確作法：先在非 cache scope 讀 cookies → 把值當參數傳進 cached function/component

---

## 5) 非決定性運算（random / now / uuid）
- 想每次 request 都不同：
  - 先 `await connection()`（明確 defer）→ 再算 random/now/uuid → 包 `<Suspense>`
- 想大家看到同一份（直到 revalidate）：
  - 放進 `'use cache'` scope

---

## 6) `'use cache'` 的更新方式（記這兩個就好）
- 時間型：`cacheLife('hours' | 'days' | 'max' | 自訂)`
- 事件型（on-demand）：
  - `cacheTag()` + `updateTag()`（立即刷新）
  - `cacheTag()` + `revalidateTag()`（可接受延遲更新）

---

## 7) 啟用後常見設定的變化（快速記）
- `revalidate` → 用 `cacheLife()` 取代
- `fetchCache` → 不需要（use cache scope 內 fetch 自動快取）
- `dynamic=force-*` → 多數情境不再需要（依錯誤提示改用 Suspense / use cache）
- `runtime='edge'` → ❌ 不支援（Cache Components 需要 Node.js runtime）

---

## 8) 導航體驗：`<Activity>`（知道概念即可）
- client-side navigation 時，舊頁面可能先變 `hidden` 而非 unmount
- 好處：返回上一頁時 **狀態保留**（表單、展開區塊）

---
