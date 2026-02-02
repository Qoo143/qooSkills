---
name: API-V3 Development
description: API-V3 架構開發指南，包含 Controller、Service、DAO、Helper 層的開發規範與最佳實踐
---

# API-V3 Development Skill

本技能提供 API-V3 開發的完整指南，包含架構設計、分層職責、開發流程及最佳實踐。

## 專案架構概述

### 核心目標
將既有的舊功能統一轉換為 RESTful API 形式，達成：
- **統一入口**：所有請求經由單一入口 (`index.php`) 處理
- **標準化回應**：統一 JSON 回應格式與錯誤處理
- **分層架構**：將業務邏輯 (Service)、資料存取 (DAO)、請求處理 (Controller) 明確分離
- **安全強化**：統一的 Token 驗證與輸入過濾機制

### 目錄結構
```
v3/
├── Bootstrap/autoload.php        # 自動載入器
├── Core/                         # 框架核心 (通用、不依賴業務)
│   ├── App.php                   # Express 風格應用程式入口
│   ├── MiddlewareInterface.php   # 中間件介面
│   ├── Pipeline.php              # 中間件管線執行器
│   ├── Request.php               # HTTP 請求封裝
│   ├── Response.php              # API 回應處理
│   ├── Router.php                # 路由分發器 (慣例式)
│   └── Validator.php             # 參數驗證器
├── Public/index.php              # API 入口點
└── App/                          # 應用程式層 (業務相關)
    ├── Controller/               # 控制器
    ├── Service/                  # 商業邏輯
    ├── Dao/                      # 資料存取
    ├── Helper/                   # 共用函式 (工具類別)
    ├── Middleware/               # 全域中間件
    └── Exception/                # 業務異常
```

## 分層職責

> **說明**：Core 雖然不在 `App/` 目錄下，但作為框架基礎設施被所有層依賴，因此列入職責表中。

| 層級 | 位置 | 職責 | 不應該做 |
|------|------|------|----------|
| **Core** | `Core/` | 框架基礎設施，不依賴業務邏輯 | 不包含業務邏輯 |
| **Middleware** | `App/Middleware/` | 驗證/過濾/錯誤處理 | 不處理業務邏輯 |
| **Controller** | `App/Controller/` | 接收參數、呼叫 Service、回傳 JSON | 不包含業務邏輯、不直接操作資料庫 |
| **Service** | `App/Service/` | 商業邏輯、業務驗證、流程控制 | 不處理 HTTP 請求、不直接寫 SQL |
| **DAO** | `App/Dao/` | 資料庫 CRUD、SQL 查詢 | 不包含業務邏輯、不處理異常 |
| **Helper** | `App/Helper/` | 可重用的工具函式、資料轉換、格式化 | 不包含業務邏輯、不操作資料庫 |
| **Exception** | `App/Exception/` | 業務異常定義（AppException） | 只定義異常類別，不包含業務邏輯 |

### 依賴方向
```
Core ← App/Middleware ← App/Controller → App/Service → App/Dao
                                    ↓          ↓
                                 Helper ←──────┘
```

> **Helper 使用原則**：Controller 和 Service 都可以使用 Helper，但 Helper 不應依賴其他層級

## 開發步驟範例

### Controller 範例

```php
<?php
namespace App\Controller;

use App\Service\XxxService;

class XxxController extends BaseController
{
    private $xxxService;

    public function __construct($request, $response)
    {
        parent::__construct($request, $response);
        $this->xxxService = new XxxService();
    }

    public function query()
    {
        $this->guard(['method' => 'POST', 'token' => true]);
        $params = $this->validate(['id' => 'required|integer']);
        $userId = $this->request->getUserId();
        $result = $this->xxxService->getById($userId, $params['id']);
        $this->success(['data' => $result], '查詢成功');
    }
}
```

### Service 範例

```php
<?php
namespace App\Service;

use App\Dao\XxxDao;
use App\Helper\DateHelper;
use App\Exception\AppException;

class XxxService
{
    private $xxxDao;

    public function __construct()
    {
        $this->xxxDao = new XxxDao();
    }

    public function getById($userId, $id)
    {
        $existing = $this->xxxDao->getById($userId, $id);
        if (!$existing) {
            throw new AppException('資料不存在', 404);
        }
        
        // 使用 Helper 格式化資料
        $existing['roc_date'] = DateHelper::toRocDate($existing['created_at']);
        
        return $existing;
    }
}
```

### DAO 範例

```php
<?php
namespace App\Dao;

class XxxDao
{
    private $db;
    private $dbSlave;

    public function __construct()
    {
        global $dbh, $dbh_slave;
        $this->db = $dbh;
        $this->dbSlave = $dbh_slave;
    }

    public function getById($userId, $id)
    {
        $sql = "SELECT * FROM xxx_table WHERE user_id = :userId AND id = :id";
        $stmt = $this->dbSlave->prepare($sql);
        $stmt->execute([':userId' => $userId, ':id' => $id]);
        return $stmt->fetch(\PDO::FETCH_ASSOC);
    }
}
```

