---
name: api-v4
description: API v4 開發工作流與 Agent 指揮中心。當需要分析舊檔案、撰寫 v4 端點、審閱 v4 程式碼時觸發。
---

# API v4 開發工作流

> 此 Skill 管理 v4 API 的開發流程：從分析舊檔案到產出可用的 FlightPHP 端點。
> 框架 API 細節請參考 `flightphp` skill。

## 專案概況

**目標**：將 `modules/*/prodb_*.php` 舊功能遷移為 FlightPHP RESTful API。

**技術棧**：
- 框架：FlightPHP（`ADLAPI/v4/libs/core/`）
- 架構：Controller → Service → Repository（三層分離）
- 認證：AuthMiddleware（Session-based）
- 資料庫：`Flight::db()`（寫入）/ `Flight::dbSlave()`（讀取）

**已完成端點**（routes.php）：
- `/health` - 健康檢查
- `/mission-types` - 任務類型
- `/student/missions` - 學生任務卡
- `/parent/children` - 家長子女列表
- `/calendar/*` - 行事曆 CRUD
- `/user/config` - 使用者設定

## 核心檔案路徑

```
ADLAPI/v4/
├── public/index.php              # 入口
├── config/
│   ├── bootstrap.php             # 初始化（DB、CORS、Helper）
│   └── routes.php                # 路由定義
└── src/App/
    ├── Controller/               # HTTP 層（薄）
    ├── Service/                  # 業務邏輯
    ├── Repository/               # SQL 查詢
    └── Middleware/                # 認證、參數驗證
```

---

## 三步工作流

```
舊 PHP 檔案 (modules/*/prodb_*.php)
    │
    ▼
┌─────────────────────────────────┐
│  Step 1: /api-v4-analyze        │
│  分析舊檔案 → 端點對照表        │
│  輸出: 端點規劃、參數、SQL 位置  │
└─────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────┐
│  Step 2: /api-v4-write          │
│  逐一實作端點                    │
│  順序: Repository → Service      │
│       → Controller → routes.php │
└─────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────┐
│  Step 3: /api-v4-review         │
│  程式碼審閱                      │
│  檢查: 分層、安全、一致性       │
└─────────────────────────────────┘
```

### 何時使用哪個 Agent

| 情境 | Agent |
|------|-------|
| 拿到一個舊 prodb 檔案，要規劃怎麼遷移 | `/api-v4-analyze` |
| 已有規劃，要開始寫程式碼 | `/api-v4-write` |
| 程式碼寫好了，要檢查品質 | `/api-v4-review` |
| 想了解 FlightPHP 框架用法 | 參考 `flightphp` skill |

---

## 開發鐵律

1. **一次一個端點** — 逐一討論、逐一實作、逐一驗證
2. **SQL 標位置不複製** — 只標註舊檔案行號，由用戶確認後貼入
3. **資料庫連線用現有的** — `Flight::db()` / `Flight::dbSlave()`，不建新連線
4. **Session 已可用** — `index.php` 已 require `config.php`，`$_SESSION` 直接使用
5. **三層職責嚴格分離** — Controller 不寫 SQL、Repository 不處理 HTTP、Service 不碰 request/response

## 命名規範

| 項目 | 規範 | 範例 |
|------|------|------|
| Controller | PascalCase + Controller | `CalendarController` |
| Service | PascalCase + Service | `CalendarService` |
| Repository | PascalCase + Repository | `CalendarRepository` |
| 路由 | kebab-case, RESTful | `GET /calendar/events` |
| 方法名 | camelCase | `getEventsByDateRange` |
| 檔案位置 | 對應 namespace | `src/App/Controller/CalendarController.php` |

## 回應格式

```php
// 成功
Flight::success($data);           // {"success":true,"data":{...}}
Flight::success($data, 201);      // 新增成功

// 分頁
Flight::success(['items'=>$rows, 'meta'=>$meta]);
// {"success":true,"data":[...],"meta":{...}}

// 失敗
Flight::fail('錯誤訊息', 400);    // {"success":false,"error":{"code":400,"message":"..."}}

// 例外（Service 層拋出，bootstrap 全域攔截）
throw new \Exception('資料不存在', 404);
```
