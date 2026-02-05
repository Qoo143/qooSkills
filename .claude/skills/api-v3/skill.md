---
name: api-v3
description: API-V3 架構開發指南，包含 Controller、Service、DAO、Helper 層的開發規範與最佳實踐
---

# API-V3 Development Guide

本文件定義 API-V3 的架構規範、分層職責與程式碼標準。

## 專案架構概述

### 核心目標

將既有功能統一轉換為 RESTful API，達成：
- 統一入口：所有請求經由 `index.php` 處理
- 標準化回應：統一 JSON 格式與錯誤處理
- 分層架構：業務邏輯、資料存取、請求處理明確分離
- 安全強化：統一 Token 驗證與輸入過濾

### 目錄結構

```
v3/
├── Bootstrap/autoload.php        # 自動載入器
├── Core/                         # 框架核心（通用、不依賴業務）
│   ├── App.php                   # Express 風格應用程式入口
│   ├── MiddlewareInterface.php   # 中間件介面
│   ├── Pipeline.php              # 中間件管線執行器
│   ├── Request.php               # HTTP 請求封裝
│   ├── Response.php              # API 回應處理
│   ├── Router.php                # 路由分發器（慣例式）
│   └── Validator.php             # 參數驗證器
├── Public/index.php              # API 入口點
└── App/                          # 應用程式層（業務相關）
    ├── Controller/               # 控制器
    ├── Service/                  # 商業邏輯
    ├── Dao/                      # 資料存取
    ├── Helper/                   # 共用函式（工具類別）
    ├── Middleware/               # 全域中間件
    └── Exception/                # 業務異常
```

## 分層職責

| 層級 | 位置 | 職責 | 禁止事項 |
|------|------|------|----------|
| Core | `Core/` | 框架基礎設施 | 不包含業務邏輯 |
| Middleware | `App/Middleware/` | 驗證、過濾、錯誤處理 | 不處理業務邏輯 |
| Controller | `App/Controller/` | 接收參數、呼叫 Service、回傳 JSON | 不包含業務邏輯、不直接操作資料庫 |
| Service | `App/Service/` | 商業邏輯、業務驗證、流程控制 | 不處理 HTTP 請求、不直接寫 SQL |
| DAO | `App/Dao/` | 資料庫 CRUD、SQL 查詢 | 不包含業務邏輯、不處理異常 |
| Helper | `App/Helper/` | 可重用工具函式、資料轉換 | 不包含業務邏輯、不操作資料庫 |
| Exception | `App/Exception/` | 業務異常定義 | 只定義異常類別 |

### 依賴方向

```
Core <- App/Middleware <- App/Controller -> App/Service -> App/Dao
                                    |              |
                                 Helper <----------+
```

Helper 使用原則：Controller 和 Service 都可使用 Helper，但 Helper 不應依賴其他層級。

---

## 程式碼範例

### Controller

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

### Service

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
        $existing['roc_date'] = DateHelper::toRocDate($existing['created_at']);
        return $existing;
    }
}
```

### DAO

所有 DAO 必須繼承 `BaseDao`，統一管理資料庫連線：

```php
<?php
namespace App\Dao;

/**
 * Xxx 資料存取層
 */
class XxxDao extends BaseDao
{
    public function getById($userId, $id)
    {
        $sql = "SELECT * FROM xxx_table WHERE user_id = :userId AND id = :id";
        $stmt = $this->dbSlave->prepare($sql);
        $stmt->execute([':userId' => $userId, ':id' => $id]);
        return $stmt->fetch(\PDO::FETCH_ASSOC);
    }
}
```

**BaseDao 提供的屬性**：
- `$this->db`：主資料庫連線（寫入用）
- `$this->dbSlave`：從資料庫連線（讀取用）

**子目錄 DAO**（如 `App\Dao\Knowledge\NodeDao`）需加入 `use` 語句：

```php
<?php
namespace App\Dao\Knowledge;

use App\Dao\BaseDao;

class NodeDao extends BaseDao
{
    // ...
}
```

### Helper

```php
<?php
namespace App\Helper;