### Helper 範例

```php
<?php
namespace App\Helper;

class DateHelper
{
    /** 日期時間轉民國年格式 */
    public static function toRocDate($datetime)
    {
        if (empty($datetime)) return '';
        $timestamp = strtotime($datetime);
        $year = (int)date('Y', $timestamp) - 1911;
        return $year . '/' . date('m/d', $timestamp);
    }
}

class ArrayHelper
{
    /** 安全取得陣列值 */
    public static function get($array, $key, $default = null)
    {
        return isset($array[$key]) ? $array[$key] : $default;
    }
}
```

### 使用者權限與身份判斷

#### BaseController 提供的方法

BaseController 提供統一的使用者資料查詢方法，這些方法會自動快取資料到 Request 物件中：

```php
// Controller 中使用
class XxxController extends BaseController
{
    public function someMethod()
    {
        $this->guard(['method' => 'POST', 'token' => true]);

        // 取得 access_level (從 user_status 表，會快取到 Request)
        $accessLevel = $this->getAccessLevel();

        // 取得 organization_id (從 user_info 表，會快取到 Request)
        $orgId = $this->getOrganizationId();

        // 取得年級 (從 user_info 表)
        $grade = $this->getGrade();
    }
}
```

#### Service 層使用 UserDao 和 UserHelper

**原則**：
- **UserDao**：負責從資料庫查詢使用者資料（`getUserInfo`、`getUserStatus`）
- **UserHelper**：負責工具函數，如將 `access_level` 轉換為身份字串

```php
// Service 層範例
namespace App\Service;

use App\Dao\UserDao;
use App\Helper\UserHelper;

class AnnouncementService
{
    private $userDao;

    public function __construct()
    {
        $this->userDao = new UserDao();
    }

    private function getUserIdentity($userId)
    {
        // 1. 使用 UserDao::getUserStatus() 查詢 user_status 表
        $userStatus = $this->userDao->getUserStatus($userId);

        if ($userStatus === null || !isset($userStatus['access_level'])) {
            return 'student';
        }

        // 2. 使用 UserHelper 轉換 access_level 為身份字串
        $accessLevel = intval($userStatus['access_level']);
        return UserHelper::getIdentityByAccessLevel($accessLevel);
    }
}
```

#### Access Level 對應表

| Access Level | 身份 | 說明 |
|--------------|------|------|
| 1, 4, 9 | `student` | 學生 |
| 11 | `parent` | 家長 |
| 21, 25 | `teacher` | 教師 |
| 22, 23, 24 | `principal` | 校長 |
| 26-33 | `schoolAdmin` | 學校管理員 |

#### UserHelper::getIdentityByAccessLevel()

**位置**：`App/Helper/UserHelper.php`

**用途**：將 `access_level` 數值轉換為身份字串

**簽名**：
```php
public static function getIdentityByAccessLevel(int $accessLevel): string
```

**回傳值**：`'student'` | `'teacher'` | `'parent'` | `'principal'` | `'schoolAdmin'`

**使用時機**：
- ✅ Service 層需要根據身份判斷業務邏輯
- ✅ 需要將數值轉換為語意化的身份字串
- ❌ 不要在 Controller 中直接使用（Controller 應使用 `$this->getAccessLevel()` 取得數值）

**最佳實踐**：
1. **Controller 層**：使用 `$this->getAccessLevel()` 取得數值，快取在 Request 中
2. **Service 層**：使用 `UserDao::getUserStatus()` + `UserHelper::getIdentityByAccessLevel()` 組合
3. **避免重複查詢**：user_status 表的資料應一次查詢，從返回的陣列中取得 access_level

## 核心規範

### Read-before-Write 模式
> **執行 Update 或 Delete 操作前，必須先查詢資料。**

```php
public function update($userId, $params)
{
    // 1. Read：檢查資料是否存在
    $existing = $this->xxxDao->getById($userId, $params['id']);
    if (!$existing) {
        throw new AppException('資料不存在', 404);
    }

    // 2. 合併資料（支援部分更新）
    $data = [
        'field1' => $params['field1'] ?? $existing['field1'],
       'field2' => $params['field2'] ?? $existing['field2'],
    ];

    // 3. Write：執行更新
    $this->xxxDao->update($userId, $params['id'], $data);
    return ['id' => $params['id']];
}
```

### 錯誤處理

| 層級 | 錯誤處理方式 |
|------|--------------|
| **DAO** | 不處理異常，讓 PDOException 自動冒泡 |
| **Service** | 使用 `throw new AppException('訊息', HTTP狀態碼)` |
| **Controller** | 不需要 try-catch，由 ErrorHandlerMiddleware 統一處理 |

### 命名規則

