# Next.js Linking & Navigating（重點複習版）

> 目標：掌握 Next.js 導航為什麼快：**Server Rendering + Prefetch + Streaming + Client-side transitions** ✅  
>（可直接用 markmap）

---

## 1) Next.js 導航為什麼需要最佳化？
- 路由預設 **在 server render**
- 問題：切頁時可能要等 server 回應才看到新頁
- 解法：Next.js 內建
  - **Prefetching**
  - **Streaming（loading.tsx / Suspense）**
  - **Client-side transitions（Link）**

---

## 2) Server Rendering（兩種）
- **Static Rendering / Prerendering**
  - build time 或 revalidation 時產生
  - 結果可快取 → 速度快
- **Dynamic Rendering**
  - request time 才 render
  - 資料新但可能較慢

> 初次造訪也會產生 HTML（首屏顯示用）

---

## 3) Prefetching（預抓路由）
- 定義：使用者還沒點之前，先在背景把「下一頁需要的東西」載好
- Next.js 會自動 prefetch：
  - `<Link>` 進入 viewport（或 hover）時
- `<a>`（原生）不會自動 prefetch

### Prefetch 的範圍（重點）
- **Static route**：通常可 **完整 prefetch**
- **Dynamic route**：可能 **跳過**，或有 `loading.tsx` 時 **部分 prefetch**

---

## 4) Streaming：用 `loading.tsx` 讓動態路由也「先有畫面」
- 在 route 資料夾加 `loading.tsx`
- Next.js 會自動用 `<Suspense>` 包住 page
- 效果：
  - 先顯示 **fallback/loading UI**
  - 真正內容 ready 後再替換

### 好處（記 3 個就好）
- 使用者立刻看到回饋（不會覺得卡住）
- shared layout 仍可互動、導航可中斷
- Core Web Vitals 更好（TTFB / FCP / TTI）

---

## 5) Client-side transitions（Link 的關鍵價值）
- 傳統 SSR 切頁＝整頁 reload → state 清空、scroll 重置、互動中斷
- Next.js `<Link>`：
  - 保留 shared layout
  - 用 prefetched loading 或新頁內容「局部替換」
- 讓 SSR app 也有 SPA 的切頁體感

---

## 6) 常見「切頁變慢」原因 + 對策
### A) Dynamic route 沒有 `loading.tsx`
- 會等 server 回應才有畫面 → 體感卡
- ✅ 對策：加 `loading.tsx`

### B) Dynamic segment 沒有 `generateStaticParams`
- 原本可 prerender 的路由變成 request-time dynamic
- ✅ 對策：補 `generateStaticParams`

### C) 慢網路：prefetch 來不及
- fallback 可能也還沒 prefetch 到
- ✅ 對策：用 `useLinkStatus()` 做「立即提示」（如小 spinner / 進度條）

### D) Prefetch 太多浪費資源（大量 links）
- ✅ 對策 1：`<Link prefetch={false}>`
- ✅ 對策 2：只在 hover 才 prefetch（hover prefetch pattern）

### E) Hydration 還沒完成
- `<Link>` 是 Client Component，未 hydrated 前無法開始 prefetch
- ✅ 對策：
  - 降低 client bundle（bundle analyzer）
  - 能搬到 server 的邏輯就搬到 server

---

## 7) Native History API（可與 Next Router 整合）
- `window.history.pushState`：
  - 新增 history entry（可返回）
  - 常用：用 query 更新排序/篩選
- `window.history.replaceState`：
  - 替換當前 entry（不可返回）
  - 常用：切換語系/路徑但不想新增返回點
- 這些操作可與 Next Router 同步（搭配 `usePathname`, `useSearchParams`）

---
