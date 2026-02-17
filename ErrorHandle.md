# Next.js Error Handling（重點複習版）

> 目標：分清楚兩類錯誤：**Expected errors**（可預期）vs **Uncaught exceptions**（非預期/bug）✅  
> 對應工具：`useActionState`、`notFound()` + `not-found.tsx`、`error.tsx`、`global-error.tsx`

---

## 1) 兩大類錯誤（先背這個）
### A) Expected errors（可預期）
- 例：表單驗證失敗、API 回傳失敗（正常流程可能發生）
- 原則：**不要 throw**，要「明確處理並回傳給 client」

### B) Uncaught exceptions（非預期）
- 例：程式 bug、不可預期例外
- 原則：**throw** → 交給 **Error Boundary** 顯示 fallback UI

---

## 2) Expected errors 怎麼處理？
### Server Functions / Server Actions
- 用 `useActionState`
- 作法：
  - action 以「回傳值」表示錯誤（例如 `{ message: '...' }`）
  - client 端用 `state.message` 顯示錯誤
- 重點：對 expected errors **避免 try/catch + throw**

### Server Components（抓資料）
- 用 `res.ok` 判斷
- 可以：
  - conditional render error UI
  - 或 redirect（視需求）

---

## 3) 404：Not Found
- 在 route segment 內呼叫 `notFound()`
- 搭配 `not-found.tsx` 定義 404 UI

---

## 4) Uncaught exceptions：用 Error Boundaries（error.tsx）
- Next.js 用 Error Boundary 捕捉「渲染期間」的例外
- 建立方式：
  - 在 route segment 放 `error.tsx`
  - 必須是 **Client Component**（要 `'use client'`）
- 會收到：
  - `error`
  - `reset()`：嘗試重新 render 該 segment
- 錯誤會往上 bubble 到最近的 error boundary  
  → 可在不同層級放多個 `error.tsx` 做「分層處理」

---

## 5) Error Boundary 捕不到哪些錯？
- **event handlers** 內的錯（onClick 等）不會被 error boundary 自動捕捉
- 這類要你自己：
  - try/catch
  - 用 `useState/useReducer` 存錯誤狀態並顯示 fallback

> 補充：`useTransition` 的 `startTransition` 裡若丟未處理錯誤，會往最近的 error boundary 冒泡

---

## 6) 全域錯誤：global-error.tsx
- 放在 `app/global-error.tsx`
- 用於 root layout 層級的錯誤 UI
- 必須自己提供 `<html>` 與 `<body>`（因為會取代 root layout）

---