class DateHelper
{
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
    public static function get($array, $key, $default = null)
    {
        return isset($array[$key]) ? $array[$key] : $default;
    }
}
```

---

## 使用者權限與身份判斷

### BaseController 提供的方法

```php
class XxxController extends BaseController
{
    public function someMethod()
    {
        $this->guard(['method' => 'POST', 'token' => true]);
        $accessLevel = $this->getAccessLevel();
        $orgId = $this->getOrganizationId();
        $grade = $this->getGrade();
    }
}
```

### Service 層使用 UserDao 和 UserHelper

```php
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
        $userStatus = $this->userDao->getUserStatus($userId);
        if ($userStatus === null || !isset($userStatus['access_level'])) {
            return 'student';
        }
        $accessLevel = intval($userStatus['access_level']);
        return UserHelper::getIdentityByAccessLevel($accessLevel);
    }
}
```

### Access Level 對應表

身分常數定義於 `include/config.php`：

| 常數 | Access Level | 身份 |
|------|--------------|------|
| `USER_STUDENT_GROUP` | 1, 4, 9 | 學生 |
| - | 11 | 家長 |
| `USER_TEACHER_GROUP` | 21, 25 | 教師 |
| - | 22, 23, 24 | 校長 |
| - | 26-33 | 學校管理員 |

### 權限檢查位置

權限檢查應在 **Controller 層**執行，不符合權限時直接拒絕。

### UserHelper 身分驗證方法

`App\Helper\UserHelper` 提供以下公用方法：

| 方法 | 說明 |
|------|------|
| `isStudent(int $accessLevel): bool` | 檢查是否為學生 |
| `isTeacher(int $accessLevel): bool` | 檢查是否為教師 |
| `guardStudent(int $accessLevel): void` | 驗證學生身分，否則拋出 403 |
| `guardTeacher(int $accessLevel): void` | 驗證教師身分，否則拋出 403 |
| `getIdentityByAccessLevel(int $accessLevel): string` | 取得身分字串 |

### 使用方式（推薦）

```php
use App\Helper\UserHelper;

class MissionReportController extends BaseController
{
    // 學生專用端點
    public function list()
    {
        $this->guard(['method' => 'POST', 'token' => true]);
        UserHelper::guardStudent($this->getAccessLevel());

        // ... 業務邏輯
    }

    // 教師專用端點
    public function classReport()
    {
        $this->guard(['method' => 'POST', 'token' => true]);
        UserHelper::guardTeacher($this->getAccessLevel());

        // ... 業務邏輯
    }
}
```

### 自訂身分檢查

若需檢查其他身分（如家長），可手動判斷：

```php
public function students()
{
    $this->guard(['method' => 'POST', 'token' => true]);

    $accessLevel = $this->getAccessLevel();
    $identity = UserHelper::getIdentityByAccessLevel($accessLevel);

    if ($identity !== 'parent') {
        throw new AppException('僅限家長身份使用', 403);
    }

    // ... 業務邏輯
}
```

Service 層只處理業務邏輯，不做權限驗證。

---

## 核心規範

### Read-before-Write 模式

執行 Update 或 Delete 前，必須先查詢資料：

```php
public function update($userId, $params)
{
    $existing = $this->xxxDao->getById($userId, $params['id']);
    if (!$existing) {
        throw new AppException('資料不存在', 404);
    }

    $data = [
        'field1' => $params['field1'] ?? $existing['field1'],
        'field2' => $params['field2'] ?? $existing['field2'],
    ];

    $this->xxxDao->update($userId, $params['id'], $data);
    return ['id' => $params['id']];
}
```

### 錯誤處理

| 層級 | 處理方式 |
|------|----------|
| DAO | 不處理異常，讓 PDOException 自動冒泡 |
| Service | 使用 `throw new AppException('訊息', HTTP狀態碼)` |
| Controller | 不需要 try-catch，由 ErrorHandlerMiddleware 統一處理 |

### API 回應格式規範

API-V3 為純 API，回傳乾淨 JSON 結構：

#### 禁止事項

- **不產生舊式 URL**：不使用 `modules_new.php` 等前端路由
- **不做非必要 Hash**：ID 欄位直接回傳原始值，除非有安全性需求

#### 正確範例

```php
// ✅ 正確：回傳乾淨 JSON
return [
    'mission_sn' => $missionSn,
    'student_id' => $studentId,
    'exam_sn' => $examSn,
    'score' => 85,
    'date' => '2024-01-15'
];
```

#### 錯誤範例

```php
// ❌ 錯誤：產生舊式 URL
return [
    'url' => 'modules_new.php?op=modload&name=ExamResult&file=detail...'
];

