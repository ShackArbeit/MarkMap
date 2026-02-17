# Next.js Metadata & OG Images（重點複習版）

> 目標：用 Metadata API 提升 **SEO / 分享預覽**，並理解 **靜態 vs 動態 metadata**、**檔案式 favicon/OG**、**動態 OG 圖** ✅

---

## 1) Metadata API 提供什麼？
- 用來定義頁面的 `<head>`（SEO、社群分享）
- 三種主要方式：
  1) 靜態 `metadata` 物件
  2) 動態 `generateMetadata()`
  3) 特殊檔案（favicon / OG / twitter / robots / sitemap）

> `metadata` / `generateMetadata` **只能在 Server Components** 使用

---

## 2) 預設一定會有的 meta（即使你沒寫）
- charset：`<meta charset="utf-8" />`
- viewport：`<meta name="viewport" content="width=device-width, initial-scale=1" />`

---

## 3) 靜態 metadata（metadata 物件）
- 在 `layout.tsx` 或 `page.tsx` export：
  - `export const metadata: Metadata = {...}`

---

## 4) 動態 metadata（generateMetadata）
- 用於 metadata 需要依賴資料（例如文章標題）
- `export async function generateMetadata(...) { ... }`

### Streaming metadata（只記重點）
- 動態頁：metadata 可能會「獨立串流」注入 HTML（不阻塞 UI）
- 但對部分 bots/crawlers（要求 head 先完整）會停用 streaming
  - 以 request 的 User-Agent 判斷
- 可用 `htmlLimitedBots` 在 next config 調整/關閉
- 靜態頁（build time 可算完）不需要 streaming

---

## 5) 避免 metadata 與 page 重複抓資料
- 用 `React.cache()` 把資料取得函式 memoize
- `generateMetadata` 和 `Page` 都呼叫同一個 cached function → 同一次 request 只抓一次

---

## 6) File-based metadata（特殊檔案）
常見特殊檔案：
- `favicon.ico` / `icon.*` / `apple-icon.*`
- `opengraph-image.*`（OG 圖）
- `twitter-image.*`
- `robots.txt`
- `sitemap.xml`

### 覆蓋規則（記這句）
- 越「深層路由資料夾」的檔案 **優先**（更 specific）

---

## 7) Favicons（favicon）
- 最常見：在 `app/` 根目錄放 `favicon.ico`
- 也可以用程式動態生成（知道即可）

---

## 8) Open Graph（OG）圖片
### 靜態 OG 圖
- 放 `opengraph-image.jpg/png/...`
- 可在 `app/` 根目錄（全站預設）
- 也可放在子資料夾（例如 `app/blog/opengraph-image.jpg`）做路由專用

### 動態 OG 圖（依資料生成）
- 用 `opengraph-image.tsx`
- 回傳 `new ImageResponse(...)`（JSX + 部分 CSS）
- 常見用途：每篇文章一張不同 OG 圖

> ImageResponse：支援 flexbox 與部分 CSS；`display: grid` 等進階排版不一定可用

---
