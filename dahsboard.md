# Vibe Coding 課程管理後台系統 - 產品需求文檔 (PRD)

## 文檔版本
- **版本**: 1.0.0
- **最後更新**: 2024-10-07
- **作者**: Claude Code

---

## 目錄
1. [產品概述](#1-產品概述)
2. [系統架構](#2-系統架構)
3. [技術棧](#3-技術棧)
4. [核心功能模塊](#4-核心功能模塊)
5. [數據模型](#5-數據模型)
6. [API 規格](#6-api-規格)
7. [UI/UX 設計規範](#7-uiux-設計規範)
8. [第三方集成](#8-第三方集成)
9. [環境配置](#9-環境配置)
10. [部署指南](#10-部署指南)

---

## 1. 產品概述

### 1.1 產品定位
Vibe Coding 課程管理後台系統是一個專為線上課程銷售與學員管理設計的全功能管理平台，整合了 LINE Bot 自動化訊息系統、用戶追蹤、付款管理與數據分析功能。

### 1.2 核心價值
- **自動化管理**: 透過 LINE Bot 自動處理用戶互動、報名流程與付款確認
- **智能再行銷**: 基於用戶行為的自動化再行銷系統，提升轉換率
- **數據驅動**: 完整的數據追蹤與視覺化分析，協助優化銷售策略
- **高效溝通**: 整合聊天室與訊息推播功能，實現一對一與群發訊息管理

### 1.3 目標用戶
- 課程管理員
- 銷售團隊
- 客服人員

---

## 2. 系統架構

### 2.1 整體架構
```
┌─────────────────────────────────────────────────────────────┐
│                        Frontend Layer                        │
│  Next.js 15 (App Router) + React + TypeScript + Tailwind   │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                      Backend Layer                           │
│         Next.js API Routes + Server Actions                  │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────┬──────────────────┬─────────────────────┐
│   Firebase       │   LINE Messaging │   Vercel Cron       │
│   Firestore      │   API            │   (Scheduled Jobs)  │
│   (Database)     │   (Integration)  │                     │
└──────────────────┴──────────────────┴─────────────────────┘
```

### 2.2 路由結構
```
/admin
  /login           → 管理員登入頁
  /dashboard       → 主控制台（用戶管理、數據分析）
  /messages        → 訊息管理中心
  /chat            → 即時聊天室

/api/admin
  /auth
    /login         → 登入 API
    /logout        → 登出 API
  /users           → 用戶列表與篩選
    /[userId]
      /activities           → 用戶活動記錄
      /confirm-payment      → 確認付款
      /clear                → 清除用戶資料
      /remarketing-history  → 再行銷歷史
  /messages
    /route         → 訊息列表
    /send          → 發送訊息
  /chat
    /messages      → 聊天訊息（讀取與發送）
  /analytics
    /daily-users   → 每日用戶統計
  /remarketing
    /templates     → 再行銷範本
    /send          → 發送再行銷訊息
    /analytics     → 再行銷分析

/api/remarketing
  /rules           → 再行銷規則管理（CRUD）
  /execute         → 執行再行銷

/api/linebot
  /webhook         → LINE Bot Webhook 接收訊息
  /test            → 測試 LINE Bot 連線

/api/marketing
  /schedule        → 行銷活動排程
  /cron            → Cron Job 執行再行銷
```

---

## 3. 技術棧

### 3.1 前端技術
| 技術 | 版本 | 用途 |
|------|------|------|
| Next.js | 15.x | React 框架，支援 SSR/SSG |
| React | 18.x | UI 組件庫 |
| TypeScript | 5.x | 類型安全 |
| Tailwind CSS | 3.x | CSS 框架 |
| Lucide React | latest | Icon 圖標庫 |
| ECharts for React | latest | 數據圖表視覺化 |

### 3.2 後端技術
| 技術 | 版本 | 用途 |
|------|------|------|
| Next.js API Routes | 15.x | RESTful API |
| Firebase Admin SDK | latest | 後端 Firebase 操作 |
| @line/bot-sdk | latest | LINE Messaging API SDK |

### 3.3 數據庫
| 技術 | 用途 |
|------|------|
| Firebase Firestore | NoSQL 數據庫，儲存用戶資料、活動記錄、再行銷規則 |

### 3.4 部署平台
| 平台 | 用途 |
|------|------|
| Vercel | 前端與 API 部署、Cron Jobs |

---

## 4. 核心功能模塊

### 4.1 認證系統

#### 4.1.1 功能描述
- 管理員密碼登入機制
- Session 管理（基於 Cookie）
- 受保護路由的 Middleware 驗證

#### 4.1.2 登入流程
1. 用戶訪問 `/admin/*` 路由（除了 `/admin/login`）
2. Middleware 檢查 `admin-auth` Cookie
3. 若未登入，重定向至 `/admin/login?from=[原始路徑]`
4. 輸入管理員密碼
5. 驗證成功後設置 Cookie，重定向回原始路徑

#### 4.1.3 實作細節
- **密碼儲存**: 環境變數 `ADMIN_PASSWORD`（預設: `vibe-admin-2024`）
- **Token 格式**: Base64 編碼的 `admin:${ADMIN_PASSWORD}`
- **Cookie 名稱**: `admin-auth`
- **Middleware 路徑**: `/middleware.ts`

#### 4.1.4 關鍵代碼檔案
- `app/admin/login/page.tsx` - 登入頁面
- `app/api/admin/auth/login/route.ts` - 登入 API
- `app/api/admin/auth/logout/route.ts` - 登出 API
- `middleware.ts` - 路由保護

---

### 4.2 Dashboard 主控制台

#### 4.2.1 功能概述
Dashboard 是系統的核心頁面，提供完整的用戶管理、數據分析與行銷功能。

#### 4.2.2 頁面結構
```
Dashboard (app/admin/dashboard/page.tsx)
├── Header
│   ├── Logo/Title
│   └── Logout Button
├── Stats Cards (4 個統計卡片)
│   ├── 總用戶數
│   ├── 有興趣人數
│   ├── 已繳費人數
│   └── 總營收
├── Tab Navigation
│   ├── 用戶管理
│   ├── 數據分析
│   ├── 再行銷
│   └── 群發訊息
└── Tab Content (根據選中的 Tab 顯示)
```

#### 4.2.3 統計卡片計算邏輯

##### 總用戶數
```typescript
const totalUsers = users.filter(u => u.isFollowing).length
```
- **說明**: 統計所有追蹤（未封鎖）的用戶數量

##### 有興趣人數
```typescript
const activeUsers = users.filter(u => u.registrationStatus === 'interested').length
```
- **說明**: 統計已查詢報名資訊但尚未付款的用戶

##### 已繳費人數
```typescript
const registeredUsers = users.filter(u =>
  u.registrationStatus === 'paid' || u.registrationStatus === 'confirmed'
).length
```
- **說明**: 統計已匯款（待確認或已確認）的用戶

##### 總營收
```typescript
const revenue = users.reduce((total, user) => {
  if (user.registrationStatus === 'confirmed' && user.paymentInfo?.amount) {
    return total + user.paymentInfo.amount
  }
  return total
}, 0)
```
- **說明**: 僅計算已確認收款的金額總和

---

### 4.3 用戶管理（Tab 1）

#### 4.3.1 功能列表
- 用戶列表展示
- 用戶搜尋與篩選
- 用戶狀態管理
- 付款確認
- 用戶活動歷史查看
- 再行銷歷史查看

#### 4.3.2 用戶列表 UI 設計

##### 表格欄位
| 欄位名稱 | 說明 | 顯示方式 |
|---------|------|---------|
| 用戶名稱 | LINE 顯示名稱 + 頭像 | 頭像 + 文字 |
| User ID | LINE User ID（部分隱藏） | 前 8 碼 + ... + 複製按鈕 |
| 狀態 | 報名狀態 | Badge（顏色標示） |
| 金額 | 付款金額 | NT$ 格式 |
| 票種 | 票券類型 | 文字 |
| 最後活動 | 最後互動時間 | 相對時間 |
| 操作 | 快速操作按鈕 | 按鈕組 |

##### 狀態 Badge 顏色規範
```typescript
const statusColors = {
  'confirmed': 'bg-green-100 text-green-800',  // 已確認收款
  'paid': 'bg-yellow-100 text-yellow-800',     // 已匯款待確認
  'interested': 'bg-blue-100 text-blue-800',   // 有興趣
  'default': 'bg-gray-100 text-gray-800'       // 尚未開始
}
```

#### 4.3.3 篩選功能

##### 狀態篩選
- **全部**: 顯示所有追蹤中的用戶
- **已確認**: `registrationStatus === 'confirmed'`
- **待確認**: `registrationStatus === 'paid'`
- **有興趣**: `registrationStatus === 'interested'`
- **尚未開始**: 沒有 `registrationStatus`

##### 搜尋功能
- 輸入框即時搜尋
- 搜尋欄位: `displayName`（用戶名稱）
- 不區分大小寫

#### 4.3.4 付款確認功能

##### 流程圖
```
用戶點擊「確認收款」
    ↓
彈出付款確認對話框 (PaymentConfirmDialog)
    ↓
顯示用戶資訊與匯款資訊
    ↓
管理員輸入確認金額
    ↓
選擇票種（一般票/邀請票/公關票）
    ↓
點擊「確認收款」
    ↓
調用 API: POST /api/admin/users/[userId]/confirm-payment
    ↓
更新 Firestore 用戶狀態為 'confirmed'
    ↓
發送 LINE 確認訊息給用戶
    ↓
刷新用戶列表
```

##### 付款確認對話框 (PaymentConfirmDialog)
**組件路徑**: `components/admin/PaymentConfirmDialog.tsx`

**Props**:
```typescript
interface PaymentConfirmDialogProps {
  isOpen: boolean
  user: User | null
  onClose: () => void
  onConfirm: (confirmedAmount: number) => Promise<void>
}
```

**UI 元素**:
1. 用戶資訊區
   - 用戶名稱
   - 用戶 ID
   - 用戶頭像

2. 匯款資訊區
   - 匯款後五碼
   - 申報金額
   - 匯款時間

3. 票種選擇
   - 一般票（需填金額）
   - 邀請票（免費）
   - 公關票（免費）

4. 金額輸入
   - 預填用戶申報金額
   - 可手動調整

5. 操作按鈕
   - 取消
   - 確認收款

##### 確認訊息範本
```
🎉 恭喜！您的報名已確認成功，期待在課程中見到您！

📅 課程名稱：《Vibe Coding 工作術》
📅 課程日期：2025/10/12 整日線下大課 + 四週實戰陪跑
📍 上課地點：臺北市中山區民權西路20號 2樓-204房

⚠️ 麻煩在「10/5（日）」23:59 前完成課前表單～
裡面有基本資料、飲食需求、交通方式，
需要先統計好才能安排訂餐與課程準備 🙏

表單連結：
[課前表單 URL]

開課前一週會發送詳細的行前通知給您。

如有任何問題，歡迎隨時詢問！
```

#### 4.3.5 手動新增付款功能

##### 使用場景
- 用戶直接透過其他管道付款（如現場、ATM 轉帳但未回報）
- 特殊優惠或免費票種
- 補登遺漏的付款記錄

##### 操作流程
```
點擊「手動新增付款」按鈕
    ↓
選擇用戶
    ↓
輸入金額
    ↓
選擇票種
    ↓
確認
    ↓
更新用戶狀態為 'confirmed'
    ↓
發送確認訊息
```

##### 手動付款對話框 (ManualPaymentDialog)
**組件路徑**: `components/admin/ManualPaymentDialog.tsx`

**功能**:
- 用戶選擇器（下拉選單或搜尋）
- 金額輸入
- 票種選擇
- 備註欄位（選填）

---

### 4.4 數據分析（Tab 2）

#### 4.4.1 功能概述
提供視覺化的數據分析工具，協助管理員了解銷售漏斗、用戶行為與轉換率。

#### 4.4.2 分析組件

##### 4.4.2.1 銷售漏斗圖 (SalesFunnel)
**組件路徑**: `components/admin/SalesFunnelBar.tsx`

**資料計算**:
```typescript
const funnelData = [
  {
    stage: '用戶數',
    value: users.filter(u => u.isFollowing).length
  },
  {
    stage: '有興趣',
    value: users.filter(u => u.registrationStatus === 'interested').length
  },
  {
    stage: '已轉換',
    value: users.filter(u =>
      u.registrationStatus === 'paid' || u.registrationStatus === 'confirmed'
    ).length
  }
]
```

**視覺化**:
- 使用 ECharts 漏斗圖
- 顏色: 藍色 → 綠色 → 金色
- 顯示百分比與實際數字

**轉換率計算**:
```typescript
// 階段轉換率
const stageConversionRate = (nextStage / currentStage) * 100

// 流失率
const dropoffRate = ((currentStage - nextStage) / currentStage) * 100

// 總體轉換率
const overallConversionRate = (finalStage / initialStage) * 100
```

**優化建議**:
- 自動分析最大流失階段
- 提供改善建議
- 標示關鍵指標

##### 4.4.2.2 用戶狀態圓餅圖 (UserStatusPieChart)
**組件路徑**: `components/admin/UserStatusPieChart.tsx`

**資料分類**:
```typescript
const statusDistribution = {
  'confirmed': users.filter(u => u.registrationStatus === 'confirmed').length,
  'paid': users.filter(u => u.registrationStatus === 'paid').length,
  'interested': users.filter(u => u.registrationStatus === 'interested').length,
  'no_action': users.filter(u => !u.registrationStatus && u.isFollowing).length
}
```

**視覺化**:
- ECharts 圓餅圖
- 顏色對應狀態 Badge
- 顯示百分比與數量

##### 4.4.2.3 每日用戶統計 (DailyUserStats)
**組件路徑**: `components/admin/DailyUserStats.tsx`

**功能**:
- 顯示過去 30 天每日新增用戶數
- 折線圖展示趨勢
- 標示重要日期（如行銷活動日）

**資料來源**:
- API: `/api/admin/analytics/daily-users`
- 查詢 Firestore `users` collection 的 `followedAt` 欄位
- 按日期分組統計

---

### 4.5 再行銷系統（Tab 3）

#### 4.5.1 功能概述
自動化的再行銷系統，根據用戶行為與時間觸發條件，自動發送精準的行銷訊息。

#### 4.5.2 再行銷規則管理

##### 規則數據結構
```typescript
interface RemarketingRule {
  id: string
  name: string                    // 規則名稱
  description: string             // 規則說明
  triggerDays: number             // 加入後幾天觸發
  targetConditions: {
    hasInteracted: boolean        // 是否有互動過
    registrationStatus?: string[] // 目標註冊狀態
    notInStatus?: string[]        // 排除的狀態
  }
  messageTemplate: string         // 訊息模板 ID
  customMessage?: string          // 自訂訊息內容
  isActive: boolean               // 是否啟用
  createdAt: Date                 // 建立時間
  lastExecuted?: Date             // 最後執行時間
}
```

##### 預設規則
系統內建 5 個預設規則（可在 `lib/line/remarketingService.ts` 查看）:

1. **第 1 天 - 歡迎提醒**
   - 觸發時間: 加入後 1 天
   - 目標: 未互動且未付款的用戶
   - 訊息: 歡迎 + 課程介紹

2. **第 3 天 - 課程價值提醒**
   - 觸發時間: 加入後 3 天
   - 目標: 已互動但未付款的用戶
   - 訊息: 學員見證 + 價值強調

3. **第 5 天 - 優惠即將結束**
   - 觸發時間: 加入後 5 天
   - 目標: 有興趣但未付款的用戶
   - 訊息: 優惠倒數 + 急迫感

4. **第 7 天 - 最後機會**
   - 觸發時間: 加入後 7 天
   - 目標: 所有未付款用戶
   - 訊息: 最後召回

5. **付款提醒 - 24 小時**
   - 觸發時間: 查詢報名後 1 天
   - 目標: 有興趣但未付款
   - 訊息: 付款提醒

#### 4.5.3 規則 CRUD 操作

##### 新增規則
**UI 流程**:
```
點擊「新增規則」按鈕
    ↓
展開表單
    ↓
填寫規則資訊
  - 規則名稱（必填）
  - 規則說明（必填）
  - 觸發天數（必填，數字）
  - 目標條件
    - 是否需要互動
    - 包含狀態（多選）
    - 排除狀態（多選）
  - 訊息模板選擇或自訂訊息
  - 是否立即啟用
    ↓
點擊「儲存」
    ↓
POST /api/remarketing/rules
    ↓
儲存至 Firestore `remarketingRules` collection
    ↓
刷新規則列表
```

##### 編輯規則
**UI 流程**:
```
點擊規則卡片的「編輯」按鈕
    ↓
進入編輯模式（inline editing）
    ↓
修改規則欄位
    ↓
點擊「儲存」
    ↓
PUT /api/remarketing/rules
    ↓
更新 Firestore
    ↓
刷新規則列表
```

##### 刪除規則
**UI 流程**:
```
點擊規則卡片的「刪除」按鈕
    ↓
顯示確認對話框
    ↓
確認刪除
    ↓
DELETE /api/remarketing/rules?id=[ruleId]
    ↓
從 Firestore 刪除
    ↓
刷新規則列表
```

##### 啟用/停用規則
**UI 流程**:
```
點擊規則卡片的切換開關
    ↓
PUT /api/remarketing/rules
    ↓
更新 isActive 欄位
    ↓
立即生效（下次 Cron Job 執行時會讀取新狀態）
```

#### 4.5.4 再行銷執行機制

##### 執行方式
1. **自動執行（Cron Job）**
   - Vercel Cron Job 每天 09:00 執行
   - 路由: `/api/marketing/cron`
   - 配置於 `vercel.json`

2. **手動執行**
   - Dashboard 點擊「立即執行」按鈕
   - 路由: `POST /api/remarketing/execute`

##### 執行流程
```
開始執行
    ↓
檢查環境變數 ENABLE_AUTO_REMARKETING
    ↓
載入所有 isActive = true 的規則
    ↓
對每個規則:
  ├─ 計算目標日期（今天 - triggerDays）
  ├─ 查詢符合條件的用戶
  │   - followedAt 在目標日期範圍
  │   - isFollowing = true
  │   - 符合 targetConditions
  ├─ 檢查該用戶是否已收過此規則訊息（查 remarketingLogs）
  ├─ 若未收過:
  │   ├─ 取得訊息模板
  │   ├─ 個人化訊息（替換變數）
  │   ├─ 發送 LINE 訊息
  │   └─ 記錄至 remarketingLogs
  └─ 更新規則的 lastExecuted 時間
    ↓
回傳執行結果
  - 總處理數
  - 成功數
  - 失敗數
  - 詳細 log
```

##### 訊息模板變數替換
```typescript
const personalizedMessage = template
  .replace(/\{\{userId\}\}/g, user.userId)
  .replace(/\{\{userName\}\}/g, user.displayName)
  .replace(/\{\{name\}\}/g, user.displayName)
  .replace(/\{\{displayName\}\}/g, user.displayName)
```

#### 4.5.5 再行銷分析 (RemarketingAnalytics)
**組件路徑**: `components/admin/RemarketingAnalytics.tsx`

**功能**:
- 各規則發送統計
- 成功率分析
- 時間趨勢圖
- ROI 計算（若有轉換追蹤）

**資料來源**:
- API: `/api/admin/remarketing/analytics`
- 查詢 `remarketingLogs` collection

---

### 4.6 群發訊息（Tab 4）

#### 4.6.1 功能概述
允許管理員向特定群組的用戶發送大量訊息，並提供測試發送功能。

#### 4.6.2 UI 結構
```
群發訊息 Tab
├── 訊息內容區
│   ├── 訊息編輯器（Textarea）
│   ├── 變數提示
│   │   - {{userName}}: 用戶名稱
│   │   - {{userId}}: 用戶 ID
│   └── 字數統計
├── 目標選擇
│   ├── 全部好友
│   ├── 已確認收款
│   └── 已繳費（含待確認和已確認）
├── 目標用戶統計
│   - 顯示符合條件的用戶數量
├── 操作按鈕
│   ├── 測試發送（開啟測試對話框）
│   └── 正式群發
└── 訊息範本庫（可選）
    - 預設常用訊息範本
```

#### 4.6.3 測試發送功能

##### 測試對話框
**功能**:
- 從用戶列表選擇 1-5 位測試用戶
- 預覽將發送的訊息（含變數替換）
- 確認後僅發送給選中的測試用戶

##### 流程
```
點擊「測試發送」
    ↓
彈出測試對話框
    ↓
顯示所有用戶列表
    ↓
勾選測試用戶（Checkbox）
    ↓
顯示發送預覽
    ↓
點擊「發送測試訊息」
    ↓
逐一發送給選中用戶
    ↓
顯示發送結果（成功 X 位 / 失敗 Y 位）
```

#### 4.6.4 正式群發流程
```
填寫訊息內容
    ↓
選擇目標群組
    ↓
點擊「群發訊息」
    ↓
顯示確認對話框
  - 目標群組名稱
  - 目標用戶數量
  - 訊息預覽
    ↓
確認後開始發送
    ↓
顯示進度條
    ↓
逐一發送（含變數替換）
    ↓
完成後顯示結果
  - 總計發送數
  - 成功數
  - 失敗數
  - 失敗用戶列表（若有）
```

#### 4.6.5 發送限制與安全機制
- **速率限制**: 每秒最多 5 則訊息（避免 LINE API 限制）
- **失敗重試**: 失敗訊息不自動重試（記錄到 log）
- **取消發送**: 發送過程中可中斷（已發送的無法撤回）

---

### 4.7 訊息管理中心

#### 4.7.1 功能概述
**路由**: `/admin/messages`

提供完整的訊息發送與管理功能，包含個別發送、群發、訊息歷史與統計分析。

#### 4.7.2 頁面結構
```
訊息管理中心
├── Tab Navigation
│   ├── 發送訊息
│   ├── 訊息紀錄
│   └── 統計分析
└── Tab Content
```

#### 4.7.3 發送訊息 Tab

##### UI Layout
```
├── 左側區域（2/3 寬度）
│   ├── 訊息類型選擇
│   │   ├── 個別發送
│   │   └── 群發訊息
│   ├── 用戶選擇器（僅個別發送顯示）
│   │   ├── 搜尋框
│   │   └── 用戶列表（可選擇）
│   └── 訊息編輯器
│       ├── Textarea
│       ├── 字數統計
│       └── 發送按鈕
└── 右側區域（1/3 寬度）
    └── 訊息範本庫
        - 早鳥優惠提醒
        - 課程開始提醒
        - 匯款提醒
        - 課程資源分享
```

##### 訊息範本
系統預設 4 個訊息範本（定義於 `app/admin/messages/page.tsx`）:

1. **早鳥優惠提醒**
```
🔥 早鳥優惠最後倒數！

《Vibe Coding 工作術》課程早鳥優惠即將結束，把握最後機會享受 NT$ 3,000 的優惠價格！

原價 NT$ 4,500，現在報名立省 NT$ 1,500！

輸入「報名」立即開始報名流程 👉
```

2. **課程開始提醒**
```
📚 課程即將開始！

親愛的學員您好，

《Vibe Coding 工作術》將於 2025/10/12 開課，請記得準時參加！

📍 地點：臺北市中山區民權西路20號 2樓-204房
⏰ 時間：整日課程

期待與您見面！
```

3. **匯款提醒**
```
💳 匯款提醒

您好！我們注意到您對課程有興趣但尚未完成匯款。

匯款資訊：
銀行：玉山銀行 (808)
戶名：陳威達
帳號：0370968229188
金額：NT$ 3,000（超早鳥優惠價）

完成匯款後請回傳後五碼，謝謝！
```

4. **課程資源分享**
```
📚 本週學習資源

為您準備了本週的學習資源：

1. JavaScript 基礎教學影片
2. 實戰範例程式碼
3. 作業練習題

請至課程平台查看詳細內容！

有任何問題歡迎隨時詢問 😊
```

#### 4.7.4 訊息紀錄 Tab

##### 功能
- 顯示所有已發送的訊息歷史
- 篩選與搜尋
- 訊息狀態追蹤

##### 表格欄位
| 欄位 | 說明 | 資料來源 |
|------|------|---------|
| 時間 | 發送時間 | `sentAt` |
| 類型 | 個別/群發 | `type` |
| 接收者 | 用戶名稱或「所有用戶」 | `recipientName` |
| 內容 | 訊息內容（摘要） | `content` |
| 狀態 | 已發送/失敗/待發送 | `status` |

##### 狀態 Badge
```typescript
const statusBadge = {
  'sent': 'bg-green-100 text-green-800',
  'failed': 'bg-red-100 text-red-800',
  'pending': 'bg-yellow-100 text-yellow-800'
}
```

#### 4.7.5 統計分析 Tab

##### 統計卡片
1. **總發送訊息**
   - 累計所有訊息數量

2. **今日發送**
   - 今天發送的訊息數量

3. **成功率**
   - (成功訊息數 / 總訊息數) × 100%

##### 發送趨勢圖（開發中）
- 過去 30 天每日發送量
- 折線圖展示

---

### 4.8 即時聊天室

#### 4.8.1 功能概述
**路由**: `/admin/chat`

提供類似 LINE 官方帳號管理員介面的聊天功能，實現一對一即時溝通。

#### 4.8.2 頁面結構
```
聊天室
├── 左側邊欄（用戶列表）
│   ├── Header
│   │   ├── 標題「聊天室」
│   │   ├── 搜尋框
│   │   └── 標籤篩選（全部/已付款/有興趣）
│   └── 用戶列表
│       - 用戶頭像
│       - 用戶名稱
│       - 最後訊息預覽
│       - 最後訊息時間
│       - 未讀計數（紅點）
├── 中間區域（聊天視窗）
│   ├── 聊天 Header
│   │   ├── 用戶資訊
│   │   └── 操作按鈕（查看資訊/標記/封存）
│   ├── 訊息區域
│   │   - 訊息氣泡
│   │   - 時間戳記
│   │   - 已讀狀態（管理員訊息）
│   │   - Typing Indicator
│   ├── 快速回覆
│   │   - 常用回覆按鈕
│   └── 輸入區
│       ├── 附件按鈕
│       ├── 圖片按鈕
│       ├── Textarea
│       ├── Emoji 按鈕
│       └── 發送按鈕
└── 右側邊欄（用戶資訊）- 可展開
    ├── 用戶頭像與名稱
    ├── LINE ID
    ├── 標籤管理
    ├── 筆記欄
    └── 歷史統計
```

#### 4.8.3 訊息氣泡設計

##### 用戶訊息（左側）
- **背景色**: 白色 (`bg-white`)
- **文字色**: 深灰 (`text-gray-900`)
- **對齊**: 靠左
- **頭像**: 顯示於左側

##### 管理員訊息（右側）
- **背景色**: 紫色 (`bg-purple-600`)
- **文字色**: 白色 (`text-white`)
- **對齊**: 靠右
- **頭像**: 不顯示

##### 系統訊息（中間）
- **背景色**: 黃色 (`bg-yellow-100`)
- **文字色**: 深灰 (`text-gray-800`)
- **對齊**: 置中
- **用途**: 自動化訊息、狀態更新

#### 4.8.4 訊息狀態指示器

##### 已讀狀態（僅管理員訊息）
```typescript
const messageStatus = {
  sending: <Clock className="w-3 h-3 text-gray-400" />,
  sent: <CheckCircle className="w-3 h-3 text-gray-400" />,
  delivered: (
    <>
      <CheckCircle className="w-3 h-3 text-gray-400" />
      <CheckCircle className="w-3 h-3 text-gray-400 -ml-1" />
    </>
  ),
  read: (
    <>
      <CheckCircle className="w-3 h-3 text-blue-500" />
      <CheckCircle className="w-3 h-3 text-blue-500 -ml-1" />
    </>
  )
}
```

#### 4.8.5 即時更新機制

##### 輪詢機制
- **輪詢間隔**: 3 秒
- **API**: `GET /api/admin/chat/messages?userId=[userId]&lastMessageId=[lastMessageId]`
- **實作**: 使用 `setInterval` 定期查詢新訊息
- **優化**: 僅返回 `lastMessageId` 之後的新訊息

##### 程式碼實作
```typescript
const startMessagePolling = (userId: string) => {
  if (pollingInterval.current) {
    clearInterval(pollingInterval.current)
  }

  pollingInterval.current = setInterval(() => {
    fetchMessages(userId, true) // isPolling = true
  }, 3000)
}

// Cleanup on unmount or user switch
useEffect(() => {
  return () => {
    if (pollingInterval.current) {
      clearInterval(pollingInterval.current)
    }
  }
}, [selectedUser])
```

#### 4.8.6 快速回覆功能

##### 預設快速回覆
```typescript
const quickReplies = [
  '感謝您的詢問！',
  '課程資訊請輸入「報名」',
  '有什麼可以幫助您的嗎？',
  '稍後會有專人為您服務',
  '請提供您的聯絡方式'
]
```

##### 使用方式
- 點擊快速回覆按鈕
- 訊息自動填入輸入框
- 可編輯後再發送

#### 4.8.7 Typing Indicator

##### 觸發條件
- 用戶傳送新訊息時
- 顯示 1 秒後消失

##### UI 設計
```
[用戶頭像] [三個跳動的小圓點]
```

---

## 5. 數據模型

### 5.1 Firestore Collections 結構

#### 5.1.1 `users` Collection

**Collection Path**: `/users/{userId}`

**欄位定義**:
```typescript
{
  userId: string                  // LINE User ID（Document ID）
  displayName: string             // 用戶顯示名稱
  pictureUrl?: string             // 用戶頭像 URL
  statusMessage?: string          // LINE 狀態訊息
  followedAt: Timestamp           // 追蹤時間（加入 LINE Bot 時間）
  isFollowing: boolean            // 是否仍在追蹤（未封鎖）
  lastActivity: Timestamp         // 最後活動時間
  registrationStatus?: string     // 報名狀態: 'interested' | 'paid' | 'confirmed'
  paymentInfo?: {
    amount?: number               // 付款金額
    last5Digits?: string          // 匯款後五碼
    ticketType?: string           // 票種: 'regular' | 'invited' | 'complimentary'
    submittedAt?: Timestamp       // 提交匯款資訊時間
    confirmedAt?: Timestamp       // 確認收款時間
  }
  tags?: string[]                 // 用戶標籤（用於分類與篩選）
  notes?: string                  // 管理員備註
  firstViewedRegistrationAt?: Timestamp  // 第一次查看報名資訊的時間
}
```

**索引（Indexes）**:
```
- isFollowing (ASC)
- registrationStatus (ASC)
- followedAt (ASC/DESC)
- lastActivity (ASC/DESC)
- tags (ARRAY)
```

#### 5.1.2 `activities` Collection

**Collection Path**: `/activities/{activityId}`

**欄位定義**:
```typescript
{
  userId: string                  // 用戶 ID（關聯到 users）
  displayName: string             // 用戶名稱（冗餘，便於查詢）
  pictureUrl?: string             // 用戶頭像
  message: string                 // 活動訊息內容
  timestamp: Timestamp            // 活動時間
  type: string                    // 活動類型: 'message' | 'registration' | 'payment_confirm' | 'general_message'
}
```

**索引（Indexes）**:
```
- userId (ASC) + timestamp (DESC)
```

**用途**:
- 用戶活動歷史記錄
- 用於「用戶活動」面板顯示
- 追蹤用戶行為軌跡

#### 5.1.3 `remarketingRules` Collection

**Collection Path**: `/remarketingRules/{ruleId}`

**欄位定義**:
```typescript
{
  name: string                    // 規則名稱
  description: string             // 規則說明
  triggerDays: number             // 觸發天數（加入後幾天）
  targetConditions: {
    hasInteracted: boolean        // 是否需要有互動
    registrationStatus?: string[] // 目標註冊狀態
    notInStatus?: string[]        // 排除的狀態
  }
  messageTemplate: string         // 訊息模板 ID
  customMessage?: string          // 自訂訊息（若不使用模板）
  isActive: boolean               // 是否啟用
  createdAt: Timestamp            // 建立時間
  lastExecuted?: Timestamp        // 最後執行時間
}
```

#### 5.1.4 `remarketingLogs` Collection

**Collection Path**: `/remarketingLogs/{logId}`

**欄位定義**:
```typescript
{
  userId: string                  // 用戶 ID
  ruleId: string                  // 規則 ID
  ruleName: string                // 規則名稱
  sentAt: Timestamp               // 發送時間
  status: string                  // 發送狀態: 'success' | 'failed' | 'skipped'
  reason?: string                 // 失敗或跳過的原因
}
```

**索引（Indexes）**:
```
- userId (ASC) + sentAt (DESC)
- ruleId (ASC) + sentAt (DESC)
```

**用途**:
- 防止重複發送同一規則訊息
- 追蹤再行銷效果
- 提供再行銷分析資料

#### 5.1.5 `messages` Collection（可選）

**Collection Path**: `/messages/{messageId}`

**欄位定義**:
```typescript
{
  type: string                    // 訊息類型: 'broadcast' | 'individual' | 'reply'
  recipientId?: string            // 接收者 ID（個別訊息）
  recipientName?: string          // 接收者名稱
  content: string                 // 訊息內容
  sentAt: Timestamp               // 發送時間
  sentBy: string                  // 發送者（管理員 ID）
  status: string                  // 發送狀態: 'sent' | 'failed' | 'pending'
  replyTo?: string                // 回覆的訊息 ID
}
```

**用途**:
- 訊息發送記錄
- 訊息管理中心的「訊息紀錄」功能

#### 5.1.6 `chatMessages` Collection（可選）

**Collection Path**: `/chatMessages/{messageId}`

**欄位定義**:
```typescript
{
  userId: string                  // 用戶 ID
  userName: string                // 用戶名稱
  userPicture?: string            // 用戶頭像
  content: string                 // 訊息內容
  timestamp: Timestamp            // 發送時間
  type: string                    // 訊息類型: 'user' | 'admin' | 'system'
  status?: string                 // 狀態（僅管理員訊息）: 'sending' | 'sent' | 'delivered' | 'read'
  attachments?: string[]          // 附件 URL（未來功能）
}
```

**索引（Indexes）**:
```
- userId (ASC) + timestamp (ASC)
```

**用途**:
- 聊天室訊息記錄
- 支援訊息歷史查詢

---

### 5.2 數據關聯圖

```
users (1) ──< (N) activities
  │
  ├──< (N) remarketingLogs
  │
  ├──< (N) messages (recipientId)
  │
  └──< (N) chatMessages

remarketingRules (1) ──< (N) remarketingLogs
```

---

## 6. API 規格

### 6.1 認證 API

#### 6.1.1 登入
**Endpoint**: `POST /api/admin/auth/login`

**Request Body**:
```json
{
  "password": "vibe-admin-2024"
}
```

**Response (Success)**:
```json
{
  "success": true
}
```
- **Set-Cookie**: `admin-auth=[token]; Path=/; HttpOnly; SameSite=Lax`

**Response (Error)**:
```json
{
  "success": false,
  "error": "密碼錯誤"
}
```

#### 6.1.2 登出
**Endpoint**: `POST /api/admin/auth/logout`

**Response**:
```json
{
  "success": true
}
```
- **Set-Cookie**: `admin-auth=; Max-Age=0`

---

### 6.2 用戶管理 API

#### 6.2.1 取得用戶列表
**Endpoint**: `GET /api/admin/users`

**Query Parameters**:
| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| `isFollowing` | boolean | 否 | 篩選追蹤狀態 |
| `registrationStatus` | string | 否 | 篩選報名狀態 |
| `tag` | string | 否 | 篩選包含特定標籤的用戶 |

**Example Request**:
```
GET /api/admin/users?isFollowing=true&registrationStatus=confirmed
```

**Response**:
```json
{
  "users": [
    {
      "userId": "U1234567890abcdef",
      "displayName": "王小明",
      "pictureUrl": "https://profile.line-scdn.net/...",
      "followedAt": "2024-09-15T08:30:00.000Z",
      "isFollowing": true,
      "lastActivity": "2024-09-20T14:22:00.000Z",
      "registrationStatus": "confirmed",
      "paymentInfo": {
        "amount": 3000,
        "last5Digits": "12345",
        "ticketType": "regular",
        "confirmedAt": "2024-09-20T10:00:00.000Z"
      },
      "tags": ["paid", "early_bird"]
    }
  ]
}
```

#### 6.2.2 取得用戶活動記錄
**Endpoint**: `GET /api/admin/users/[userId]/activities`

**Response**:
```json
{
  "activities": [
    {
      "id": "act_001",
      "userId": "U1234567890abcdef",
      "displayName": "王小明",
      "message": "查詢報名資訊",
      "timestamp": "2024-09-15T09:00:00.000Z",
      "type": "registration"
    }
  ]
}
```

#### 6.2.3 確認付款
**Endpoint**: `POST /api/admin/users/[userId]/confirm-payment`

**Request Body**:
```json
{
  "status": "confirmed",
  "paymentInfo": {
    "amount": 3000,
    "last5Digits": "12345",
    "ticketType": "regular"
  }
}
```

**Response**:
```json
{
  "success": true
}
```

**Side Effects**:
- 更新用戶 `registrationStatus` 為 `confirmed`
- 更新 `paymentInfo.confirmedAt`
- 發送 LINE 確認訊息給用戶

#### 6.2.4 清除用戶資料
**Endpoint**: `DELETE /api/admin/users/[userId]/clear`

**Response**:
```json
{
  "success": true
}
```

**功能**:
- 重置用戶的 `registrationStatus` 與 `paymentInfo`
- 保留用戶基本資料與活動記錄

#### 6.2.5 取得用戶再行銷歷史
**Endpoint**: `GET /api/admin/users/[userId]/remarketing-history`

**Response**:
```json
{
  "history": [
    {
      "id": "log_001",
      "userId": "U1234567890abcdef",
      "ruleId": "rule_001",
      "ruleName": "第1天 - 歡迎提醒",
      "sentAt": "2024-09-16T09:00:00.000Z",
      "status": "success"
    }
  ]
}
```

---

### 6.3 訊息管理 API

#### 6.3.1 取得訊息列表
**Endpoint**: `GET /api/admin/messages`

**Response**:
```json
{
  "messages": [
    {
      "id": "msg_001",
      "type": "individual",
      "recipientId": "U1234567890abcdef",
      "recipientName": "王小明",
      "content": "感謝您的報名！",
      "sentAt": "2024-09-20T10:00:00.000Z",
      "sentBy": "admin",
      "status": "sent"
    }
  ]
}
```

#### 6.3.2 發送訊息
**Endpoint**: `POST /api/admin/messages/send`

**Request Body**:
```json
{
  "type": "individual",
  "recipientId": "U1234567890abcdef",
  "content": "您好，這是測試訊息"
}
```

**Response**:
```json
{
  "success": true,
  "messageId": "msg_002"
}
```

**Side Effects**:
- 透過 LINE Messaging API 發送訊息
- 記錄至 `messages` collection（若有實作）

---

### 6.4 聊天室 API

#### 6.4.1 取得聊天訊息
**Endpoint**: `GET /api/admin/chat/messages`

**Query Parameters**:
| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| `userId` | string | 是 | 用戶 ID |
| `lastMessageId` | string | 否 | 最後已讀訊息 ID（用於輪詢） |

**Example Request**:
```
GET /api/admin/chat/messages?userId=U1234567890abcdef&lastMessageId=msg_100
```

**Response**:
```json
{
  "messages": [
    {
      "id": "msg_101",
      "userId": "U1234567890abcdef",
      "userName": "王小明",
      "userPicture": "https://...",
      "content": "請問課程何時開始？",
      "timestamp": "2024-09-20T14:30:00.000Z",
      "type": "user"
    }
  ]
}
```

#### 6.4.2 發送聊天訊息
**Endpoint**: `POST /api/admin/chat/messages`

**Request Body**:
```json
{
  "userId": "U1234567890abcdef",
  "content": "課程將於 2025/10/12 開始",
  "userName": "王小明",
  "userPicture": "https://..."
}
```

**Response**:
```json
{
  "success": true,
  "messageId": "msg_102"
}
```

**Side Effects**:
- 發送 LINE 訊息
- 儲存至 `chatMessages` collection
- 同時調用 `/api/admin/messages/send` 發送 LINE 訊息

---

### 6.5 再行銷 API

#### 6.5.1 取得再行銷規則
**Endpoint**: `GET /api/remarketing/rules`

**Response**:
```json
{
  "rules": [
    {
      "id": "rule_001",
      "name": "第1天 - 歡迎提醒",
      "description": "用戶加入後第1天，若未查詢報名資訊，發送歡迎並介紹課程",
      "triggerDays": 1,
      "targetConditions": {
        "hasInteracted": false,
        "notInStatus": ["paid", "confirmed"]
      },
      "messageTemplate": "welcome_day1",
      "isActive": true,
      "createdAt": "2024-09-01T00:00:00.000Z",
      "lastExecuted": "2024-09-20T09:00:00.000Z"
    }
  ]
}
```

#### 6.5.2 新增再行銷規則
**Endpoint**: `POST /api/remarketing/rules`

**Request Body**:
```json
{
  "name": "第10天 - 最終優惠",
  "description": "最後優惠提醒",
  "triggerDays": 10,
  "targetConditions": {
    "hasInteracted": true,
    "notInStatus": ["paid", "confirmed"]
  },
  "messageTemplate": "final_offer",
  "isActive": true
}
```

**Response**:
```json
{
  "success": true,
  "ruleId": "rule_005"
}
```

#### 6.5.3 更新再行銷規則
**Endpoint**: `PUT /api/remarketing/rules`

**Request Body**:
```json
{
  "id": "rule_001",
  "isActive": false
}
```

**Response**:
```json
{
  "success": true
}
```

#### 6.5.4 刪除再行銷規則
**Endpoint**: `DELETE /api/remarketing/rules?id=[ruleId]`

**Response**:
```json
{
  "success": true
}
```

#### 6.5.5 執行再行銷
**Endpoint**: `POST /api/remarketing/execute`

**Request Headers**:
```
Authorization: Bearer vibe-admin-2024
```

**Response**:
```json
{
  "success": true,
  "executedAt": "2024-09-20T09:00:00.000Z",
  "summary": {
    "totalProcessed": 50,
    "successful": 48,
    "failed": 2
  },
  "logs": [
    {
      "userId": "U1234567890abcdef",
      "ruleId": "rule_001",
      "ruleName": "第1天 - 歡迎提醒",
      "sentAt": "2024-09-20T09:00:05.000Z",
      "status": "success"
    }
  ]
}
```

---

### 6.6 分析 API

#### 6.6.1 每日用戶統計
**Endpoint**: `GET /api/admin/analytics/daily-users`

**Query Parameters**:
| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|
| `days` | number | 否 | 查詢天數（預設 30） |

**Response**:
```json
{
  "stats": [
    {
      "date": "2024-09-01",
      "newUsers": 5,
      "totalUsers": 100
    },
    {
      "date": "2024-09-02",
      "newUsers": 8,
      "totalUsers": 108
    }
  ]
}
```

#### 6.6.2 再行銷分析
**Endpoint**: `GET /api/admin/remarketing/analytics`

**Response**:
```json
{
  "analytics": {
    "totalSent": 500,
    "successRate": 96.5,
    "byRule": [
      {
        "ruleId": "rule_001",
        "ruleName": "第1天 - 歡迎提醒",
        "sent": 150,
        "successful": 148,
        "failed": 2
      }
    ],
    "conversionRate": {
      "totalTargeted": 500,
      "converted": 45,
      "rate": 9.0
    }
  }
}
```

---

### 6.7 LINE Bot Webhook API

#### 6.7.1 Webhook 接收
**Endpoint**: `POST /api/linebot/webhook`

**Request Headers**:
```
X-Line-Signature: [LINE 簽章]
Content-Type: application/json
```

**Request Body**:
```json
{
  "events": [
    {
      "type": "message",
      "replyToken": "abc123...",
      "source": {
        "userId": "U1234567890abcdef",
        "type": "user"
      },
      "message": {
        "type": "text",
        "id": "msg_001",
        "text": "報名"
      },
      "timestamp": 1695201600000
    }
  ]
}
```

**Response**:
```json
{
  "success": true
}
```

**處理邏輯**:
1. 驗證 LINE 簽章
2. 解析事件類型（message / follow / unfollow）
3. 根據訊息內容路由到對應 handler
4. 回覆用戶或記錄活動

**Handler 路由**:
```typescript
const handlers = {
  '報名': registrationHandler,
  '付款': paymentHandler,
  '已完成匯款': paymentHandler,
  '取消': cancelHandler,
  // ... 其他關鍵字
}
```

---

## 7. UI/UX 設計規範

### 7.1 設計系統

#### 7.1.1 色彩規範

##### 主色調
```css
/* 紫色（主要品牌色） */
--purple-50: #FAF5FF;
--purple-100: #F3E8FF;
--purple-600: #9333EA;  /* 主按鈕、連結 */
--purple-700: #7E22CE;  /* Hover 狀態 */

/* 灰色（背景與文字） */
--gray-50: #F9FAFB;     /* 淺背景 */
--gray-100: #F3F4F6;    /* 卡片背景 */
--gray-500: #6B7280;    /* 次要文字 */
--gray-900: #111827;    /* 主要文字 */
```

##### 狀態色
```css
/* 成功 */
--green-100: #D1FAE5;
--green-600: #10B981;
--green-800: #065F46;

/* 警告 */
--yellow-100: #FEF3C7;
--yellow-600: #F59E0B;
--yellow-800: #92400E;

/* 錯誤 */
--red-100: #FEE2E2;
--red-600: #EF4444;
--red-800: #991B1B;

/* 資訊 */
--blue-100: #DBEAFE;
--blue-600: #3B82F6;
--blue-800: #1E40AF;
```

#### 7.1.2 字體規範

##### 字型
```css
font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen',
  'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue',
  sans-serif;
```

##### 字級
```css
/* 標題 */
.text-3xl { font-size: 1.875rem; }  /* 30px - 頁面標題 */
.text-2xl { font-size: 1.5rem; }    /* 24px - 區塊標題 */
.text-xl { font-size: 1.25rem; }    /* 20px - 卡片標題 */
.text-lg { font-size: 1.125rem; }   /* 18px - 次標題 */

/* 內文 */
.text-base { font-size: 1rem; }     /* 16px - 預設內文 */
.text-sm { font-size: 0.875rem; }   /* 14px - 小字 */
.text-xs { font-size: 0.75rem; }    /* 12px - 輔助文字 */
```

#### 7.1.3 間距規範

```css
/* Spacing Scale (Tailwind) */
.p-2  { padding: 0.5rem; }   /* 8px */
.p-4  { padding: 1rem; }     /* 16px */
.p-6  { padding: 1.5rem; }   /* 24px */
.p-8  { padding: 2rem; }     /* 32px */

.gap-2 { gap: 0.5rem; }      /* 8px */
.gap-4 { gap: 1rem; }        /* 16px */
.gap-6 { gap: 1.5rem; }      /* 24px */
```

#### 7.1.4 圓角規範

```css
.rounded-lg { border-radius: 0.5rem; }   /* 8px - 卡片、按鈕 */
.rounded-xl { border-radius: 0.75rem; }  /* 12px - 大卡片 */
.rounded-2xl { border-radius: 1rem; }    /* 16px - Modal */
.rounded-full { border-radius: 9999px; } /* 圓形 - 頭像、Badge */
```

#### 7.1.5 陰影規範

```css
.shadow-sm {
  box-shadow: 0 1px 2px 0 rgb(0 0 0 / 0.05);
}

.shadow {
  box-shadow: 0 1px 3px 0 rgb(0 0 0 / 0.1), 0 1px 2px -1px rgb(0 0 0 / 0.1);
}

.shadow-lg {
  box-shadow: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1);
}

.shadow-2xl {
  box-shadow: 0 25px 50px -12px rgb(0 0 0 / 0.25);
}
```

---

### 7.2 組件設計規範

#### 7.2.1 按鈕 (Button)

##### 主要按鈕（Primary）
```tsx
<button className="px-6 py-2 bg-purple-600 text-white font-medium rounded-lg hover:bg-purple-700 transition-all">
  確認
</button>
```

##### 次要按鈕（Secondary）
```tsx
<button className="px-6 py-2 bg-gray-200 text-gray-700 font-medium rounded-lg hover:bg-gray-300 transition-all">
  取消
</button>
```

##### 危險按鈕（Danger）
```tsx
<button className="px-6 py-2 bg-red-600 text-white font-medium rounded-lg hover:bg-red-700 transition-all">
  刪除
</button>
```

##### 停用狀態
```tsx
<button className="px-6 py-2 bg-gray-300 text-gray-500 font-medium rounded-lg cursor-not-allowed" disabled>
  處理中...
</button>
```

#### 7.2.2 輸入框 (Input)

##### 文字輸入
```tsx
<input
  type="text"
  placeholder="請輸入..."
  className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-purple-500 focus:border-transparent"
/>
```

##### 搜尋框（含 Icon）
```tsx
<div className="relative">
  <Search className="absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-400 w-5 h-5" />
  <input
    type="text"
    placeholder="搜尋..."
    className="w-full pl-10 pr-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-purple-500"
  />
</div>
```

##### Textarea
```tsx
<textarea
  placeholder="請輸入訊息..."
  className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-purple-500 resize-none"
  rows={4}
/>
```

#### 7.2.3 卡片 (Card)

##### 基本卡片
```tsx
<div className="bg-white rounded-lg shadow p-6">
  <h3 className="text-lg font-medium text-gray-900 mb-4">卡片標題</h3>
  <p className="text-gray-600">卡片內容...</p>
</div>
```

##### 統計卡片
```tsx
<div className="bg-white rounded-lg shadow p-6">
  <div className="flex items-center justify-between mb-2">
    <span className="text-sm text-gray-500">總用戶數</span>
    <Users className="w-5 h-5 text-purple-600" />
  </div>
  <p className="text-2xl font-bold text-gray-900">150</p>
  <p className="text-sm text-green-600 mt-2">↑ 12% vs 上週</p>
</div>
```

#### 7.2.4 Badge

##### 狀態 Badge
```tsx
<span className="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-green-100 text-green-800">
  已確認
</span>
```

##### 計數 Badge
```tsx
<span className="bg-red-500 text-white text-xs rounded-full px-2 py-0.5">
  5
</span>
```

#### 7.2.5 表格 (Table)

```tsx
<div className="overflow-x-auto">
  <table className="w-full">
    <thead className="bg-gray-50">
      <tr>
        <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
          用戶名稱
        </th>
      </tr>
    </thead>
    <tbody className="bg-white divide-y divide-gray-200">
      <tr>
        <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
          王小明
        </td>
      </tr>
    </tbody>
  </table>
</div>
```

#### 7.2.6 Modal / Dialog

```tsx
<div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
  <div className="bg-white rounded-2xl shadow-2xl p-6 max-w-md w-full">
    <h3 className="text-xl font-bold text-gray-900 mb-4">對話框標題</h3>
    <p className="text-gray-600 mb-6">對話框內容...</p>
    <div className="flex justify-end gap-3">
      <button className="px-4 py-2 bg-gray-200 text-gray-700 rounded-lg">
        取消
      </button>
      <button className="px-4 py-2 bg-purple-600 text-white rounded-lg">
        確認
      </button>
    </div>
  </div>
</div>
```

#### 7.2.7 Tooltip

**組件路徑**: `components/ui/tooltip.tsx`

**使用方式**:
```tsx
<TooltipProvider>
  <Tooltip>
    <TooltipTrigger>
      <button>Hover me</button>
    </TooltipTrigger>
    <TooltipContent>
      <p>提示內容</p>
    </TooltipContent>
  </Tooltip>
</TooltipProvider>
```

---

### 7.3 響應式設計

#### 7.3.1 斷點規範

```css
/* Tailwind Breakpoints */
sm: 640px   /* 平板直向 */
md: 768px   /* 平板橫向 */
lg: 1024px  /* 筆電 */
xl: 1280px  /* 桌機 */
2xl: 1536px /* 大螢幕 */
```

#### 7.3.2 響應式 Grid

```tsx
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
  {/* 手機 1 欄 / 平板 2 欄 / 桌機 3 欄 */}
</div>
```

---

### 7.4 無障礙設計

#### 7.4.1 語意化 HTML
- 使用正確的 HTML 標籤 (`<button>`, `<nav>`, `<main>`)
- 為互動元素提供 `aria-label`

#### 7.4.2 鍵盤導航
- 所有互動元素可透過 Tab 鍵導航
- 提供明顯的 focus 樣式

#### 7.4.3 顏色對比
- 文字與背景對比度至少 4.5:1
- 重要資訊不僅依賴顏色傳達

---

## 8. 第三方集成

### 8.1 LINE Messaging API

#### 8.1.1 基本資訊
- **官方文檔**: https://developers.line.biz/en/docs/messaging-api/
- **SDK**: `@line/bot-sdk`

#### 8.1.2 環境變數配置

```env
LINE_CHANNEL_ACCESS_TOKEN=your_channel_access_token
LINE_CHANNEL_SECRET=your_channel_secret
```

#### 8.1.3 Client 初始化

**檔案**: `lib/line/client.ts`

```typescript
import { Client } from '@line/bot-sdk';

const config = {
  channelAccessToken: process.env.LINE_CHANNEL_ACCESS_TOKEN || '',
  channelSecret: process.env.LINE_CHANNEL_SECRET || ''
};

export const getLineClient = () => new Client(config);
```

#### 8.1.4 常用功能

##### 發送文字訊息
```typescript
import { getLineClient } from '@/lib/line/client';

const client = getLineClient();

await client.pushMessage(userId, {
  type: 'text',
  text: '您好，這是一則測試訊息'
});
```

##### 發送 Flex Message
```typescript
await client.pushMessage(userId, {
  type: 'flex',
  altText: '課程報名資訊',
  contents: {
    type: 'bubble',
    // ... Flex Message JSON
  }
});
```

##### 回覆訊息
```typescript
await client.replyMessage(replyToken, {
  type: 'text',
  text: '收到您的訊息了！'
});
```

##### 取得用戶資訊
```typescript
const profile = await client.getProfile(userId);
console.log(profile.displayName);
console.log(profile.pictureUrl);
```

#### 8.1.5 Webhook 簽章驗證

```typescript
import { validateSignature } from '@line/bot-sdk';

const signature = req.headers['x-line-signature'];
const body = await req.text();

if (!validateSignature(body, channelSecret, signature)) {
  return NextResponse.json({ error: 'Invalid signature' }, { status: 401 });
}
```

---

### 8.2 Firebase

#### 8.2.1 基本資訊
- **服務**: Firestore (Database)
- **SDK**: `firebase-admin`

#### 8.2.2 環境變數配置

```env
FIREBASE_PROJECT_ID=your_project_id
FIREBASE_PRIVATE_KEY_ID=your_private_key_id
FIREBASE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n"
FIREBASE_CLIENT_EMAIL=your_client_email
FIREBASE_CLIENT_ID=your_client_id
FIREBASE_AUTH_URI=https://accounts.google.com/o/oauth2/auth
FIREBASE_TOKEN_URI=https://oauth2.googleapis.com/token
FIREBASE_AUTH_PROVIDER_CERT_URL=https://www.googleapis.com/oauth2/v1/certs
FIREBASE_CLIENT_CERT_URL=your_client_cert_url
FIREBASE_STORAGE_BUCKET=your_project_id.appspot.com
NEXT_PUBLIC_FIREBASE_DATABASE_URL=https://your_project_id-default-rtdb.firebaseio.com
```

#### 8.2.3 Admin SDK 初始化

**檔案**: `lib/firebase/admin.ts`

```typescript
import * as admin from 'firebase-admin';

let firebaseApp: admin.app.App;

export const initFirebase = () => {
  if (!firebaseApp) {
    const serviceAccount = {
      type: 'service_account',
      project_id: process.env.FIREBASE_PROJECT_ID,
      private_key: process.env.FIREBASE_PRIVATE_KEY?.replace(/\\n/g, '\n'),
      client_email: process.env.FIREBASE_CLIENT_EMAIL,
      // ...其他欄位
    };

    firebaseApp = admin.initializeApp({
      credential: admin.credential.cert(serviceAccount as any),
      databaseURL: process.env.NEXT_PUBLIC_FIREBASE_DATABASE_URL,
      storageBucket: process.env.FIREBASE_STORAGE_BUCKET
    });
  }
  return firebaseApp;
};

export const getFirestore = () => {
  initFirebase();
  return admin.firestore();
};
```

#### 8.2.4 Firestore 操作範例

##### 新增文檔
```typescript
const db = getFirestore();
await db.collection('users').doc(userId).set({
  displayName: 'John Doe',
  email: 'john@example.com'
});
```

##### 查詢文檔
```typescript
const snapshot = await db.collection('users')
  .where('isFollowing', '==', true)
  .get();

const users = snapshot.docs.map(doc => ({
  id: doc.id,
  ...doc.data()
}));
```

##### 更新文檔
```typescript
await db.collection('users').doc(userId).update({
  lastActivity: admin.firestore.Timestamp.now()
});
```

##### 刪除文檔
```typescript
await db.collection('users').doc(userId).delete();
```

---

### 8.3 Vercel Cron Jobs

#### 8.3.1 配置檔案

**檔案**: `vercel.json`

```json
{
  "crons": [
    {
      "path": "/api/marketing/cron",
      "schedule": "0 9 * * *"
    }
  ]
}
```

**說明**:
- **path**: Cron Job 調用的 API 路徑
- **schedule**: Cron 表達式（`0 9 * * *` = 每天 09:00）

#### 8.3.2 Cron API 實作

**檔案**: `app/api/marketing/cron/route.ts`

```typescript
import { executeRemarketingRules } from '@/lib/line/remarketingService';

export async function GET(req: Request) {
  // 驗證 Cron Job 請求（可選）
  const authHeader = req.headers.get('authorization');
  if (authHeader !== `Bearer ${process.env.CRON_SECRET}`) {
    return Response.json({ error: 'Unauthorized' }, { status: 401 });
  }

  // 執行再行銷
  const result = await executeRemarketingRules();

  return Response.json(result);
}
```

#### 8.3.3 Cron 表達式參考

```
┌───────────── 分鐘 (0 - 59)
│ ┌─────────── 小時 (0 - 23)
│ │ ┌───────── 日期 (1 - 31)
│ │ │ ┌─────── 月份 (1 - 12)
│ │ │ │ ┌───── 星期 (0 - 7，0 和 7 都是星期日)
│ │ │ │ │
* * * * *
```

**常用範例**:
- `0 9 * * *` - 每天 09:00
- `0 */6 * * *` - 每 6 小時
- `0 9 * * 1` - 每週一 09:00
- `0 0 1 * *` - 每月 1 號 00:00

---

## 9. 環境配置

### 9.1 環境變數清單

建立 `.env.local` 檔案於專案根目錄：

```env
# Admin 密碼
ADMIN_PASSWORD=vibe-admin-2024

# LINE Messaging API
LINE_CHANNEL_ACCESS_TOKEN=your_line_channel_access_token
LINE_CHANNEL_SECRET=your_line_channel_secret

# Firebase Admin SDK
FIREBASE_PROJECT_ID=your_firebase_project_id
FIREBASE_PRIVATE_KEY_ID=your_firebase_private_key_id
FIREBASE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\nYour Private Key Here\n-----END PRIVATE KEY-----\n"
FIREBASE_CLIENT_EMAIL=your_firebase_client_email
FIREBASE_CLIENT_ID=your_firebase_client_id
FIREBASE_AUTH_URI=https://accounts.google.com/o/oauth2/auth
FIREBASE_TOKEN_URI=https://oauth2.googleapis.com/token
FIREBASE_AUTH_PROVIDER_CERT_URL=https://www.googleapis.com/oauth2/v1/certs
FIREBASE_CLIENT_CERT_URL=your_firebase_client_cert_url
FIREBASE_STORAGE_BUCKET=your_project_id.appspot.com
NEXT_PUBLIC_FIREBASE_DATABASE_URL=https://your_project_id-default-rtdb.firebaseio.com

# 再行銷設定
ENABLE_AUTO_REMARKETING=true

# Cron Job Secret（可選）
CRON_SECRET=your_cron_secret_key
```

### 9.2 取得環境變數教學

#### 9.2.1 LINE Messaging API

1. 前往 [LINE Developers Console](https://developers.line.biz/)
2. 建立 Provider 與 Messaging API Channel
3. 在 Channel 設定頁取得：
   - `Channel Secret`
   - `Channel Access Token`（需手動發行）
4. 設定 Webhook URL: `https://your-domain.com/api/linebot/webhook`

#### 9.2.2 Firebase

1. 前往 [Firebase Console](https://console.firebase.google.com/)
2. 建立專案
3. 啟用 Firestore Database
4. 前往「專案設定」→「服務帳戶」
5. 點擊「產生新的私密金鑰」
6. 下載 JSON 檔案，內含所有需要的環境變數

---

### 9.3 本地開發設定

#### 9.3.1 安裝依賴

```bash
npm install
```

#### 9.3.2 啟動開發伺服器

```bash
npm run dev
```

伺服器將運行於 `http://localhost:3000`

#### 9.3.3 測試 LINE Webhook（本地）

使用 [ngrok](https://ngrok.com/) 建立公開 URL:

```bash
ngrok http 3000
```

將 ngrok 提供的 HTTPS URL 設定為 LINE Webhook URL:
```
https://abc123.ngrok.io/api/linebot/webhook
```

---

## 10. 部署指南

### 10.1 Vercel 部署

#### 10.1.1 準備工作
1. 將專案推送至 GitHub
2. 註冊 [Vercel](https://vercel.com/) 帳號
3. 連結 GitHub 帳號

#### 10.1.2 部署步驟

1. **匯入專案**
   - 在 Vercel Dashboard 點擊「Add New Project」
   - 選擇 GitHub Repository
   - 點擊「Import」

2. **設定環境變數**
   - 在「Environment Variables」區塊
   - 逐一新增所有 `.env.local` 中的變數
   - 選擇適用環境（Production / Preview / Development）

3. **部署設定**
   - Framework Preset: Next.js
   - Root Directory: `./`
   - Build Command: `npm run build`
   - Output Directory: `.next`

4. **部署**
   - 點擊「Deploy」
   - 等待建置完成

#### 10.1.3 設定 Cron Jobs

1. 確認 `vercel.json` 已正確設定
2. Vercel 會自動偵測並啟用 Cron Jobs
3. 在 Vercel Dashboard 的「Deployments」→「Cron Jobs」查看執行狀態

#### 10.1.4 設定自訂網域（可選）

1. 前往 Vercel 專案設定
2. 點擊「Domains」
3. 新增自訂網域
4. 依照指示設定 DNS

---

### 10.2 LINE Webhook 設定

部署完成後，更新 LINE Webhook URL:

```
https://your-domain.vercel.app/api/linebot/webhook
```

**測試 Webhook**:
```bash
curl -X POST https://your-domain.vercel.app/api/linebot/test
```

---

### 10.3 防火牆與安全設定

#### 10.3.1 環境變數加密
- 所有敏感資訊儲存於 Vercel 環境變數（加密）
- 不要將 `.env.local` 提交至 Git

#### 10.3.2 API 路由保護
- Admin API 透過 Middleware 驗證
- Webhook 透過 LINE Signature 驗證
- Cron API 可選用 Bearer Token 驗證

#### 10.3.3 CORS 設定（如需）
```typescript
// app/api/*/route.ts
export async function GET(req: Request) {
  const response = NextResponse.json(data);
  response.headers.set('Access-Control-Allow-Origin', '*');
  return response;
}
```

---

## 11. 系統維護與監控

### 11.1 日誌管理

#### 11.1.1 Vercel Logs
- 前往 Vercel Dashboard → 專案 → Deployments
- 點擊特定部署查看即時 Logs
- 支援篩選與搜尋

#### 11.1.2 應用層日誌
所有重要操作都有 `console.log` / `console.error`：
- 用戶活動記錄
- API 錯誤
- 再行銷執行結果
- LINE Webhook 事件

**建議**: 整合第三方日誌服務（如 Sentry, LogRocket）

---

### 11.2 錯誤處理

#### 11.2.1 全域錯誤邊界
```tsx
// app/error.tsx
'use client';

export default function Error({
  error,
  reset,
}: {
  error: Error;
  reset: () => void;
}) {
  return (
    <div className="min-h-screen flex items-center justify-center">
      <div className="text-center">
        <h2 className="text-2xl font-bold text-red-600 mb-4">發生錯誤</h2>
        <p className="text-gray-600 mb-4">{error.message}</p>
        <button onClick={reset} className="px-4 py-2 bg-purple-600 text-white rounded-lg">
          重試
        </button>
      </div>
    </div>
  );
}
```

#### 11.2.2 API 錯誤處理
```typescript
export async function POST(req: Request) {
  try {
    // ... 處理邏輯
  } catch (error) {
    console.error('API Error:', error);
    return NextResponse.json(
      { error: 'Internal Server Error', details: (error as Error).message },
      { status: 500 }
    );
  }
}
```

---

### 11.3 效能監控

#### 11.3.1 Vercel Analytics
- 自動整合
- 追蹤 Page Views, Unique Visitors
- Core Web Vitals 指標

#### 11.3.2 建議整合工具
- **Google Analytics**: 用戶行為分析
- **Sentry**: 錯誤追蹤
- **Hotjar**: 使用者體驗分析

---

## 12. 常見問題與故障排除

### 12.1 登入問題

**問題**: 無法登入管理後台

**檢查項目**:
1. 確認環境變數 `ADMIN_PASSWORD` 已設定
2. 清除瀏覽器 Cookie
3. 檢查 Middleware 是否正常運作

---

### 12.2 LINE Bot 無回應

**問題**: 用戶傳送訊息後無回應

**檢查項目**:
1. 確認 LINE Webhook URL 已正確設定
2. 檢查 Webhook 簽章驗證是否通過
3. 查看 Vercel Logs 是否有錯誤訊息
4. 確認 `LINE_CHANNEL_ACCESS_TOKEN` 正確

**測試指令**:
```bash
curl -X POST https://your-domain.com/api/linebot/test
```

---

### 12.3 再行銷未執行

**問題**: Cron Job 未自動發送再行銷訊息

**檢查項目**:
1. 確認 `vercel.json` 已正確設定
2. 檢查環境變數 `ENABLE_AUTO_REMARKETING=true`
3. 前往 Vercel Dashboard 查看 Cron Jobs 執行記錄
4. 確認至少有一個 `isActive=true` 的規則

**手動測試**:
```bash
curl -X POST https://your-domain.com/api/remarketing/execute \
  -H "Authorization: Bearer vibe-admin-2024"
```

---

### 12.4 Firestore 連線失敗

**問題**: 無法讀取或寫入 Firestore

**檢查項目**:
1. 確認所有 Firebase 環境變數已正確設定
2. 確認 `FIREBASE_PRIVATE_KEY` 包含完整的私鑰（含換行符）
3. 檢查 Firebase 專案是否已啟用 Firestore
4. 確認 Service Account 權限正確

**除錯方式**:
- 查看 Vercel Logs 的 Firebase 初始化訊息
- 測試 API: `GET /api/test/firebase`

---

## 13. 未來優化建議

### 13.1 功能增強

1. **多管理員支援**
   - 角色權限管理（Admin / Editor / Viewer）
   - 個人化 Session

2. **進階數據分析**
   - Cohort Analysis（同期群分析）
   - A/B Testing 框架
   - 預測分析（機器學習）

3. **自動化工作流程**
   - 視覺化規則編輯器（Drag & Drop）
   - 多步驟自動化（Automation Workflows）

4. **多媒體支援**
   - 聊天室支援圖片、影片、檔案
   - 批次上傳與管理

5. **整合其他通訊平台**
   - Facebook Messenger
   - Instagram DM
   - Email

---

### 13.2 技術優化

1. **效能提升**
   - 實作 Redis Cache（快取用戶資料）
   - 使用 SWR 或 React Query 優化前端資料請求
   - 圖片 CDN 與 Lazy Loading

2. **即時通訊**
   - 將輪詢改為 WebSocket 或 Server-Sent Events
   - 即時通知（Push Notifications）

3. **資料庫優化**
   - 建立複合索引加速查詢
   - 定期清理過期資料
   - 資料備份機制

4. **測試覆蓋率**
   - 單元測試（Jest）
   - 整合測試（Playwright）
   - E2E 測試

5. **CI/CD Pipeline**
   - GitHub Actions 自動化測試
   - 自動部署至 Staging 環境
   - 程式碼品質檢查（ESLint, Prettier）

---

## 14. 結語

本 PRD 文檔詳細描述了 Vibe Coding 課程管理後台系統的所有功能、技術架構與實作細節。依照本文檔，開發者可以從零開始重建一個功能完全相同的系統。

### 快速開始檢查清單
- [ ] 複製專案並安裝依賴 (`npm install`)
- [ ] 設定 `.env.local` 環境變數
- [ ] 初始化 Firebase Firestore
- [ ] 設定 LINE Bot Webhook
- [ ] 本地測試 (`npm run dev`)
- [ ] 部署至 Vercel
- [ ] 設定 Cron Jobs
- [ ] 驗證所有功能正常運作

### 支援與聯絡
如有任何問題，請參考：
- 原始 Repository: [GitHub URL]
- 文檔更新日誌: [CHANGELOG.md]
- Issue Tracker: [GitHub Issues]

---

**文檔結束**