// ❌ 錯誤：非必要的 Hash
return [
    'student_id' => string2hash($studentId),  // 除非有安全需求
    'mission_sn' => string2hash($missionSn)
];
```

#### 何時需要 Hash

| 情境 | 是否 Hash |
|------|-----------|
| API 內部資料傳遞 | ❌ 不需要 |
| 回傳給前端的一般 ID | ❌ 不需要 |
| 來自舊系統類別的參數（如 TeacherClasses） | ✅ 可能需要解碼 |
| 涉及安全性的敏感資料 | ✅ 視需求決定 |

### 命名規則

| Namespace | 路徑 |
|-----------|------|
| `App\Controller\XxxController` | `App/Controller/XxxController.php` |
| `App\Service\XxxService` | `App/Service/XxxService.php` |
| `App\Dao\XxxDao` | `App/Dao/XxxDao.php` |
| `App\Helper\XxxHelper` | `App/Helper/XxxHelper.php` |
| `App\Exception\AppException` | `App/Exception/AppException.php` |

---

## 開發檢查清單

### Controller

- [ ] 繼承 BaseController
- [ ] 使用 guard() 驗證 HTTP 方法和 Token
- [ ] 使用 validate() 驗證參數
- [ ] 不包含業務邏輯
- [ ] 使用 success() 回傳結果

### Service

- [ ] Update/Delete 使用 Read-before-Write 模式
- [ ] 使用 AppException 處理業務錯誤
- [ ] 不使用 try-catch 捕捉 PDOException

### DAO

- [ ] 繼承 BaseDao
- [ ] 不自行定義 __construct() 初始化資料庫連線
- [ ] 使用 Prepared Statements
- [ ] 讀取使用 dbSlave，寫入使用 db
- [ ] 不包含業務邏輯
- [ ] 不處理異常

### Helper

- [ ] 使用靜態方法
- [ ] 不包含業務邏輯
- [ ] 不操作資料庫
- [ ] 不依賴其他層級

---

## 對照表管理

### 存放位置

`.claude/skills/api-v3/api-mapping/`

### 命名規則

`api-mapping-{舊檔案名稱}.md`

### 對照表結構

| 選擇 | 舊端點 | Controller 方法 | 狀態 | Controller 類型 | 說明 |
|------|--------|----------------|------|----------------|------|
| [x] | getList() | list() | 已完成 | TodoController（現有） | - |
| [ ] | getDetail() | detail() | 待處理 | StudentController（新建） | - |

### 狀態定義

- 待處理：尚未開始
- 進行中：正在開發
- 已完成：已完成並測試

---

## 開發工作流程

使用以下指令依序執行開發流程：

| 順序 | 指令 | 目的 |
|------|------|------|
| 1 | `/api-mapping-table` | 分析舊檔案，建立對照表 |
| 2 | `/api-verify-mapping` | 驗證對照表完整性 |
| 3 | `/api-write` | 依 DAO、Service、Controller 順序撰寫程式碼 |
| 4 | `/api-review` | 審查程式碼品質與安全性 |

各指令啟動對應的 Agent 執行自動化任務。詳細執行邏輯由 Agent 處理。

> [!TIP]
> `/api-write` Agent 包含**前置檢查**步驟，會自動讀取 config 設定檔並檢查權限繼承規則。

### 資料表結構查詢

撰寫 SQL 前必須先查詢資料表結構：

```
/database
```

或讀取：`.claude/skills/database/tables/{table_name}.md`
