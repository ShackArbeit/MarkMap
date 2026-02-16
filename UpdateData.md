# Next.js Updating Data（重點複習版）

> 目標：掌握 **Server Functions / Server Actions** 用來做資料更新（mutations），以及更新後如何 **refresh / revalidate / redirect** ✅

---

## 1) Server Functions / Server Actions 是什麼？
- **Server Function**：在 server 執行的 `async function`
- **Server Action**：在「表單提交 / mutation」情境使用的 Server Function（算是一種用法）
- 為什麼一定是 async？
  - client 呼叫時本質上是一次 network request

### 重要特性
- Actions 背後使用 **POST**（只有 POST 能觸發）
- 與 Next.js cache 架構整合：
  - 一次 roundtrip 可回傳更新後 UI + 新資料（框架負責協調）

---

## 2) 怎麼建立 Server Function？
### 用 `'use server'`
- 方式 A：在函式內加 `'use server'`
- 方式 B：在檔案頂部加 `'use server'`（該檔案 exports 都是 server functions）

---

## 3) 在哪裡可以「定義 / 呼叫」？
### Server Components
- ✅ 可以 inline 定義 Server Action（函式內 `'use server'`）
- ✅ 表單具備 progressive enhancement：
  - 就算 JS 還沒載入/被關掉，form 仍能提交

### Client Components
- ❌ 不能在 Client Component 內「定義」Server Function
- ✅ 但可以「呼叫」：
  - 從 `'use server'` 檔案 import
  - 用 `formAction` / `<form action={...}>` / event handler 呼叫
- 補充：JS 未載入時，提交會排隊；hydration 後提交不會整頁刷新

---

## 4) 觸發（invoke）Server Functions 的兩種主流方式
### A) `<form action={serverAction}>`
- action 會自動收到 `FormData`
- 用 `formData.get('field')` 取值

### B) Client event handler / useEffect
- 例如 `onClick` 內 `await action()` 再更新 client state
- useEffect 可在 mount 或依賴變動時觸發 action（例如 view count）

> 注意：client 目前是「一次 dispatch 一個 action 並 await」（實作細節）
> 需要平行工作：建議在「同一個 Server Function」內做，或改用其他 server 方案（Route Handler）

---

## 5) Pending 狀態（loading feedback）
- 用 `useActionState` + `startTransition`
- 取得 `pending` boolean 來顯示 spinner / disabled 狀態

---

## 6) 更新後 UI 同步：三個最常用的操作
### A) `refresh()`（更新畫面）
- 用於 mutation 後想讓目前頁面反映最新狀態
- 作用：刷新 client router（讓 UI 重新對齊 server）
- 注意：`refresh()` **不會** revalidate tagged data

### B) `revalidatePath()` / `revalidateTag()`（更新快取）
- 更新資料後，讓 Next.js cache 失效並重抓
- 常用：
  - `revalidatePath('/posts')`
  - `revalidateTag('posts')`（需先 tag）

### C) `redirect()`（更新後導頁）
- 更新後把使用者導到新頁
- 重點：
  - `redirect()` 會中斷流程（後面程式不會跑）
  - 若要確保資料新鮮 → 先 `revalidatePath/Tag` 再 redirect

---

## 7) Cookies（在 Server Action 內操作）
- 可在 Server Action 用 `cookies()`：
  - `get / set / delete`
- set/delete cookie 後：
  - Next.js 會在 server 重新 render 目前 page/layout
  - 讓 UI 立即反映新的 cookie 值
- 補充：
  - client state 會被保留（相同元件）
  - effects 可能依依賴而重跑

---
