---
name: FlightPHP 最佳化開發指南
description: 基於 flightphp/core 原始碼的 API 參考與最佳化開發指南，確保 AI 產出高品質 FlightPHP 程式碼
---

# FlightPHP 最佳化開發指南

> **目標讀者**：AI 編碼助手。此 Skill 提供基於原始碼的精確 API 用法，避免生成不存在的 API 或錯誤的呼叫方式。

## 必讀：6 條鐵律

在撰寫任何 FlightPHP 程式碼前，務必確認：

1. **靜態 vs 實例**: `Flight::xxx()` 是靜態呼叫、`$this->app->xxx()` 是 Engine 實例呼叫，兩者方法完全相同
2. **中間件必須有 `before()` 方法**：Flight 用 `method_exists($middleware, 'before')` 檢查
3. **回應不自動中止**：`Flight::json()` 不會停止執行，要停止必須用 `Flight::jsonHalt()` 或手動 `return`
4. **Controller 建構子自動注入 Engine**：無需手動實例化
5. **資料庫請用 `SimplePdo`**：`PdoWrapper` 已 deprecated
6. **路由參數用 `@` 前綴**：`@id`、`@name` 等，不是 `{id}`

---

## 框架核心架構

### 類別總覽

```
flight/
├── Engine.php           # 核心引擎 (所有功能的中心)
├── Flight.php           # 靜態代理 (透過 __callStatic 轉發到 Engine)
├── autoload.php         # 自動載入器
├── core/
│   ├── Dispatcher.php   # 事件分派器 (before/after hook, DI container)
│   ├── EventDispatcher.php  # 事件系統 (on/trigger，Singleton)
│   └── Loader.php       # 類別載入器
├── database/
│   ├── SimplePdo.php    # ✅ 新版資料庫封裝 (推薦使用)
│   └── PdoWrapper.php   # ❌ 已 deprecated
├── net/
│   ├── Request.php      # HTTP 請求封裝
│   ├── Response.php     # HTTP 回應封裝
│   ├── Route.php        # 單一路由物件
│   ├── Router.php       # 路由器
│   └── UploadedFile.php # 上傳檔案
├── template/
│   └── View.php         # 模板引擎
└── util/
    ├── Collection.php   # 集合（支援陣列/物件雙存取）
    └── Json.php         # JSON 編碼/解碼工具
```

### 請求生命週期

```
index.php
  → require Flight.php
  → require bootstrap.php   (設定、註冊元件)
  → require routes.php      (定義路由)
  → Flight::start()
      → Engine::_start()
          → before('start') filter 執行
          → Router::route(Request) 匹配路由
          → 執行 before middleware
          → 執行路由 callback / Controller 方法
          → 執行 after middleware
          → Response::send()
```

---

## 詳細指南目錄

遇到特定主題時，請讀取對應的 guide 文件以取得完整參考：

### 核心 API

| 指南 | 檔案 | 涵蓋內容 |
|------|------|----------|
| 路由系統 | `guides/routing.md` | 基本路由、參數、群組、別名、RESTful Resource、串流 |
| Controller | `guides/controller.md` | 基本結構、請求資料取得、服務注入模式 |
| 回應系統 | `guides/response.md` | json/jsonHalt/halt、重導向、下載、HTTP 快取 |
| 中間件 | `guides/middleware.md` | 結構、執行順序、常見使用情境（Auth、API Key、角色） |
| 資料庫 | `guides/database.md` | SimplePdo 方法、本專案 PDO 模式、交易、IN 查詢 |
| 框架擴充 | `guides/extending.md` | map/register、set/get、before/after filter、事件系統 |

### 進階主題

| 指南 | 檔案 | 涵蓋內容 |
|------|------|----------|
| 專案架構 | `guides/project-architecture.md` | 目錄結構、三層架構、回應格式、範例模板、命名規範 |
| 依賴注入 | `guides/dependency-injection.md` | DIC 概念、本專案 DI 模式、進階 Dice/PHP-DI |
| 單元測試 | `guides/unit-testing.md` | PHPUnit 設定、Controller 測試、Mock、避免過度 Mock |

### 速查與檢查

| 指南 | 檔案 | 用途 |
|------|------|------|
| API 速查卡 | `guides/cheatsheet.md` | 所有方法簽名速查 |
| 反模式檢查 | `guides/anti-patterns.md` | 產出程式碼前的自我檢查清單 |
| 開發工作流 | `guides/dev-workflow.md` | 端點分析、逐步實作流程、命名規範 |

---

## 快速範例：標準 CRUD 端點

```php
// routes.php
Flight::group('/api/v4', function () {
    Flight::group('/examples', function () {
        Flight::get('', [ExampleController::class, 'index']);
        Flight::get('/@id', [ExampleController::class, 'show']);
        Flight::post('', [ExampleController::class, 'create']);
        Flight::put('/@id', [ExampleController::class, 'update']);
        Flight::delete('/@id', [ExampleController::class, 'destroy']);
    }, [AuthMiddleware::class]);
});
```

> 完整的 Controller/Service/Repository 模板請參見 `guides/project-architecture.md`。
