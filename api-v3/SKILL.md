---
name: api-v3
description: >
  v3 API 開發規範。當詢問 API、v3、endpoint、controller、
  service、DAO、validation 相關問題時觸發。
---

# v3 API 開發規範

## 任務類型判斷

收到任務時，先判斷屬於哪種類型：

| 類型 | 描述 | 輸入範例 |
|------|------|----------|
| **A. 對照表補齊** | 補齊舊程式與 API 的對應關係 | 使用者提供舊頁面 URL 或檔案路徑 |
| **B. 新 API 開發** | 改寫舊程式為新 API | 使用者要求開發新功能或改寫模組 |

---

## A. 對照表補齊流程

當使用者提供舊頁面 URL 要求建立對照表時：

### A1. 收集舊程式資訊

1. 解析 URL 取得模組名稱 (`name`) 和檔案 (`file`)
2. 定位原始 PHP 檔案：`modules/{name}/{file}.php`
3. 閱讀原始程式碼，識別：
   - 包含的 class 或 function
   - 依賴的其他檔案（如 `classes/*.php`）
   - 使用的 `$_SESSION` 變數
   - 所有功能點（查詢、新增、修改、刪除等）

### A2. 建立對照表文件

在 `api-mapping-reference/` 建立對照表：

```
.agent/skills/api-v3/api-mapping-reference/{模組名稱}.md
```

對照表格式請參考 [功能對照表格式規範](#功能對照表-api-mapping-reference)。

### A3. 交付

- 對照表建立後，標記「未對應功能」供後續開發參考
- 若使用者僅要求對照表，任務完成
- 若使用者同時要求開發 API，則進入 B 流程

---

## B. 新 API 開發流程

當使用者要求開發新 API 時：

### B1. 分析階段

1. **確認對照表存在**：檢查 `api-mapping-reference/` 是否已有該模組對照表
   - 若無，先執行 A 流程建立對照表
   - 若有，閱讀對照表理解現有對應關係
2. 閱讀原始 PHP 檔案，理解業務需求
3. 識別所有功能點（查詢、新增、修改、刪除等）
4. 列出需要的 API endpoints
5. 檢查是否使用 `$_SESSION`，記錄哪些 session 變數被使用
6. **查閱資料庫結構**：使用 `database` skill 查詢相關資料表結構
   - 路徑：`.agent/skills/database/SKILL.md`
   - 資料表定義：`.agent/skills/database/tables/{table_name}.md`

### B2. 計畫撰寫

**在開始實作前，必須先建立執行計畫：**

```
.agent/docs/plans/{模組名稱}.md
```

計畫內容應包含：

```markdown
# {模組名稱} API 開發計畫

## 原始功能摘要
- 舊程式路徑：`modules/{name}/{file}.php`
- 對照表：`api-mapping-reference/{模組名稱}.md`
- 功能描述：...

## API 設計

| Endpoint | Method | 說明 |
|----------|--------|------|
| /xxx/query | POST | ... |

## 待建立檔案

- [ ] `App/Controller/XxxController.php`
- [ ] `App/Service/XxxService.php`
- [ ] `App/Dao/XxxDao.php`

## Session 替代方案

| 舊 Session | 新方案 |
|------------|--------|
| $_SESSION['user_id'] | $this->request->getUserId() |

## 待確認事項
- [ ] ...
```

### B3. 需要確認的情況

遇到以下情況時，**必須先詢問使用者**：

- [ ] 發現無法用現有 BaseController 方法替代的 session 變數
- [ ] 業務邏輯有多種實作方式可選擇
- [ ] 原始程式碼有邏輯不清或多義性
- [ ] 需要決定是否保留某些舊功能
- [ ] API 設計有重大取捨（效能 vs 簡潔）

### B4. 實作順序

1. DAO 層 - 資料存取
2. Service 層 - 業務邏輯
3. Controller 層 - 請求處理
4. **更新對照表** - 標記已實作的 API
5. 更新 `routes.md` - 新增 API 路由

### B5. 同步更新對照表

實作過程中，**必須同步更新對照表**：

- 新增的 API 對應到「v3 API 對應」表格
- 補充「參數對照」
- 記錄「設計異動」
- 更新「驗證檢查清單」打勾

---

## Session 處理原則

**盡量避免使用 `$_SESSION`**，改用以下替代方案：

| Session 用途 | 替代方案 |
|-------------|----------|
| 使用者身份 | Token 驗證 → `$this->request->getUserId()` |
| 使用者權限 | `$this->getAccessLevel()` |
| 使用者學校 | `$this->getOrganizationId()` |
| 使用者年級 | `$this->getGrade()` |
| 暫存資料 | 由前端保存，每次請求帶入 |

---

## 架構層次

```
Public/index.php → Pipeline (中間件) → Router → Controller → Service → DAO
```

| 層 | 位置 | 職責 |
|:---|:-----|:-----|
| Core | `Core/` | 框架基礎設施 (Request, Response, Router, Validator) |
| Middleware | `App/Middleware/` | 全域錯誤處理、輸入過濾 |
| Controller | `App/Controller/` | 請求處理、參數驗證、權限檢查、回應格式化 |
| Service | `App/Service/` | 業務邏輯、主動拋出 AppException |
| DAO | `App/Dao/` | 資料庫操作、不處理異常 (讓 PDOException 冒泡) |

## 目錄結構

```
ADLAPI/v3/
├── Bootstrap/autoload.php     # PSR-4 自動載入
├── Core/                      # 框架核心
│   ├── App.php                # 應用程式入口
│   ├── Pipeline.php           # 中間件管線
│   ├── Request.php            # HTTP 請求封裝
│   ├── Response.php           # JSON 回應處理
│   ├── Router.php             # 慣例式路由
│   └── Validator.php          # 參數驗證器
├── Public/index.php           # API 入口點
└── App/                       # 應用程式層
    ├── Controller/
    ├── Service/
    ├── Dao/
    ├── Middleware/
    │   ├── ErrorHandlerMiddleware.php  # 全域錯誤
    │   └── SanitizeMiddleware.php      # XSS 過濾
    └── Exception/AppException.php      # 業務異常
```

---

## 路由規則

使用 **慣例式路由 (kebab-case)**，無需設定檔：

```
URL: /controller-name/method/sub
→ App\Controller\ControllerNameController::methodSub()
```

| URL | Controller::method |
|-----|-------------------|
| `/user-data/load` | `UserDataController::load()` |
| `/ks-learning/video/record` | `KsLearningController::videoRecord()` |
| `/mission/list` | `MissionController::list()` |

---

## 新增 API 標準流程

### 1. Controller (請求處理層)

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

    // POST /xxx/query
    public function query()
    {
        // 1. 驗證 HTTP 方法與 Token
        $this->guard(['method' => 'POST', 'token' => true]);

        // 2. 驗證參數
        $params = $this->validate([
            'id' => 'required|integer'
        ]);

        // 3. 取得 userId
        $userId = $this->request->getUserId();

        // 4. 呼叫 Service
        $result = $this->xxxService->getById($userId, $params['id']);

        // 5. 回傳結果
        $this->success(['data' => $result], '查詢成功');
    }
}
```

### 2. Service (業務邏輯層)

```php
<?php
namespace App\Service;