| Namespace | 路徑 |
|-----------|------|
| `App\Controller\XxxController` | `App/Controller/XxxController.php` |
| `App\Service\XxxService` | `App/Service/XxxService.php` |
| `App\Dao\XxxDao` | `App/Dao/XxxDao.php` |
| `App\Helper\XxxHelper` | `App/Helper/XxxHelper.php` |
| `App\Exception\AppException` | `App/Exception/AppException.php` |

## 開發檢查清單

### Controller 檢查
- [ ] 繼承 `BaseController`
- [ ] 使用 `guard()` 驗證 HTTP 方法和 Token
- [ ] 使用 `validate()` 驗證參數
- [ ] 不包含業務邏輯
- [ ] 使用 `success()` 回傳結果

### Service 檢查
- [ ] Update/Delete 使用 Read-before-Write 模式
- [ ] 使用 `AppException` 處理業務錯誤
- [ ] 不使用 try-catch 捕捉 PDOException

### DAO 檢查
- [ ] 使用 Prepared Statements
- [ ] 讀取使用 `$dbSlave`，寫入使用 `$db`
- [ ] 不包含業務邏輯
- [ ] 不處理異常

### Helper 檢查
- [ ] 使用靜態方法
- [ ] 不包含業務邏輯
- [ ] 不操作資料庫
- [ ] 不依賴其他層級

## API 對照表管理

### 對照表目錄

所有 API 轉換對照表存放在：`.agent/SKILL/api-v3/api-mapping/`

每個舊 PHP 檔案對應一個獨立的對照表檔案。

### 命名規則

```
api-mapping-{舊檔案名稱}.md

範例：
- api-mapping-modules_student.md
- api-mapping-modules_teacher.md
```

### 對照表結構

| 選擇 | 舊端點（函式名稱） | Controller 方法 | 狀態 | Controller 類型 | 說明 |
|------|------------------|----------------|------|----------------|------|
| [x] | `getList()` | `list()` | ✅ 已完成 | TodoController（現有） | 已完成 |
| [ ] | `getDetail()` | `detail()` | ⏳ 待處理 | StudentController（新建） | 需新建 |

### 使用流程

1. **建立對照表**（`/api-mapping-table`）：分析舊檔案，列出端點，勾選要轉換的項目
2. **撰寫 API**（`/api-write`）：按 DAO → Service → Controller 順序開發
3. **Code Review**（`/api-review`）：檢查程式碼品質

### 狀態說明

- `⏳ 待處理`：尚未開始
- `🚧 進行中`：正在開發
- `✅ 已完成`：已完成並測試

### Controller 類型

- **現有**：寫入已存在的 Controller
- **新建**：需要建立新的 Controller

---

## Workflow 指南

本專案提供了一系列自動化工作流 (Workflow)，協助開發者規範化地進行 API 遷移與開發。

### 1. 建立對照表 (`/api-mapping-table`)

**用途**：分析舊 PHP 檔案，規劃 API 遷移藍圖。

**使用時機**：
- 準備遷移一個舊的 PHP 頁面 (如 `modules_student.php`)。
- 需要釐清舊程式碼有哪些功能點需要轉換。

**執行內容**：
- 掃描舊檔案。
- 建立 `api-mapping-{name}.md`。
- 列出舊功能點 (Function/Logic) 與新 Controller 方法的對應。

**下一步**：
- 執行 `/api-verify-mapping` 驗證規劃是否合理。

### 2. 驗證對照表 (`/api-verify-mapping`)

**用途**：檢查對照表與現有程式碼的落差 (Gap Analysis)。

**使用時機**：
- 完成對照表初稿後。
- 懷疑現有 Controller 無法滿足舊邏輯時 (例如：小組長權限、特殊篩選)。

**執行內容**：
- 深度比對舊檔案邏輯 (參數、權限、SQL)。
- 檢查 Controller 實作是否涵蓋所有邏輯。
- 修正對照表或標註 "待實作"。

**下一步**：
- 執行 `/api-write` 開始寫程式。

### 3. 撰寫程式碼 (`/api-write`)

**用途**：根據對照表，標準化地實作 API 代碼。

**使用時機**：
- 對照表已確認無誤 (`✅` 或 `🚧`)。
- 準備開始寫 DAO, Service, Controller。

**執行內容**：
- 依照 **DAO → Service → Controller** 順序開發。
- 自動引用 `BaseController`, `AppException` 等規範。
- 實作 Read-before-Write 模式。

**下一步**：
- 完成後，更新對照表狀態為 `✅`。
- 執行 `/api-review` 進行檢查。

### 4. 程式碼審查 (`/api-review`)

**用途**：確保程式碼符合架構規範與品質要求。

**使用時機**：
- API 功能開發完成，準備提交前。

**執行內容**：
- 檢查分層職責 (Controller 不含邏輯, DAO 不含邏輯)。
- 檢查驗證機制 (guard, validate)。
- 檢查錯誤處理 (不吃掉 Exception)。
- 檢查 SQL 安全性。

---