use App\Dao\XxxDao;
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
        // 業務驗證：資料是否存在
        $existing = $this->xxxDao->getById($userId, $id);
        if (!$existing) {
            throw new AppException('資料不存在', 404);
        }
        return $existing;
    }
}
```

### 3. DAO (資料存取層)

```php
<?php
namespace App\Dao;

class XxxDao
{
    private $db;       // 寫入用
    private $dbSlave;  // 讀取用

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

---

## BaseController 方法

### guard() - 請求守衛

```php
// 驗證 HTTP 方法
$this->guard(['method' => 'POST']);
$this->guard(['method' => ['GET', 'POST']]);

// 驗證 Token (必須帶 accesstoken 參數)
$this->guard(['method' => 'POST', 'token' => true]);
```

### validate() - 參數驗證

```php
$params = $this->validate([
    'id'     => 'required|integer',
    'name'   => 'required|string|max:100',
    'page'   => 'integer|min:1|default:1',
    'status' => 'string|in:ongoing,expired|default:ongoing'
]);
```

### 使用者資料取得

```php
$userId = $this->request->getUserId();      // 需先 guard token
$level  = $this->getAccessLevel();          // user_status.access_level
$orgId  = $this->getOrganizationId();       // user_info.organization_id
$grade  = $this->getGrade();                // user_info.grade
```

### 回應方法

```php
$this->success($data, 'OK');      // 成功回應
$this->fail('Error', 400);        // 失敗回應 (少用，建議拋 AppException)
```

---

## 驗證規則 (Validator)

| 規則 | 說明 | 範例 |
|------|------|------|
| `required` | 必填 | `'id' => 'required'` |
| `string` | 字串 | `'name' => 'string'` |
| `integer` | 整數 (自動轉型) | `'page' => 'integer'` |
| `numeric` | 數字 | `'amount' => 'numeric'` |
| `array` | 陣列 | `'ids' => 'array'` |
| `email` | Email 格式 | `'email' => 'email'` |
| `min:N` | 最小值/長度 | `'page' => 'min:1'` |
| `max:N` | 最大值/長度 | `'name' => 'max:100'` |
| `in:a,b,c` | 限定值 | `'status' => 'in:ongoing,expired'` |
| `default:X` | 預設值 | `'page' => 'default:1'` |
| `after:field` | 時間大於另一欄位 | `'end_time' => 'after:start_time'` |

> **注意**：使用未定義的規則名稱會拋出 `AppException` (500)。

### 驗證分層原則

| 層級 | 適用情境 |
|------|----------|
| Controller (Validator) | 只需參數即可驗證：必填、格式、範圍 |
| Service (AppException) | 需查資料庫才能驗證：存在性、權限、業務邏輯 |

---

## 錯誤處理機制

### 異常類型

| 類型 | 處理方式 | HTTP |
|------|----------|------|
| `AppException` | Service/Controller 主動拋出 | 自訂 (400/404/...) |
| `PDOException` | DAO 不處理，自動冒泡 | 500 |
| `Throwable` | 未預期錯誤 | 500 |

### 各層職責

```
DAO       → 不處理異常，讓 PDOException 冒泡
Service   → throw new AppException('訊息', HTTP狀態碼);
Controller → 不需 try-catch，由 ErrorHandlerMiddleware 統一處理
```

---

## Read-before-Write 模式

**Update/Delete 前必須先查詢資料**：

```php
// Service 層
public function updateXxx($userId, $params)
{
    $sn = intval($params['sn']);

    // 1. Read：檢查存在與權限
    $existing = $this->xxxDao->getById($userId, $sn);
    if (!$existing) {
        throw new AppException('資料不存在', 404);
    }

    // 2. 合併資料 (支援部分更新)
    $data = [
        'field1' => $params['field1'] ?? $existing['field1'],
    ];

    // 3. Write：執行更新
    $this->xxxDao->update($userId, $sn, $data);
    return ['sn' => $sn];
}
```

---

## 資料庫連線

| 用途 | 變數 | 來源 |
|------|------|------|
| 寫入 | `$this->db` | `global $dbh` |
| 讀取 | `$this->dbSlave` | `global $dbh_slave` |

---

## Request 物件

```php
// 取得參數
$this->request->input('key');        // POST/JSON body
$this->request->query('key');        // GET 參數
$this->request->allInput();          // 所有 body 參數
$this->request->getMethod();         // HTTP 方法
```

> 使用者資料取得請參考 [BaseController 方法](#basecontroller-方法)

---

## Namespace 對應

| Namespace | 路徑 |
|-----------|------|
| `Core\*` | `Core/*.php` |
| `App\Controller\*` | `App/Controller/*.php` |
| `App\Service\*` | `App/Service/*.php` |
| `App\Dao\*` | `App/Dao/*.php` |
| `App\Exception\AppException` | `App/Exception/AppException.php` |

---

## 子目錄組織

複雜模組可用子目錄：

```
App/Dao/Knowledge/
├── NodeDao.php
├── ResourceDao.php
└── UnitPublisherDao.php
```

Namespace: `App\Dao\Knowledge\NodeDao`

---

## 設計模式

### 策略模式 (Strategy Pattern)

任務計數模組 (`MissionTaskCounter/`) 採用策略模式，將不同任務類型的計數邏輯封裝為獨立類別，符合開閉原則 (OCP)：新增任務類型只需新增 Counter 類別，無需修改 `MissionService`。

---

## 全域輸入過濾 (SanitizeMiddleware)

在 Router 分發前執行，過濾所有輸入來源：

| 過濾項目 | 說明 |
|----------|------|
| NULL bytes | 移除 `\0` 字元 |
| `<script>` | 移除 script 標籤及內容 |
| HTML 標籤 | `strip_tags()` |
| `javascript:` | 移除 JavaScript 協議 |
| `data:` | 移除 data 協議 |
| `on*=` | 移除事件處理器 |

過濾範圍：`$_GET`、`$_POST`、`php://input` (JSON body)

---

## UserDao 設計原則

使用者資料按表名分類查詢並快取到 Request：

| 方法 | 表 | 欄位 |
|------|-----|------|
| `getUserInfo()` | user_info | organization_id, grade, city_code |
| `getUserStatus()` | user_status | access_level |

```php
// Controller 中使用 (自動載入並快取)
$level = $this->getAccessLevel();       // 從 user_status
$orgId = $this->getOrganizationId();    // 從 user_info
$grade = $this->getGrade();             // 從 user_info

// 權限判斷使用 config 常數
if (in_array($level, USER_TEACHER_GROUP)) {
    // 教師邏輯
}
```

設計原則：
- 同表欄位直接加到對應 function
- 需要新表就新增獨立 function
- 除非多表聯合查詢頻繁使用，否則不做 JOIN

---

## API 路由表

完整 API 列表請參考：[routes.md](./routes.md)

---

## 功能對照表 (API Mapping Reference)

舊程式與 v3 API 的功能對照文件存放於 `api-mapping-reference/` 目錄：

```
.agent/skills/api-v3/api-mapping-reference/
├── modules_student.md    # 學生儀表板對照
├── modules_teacher.md    # 教師儀表板對照 (待建立)
└── ...
```

### 對照表格式規範

每個舊程式頁面建立一份 `.md` 文件，包含：

1. **原始程式檔案** - 列出所有相關的舊程式檔案及路徑
2. **功能拆解與 API 對應** - 依模組分類：
   - 舊程式功能表（功能、位置、實作方式）
   - v3 API 對應表（API、Controller 方法、說明）
   - 參數對照表（舊參數、新參數、值對應）
   - 設計異動（使用 `> [!NOTE]` 標注）
3. **SQL 詳細比對** - 逐一比較舊程式與 v3 DAO 的 SQL 語句：
   - 列出每個查詢的資料表、JOIN、WHERE 條件
   - 標記差異狀態（✅ 一致 / ⚠️ 微調 / ❌ 不一致）
   - 說明差異原因（如：欄位別名、多餘 JOIN 移除、優化等）
   - 確認等效性（邏輯相同即使語法不同）
4. **未對應功能** - 列出尚未實作的舊功能
5. **驗證檢查清單** - 用 `- [x]` / `- [ ]` 標記完成狀態

### 建立時機

當改寫舊程式為 API 時，**必須同步建立對照文件**，確保：
- 功能無遺漏
- 參數變更有記錄
- 設計異動有說明
- **SQL 邏輯等效**（關鍵）

---

## API 改寫驗收清單

改寫 PHP 為 API 完成後，逐項檢查：

### 架構層面

- [ ] Controller 只做請求處理，不含業務邏輯
- [ ] Service 處理所有業務邏輯與驗證
- [ ] DAO 只做資料存取，不處理異常
- [ ] 遵循 Read-before-Write 模式

### Session 移除

- [ ] 不使用 `$_SESSION` 直接存取
- [ ] 用戶身份透過 `$this->request->getUserId()` 取得
- [ ] 權限透過 `$this->getAccessLevel()` 取得

### 錯誤處理

- [ ] Service 用 `AppException` 拋出業務錯誤
- [ ] DAO 讓 `PDOException` 自然冒泡
- [ ] Controller 不需 try-catch

### 文件更新

- [ ] 更新 `routes.md` 路由表
- [ ] 若有新概念，更新此 SKILL.md

---

## Code Review 回應原則

接收代碼審查反饋時，遵循以下原則：

### 回應流程

1. **閱讀** - 完整反饋，不做反應
2. **理解** - 用自己的話重述需求（或詢問）
3. **驗證** - 對照代碼庫現實檢查
4. **評估** - 對這個代碼庫技術上合理嗎？
5. **回應** - 技術性確認或有理由的反駁
6. **實施** - 一次一項，每項都測試

### 禁止的回應

| ❌ 不要 | ✅ 應該 |
|---------|---------|
| 「你說得太對了！」 | 重述技術需求 |
| 「好觀點！」 | 問澄清問題 |
| 「讓我現在就實施」 | 用技術推理反駁（如果錯誤） |
| 任何感謝表達 | 直接開始工作（行動 > 言語） |

### 處理不清楚的反饋

如果任何項目不清楚：**停止，還不要實施任何東西**，先詢問澄清。

### 何時反駁

- 建議會破壞現有功能
- 審查者缺乏完整上下文
- 違反 YAGNI（未使用的功能）
- 對這個技術棧技術上不正確

### 確認正確的反饋

```
✅ 「已修復。[簡短描述改了什麼]」
✅ 「發現得好 - [具體問題]。在 [位置] 修復了。」
✅ [直接修復並在代碼中展示]
```
