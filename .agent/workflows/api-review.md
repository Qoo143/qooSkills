---
description: 執行 API-V3 程式碼審查
---

# Workflow: 執行 API-V3 Code Review

本工作流程用於對已完成的 API-V3 程式碼進行系統化的審查，確保符合架構規範與最佳實踐。

## 審查目標

- 確保程式碼符合 API-V3 架構規範
- 檢查分層職責是否正確
- 驗證錯誤處理機制
- 確保安全性與效能

## 執行步驟

### 1. 準備審查

從對照表中選擇需要審查的 API，準備以下資訊：
- Controller 檔案路徑
- Service 檔案路徑
- DAO 檔案路徑
- 對應的舊功能檔案（用於比對業務邏輯）

### 2. Controller 層審查

開啟 Controller 檔案，逐項檢查：

#### 2.1 基本結構

- [ ] 繼承 `BaseController`
- [ ] Namespace 正確（`App\Controller`）
- [ ] 檔案名稱與類別名稱一致
- [ ] Constructor 正確注入 Service

```php
// ✅ 正確範例
class XxxController extends BaseController
{
    private $xxxService;

    public function __construct($request, $response)
    {
        parent::__construct($request, $response);
        $this->xxxService = new XxxService();
    }
}
```

#### 2.2 請求守衛 (guard)

- [ ] 使用 `guard()` 驗證 HTTP 方法
- [ ] 需要 Token 時設定 `'token' => true`
- [ ] HTTP 方法統一使用 POST

```php
// ✅ 正確範例
$this->guard(['method' => 'POST', 'token' => true]);

// ❌ 錯誤範例：缺少 guard
public function list() {
    $params = $this->validate([...]); // 缺少 guard
}
```

#### 2.3 參數驗證 (validate)

- [ ] 使用 `validate()` 驗證參數
- [ ] 驗證規則正確（required, integer, string, in, default 等）
- [ ] 沒有在 Controller 中進行業務邏輯驗證

```php
// ✅ 正確範例：只驗證格式
$params = $this->validate([
    'page' => 'integer|min:1|default:1',
    'status' => 'string|in:ongoing,expired'
]);

// ❌ 錯誤範例：包含業務邏輯
if ($params['status'] === 'ongoing' && $someBusinessCondition) {
    throw new AppException('xxx'); // 業務邏輯應在 Service
}
```

#### 2.4 Service 呼叫

- [ ] 從 Request 取得 userId
- [ ] 呼叫 Service 方法
- [ ] 使用 `success()` 回傳結果
- [ ] 沒有 try-catch（由 ErrorHandlerMiddleware 處理）

```php
// ✅ 正確範例
$userId = $this->request->getUserId();
$result = $this->xxxService->getList($userId, $params);
$this->success($result, '查詢成功');

// ❌ 錯誤範例：使用 try-catch
try {
    $result = $this->xxxService->getList($userId, $params);
} catch (AppException $e) {
    // Controller 不應該處理異常
}
```

### 3. Service 層審查

開啟 Service 檔案，逐項檢查：

#### 3.1 基本結構

- [ ] Namespace 正確（`App\Service`）
- [ ] Constructor 正確注入 DAO
- [ ] 引入 `AppException`

```php
// ✅ 正確範例
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
}
```

#### 3.2 業務邏輯

- [ ] 包含明確的業務邏輯
- [ ] 使用 `AppException` 處理業務錯誤
- [ ] HTTP 狀態碼正確（404, 400 等）
- [ ] 錯誤訊息清晰易懂

```php
// ✅ 正確範例
$item = $this->xxxDao->getById($userId, $id);
if (!$item) {
    throw new AppException('資料不存在', 404);
}

// ❌ 錯誤範例：錯誤訊息不明確
if (!$item) {
    throw new AppException('error', 500); // 訊息不清楚、狀態碼錯誤
}
```

#### 3.3 Read-before-Write 模式

- [ ] Update 操作前先查詢資料
- [ ] Delete 操作前先查詢資料
- [ ] 支援部分更新（使用 `??` 合併現有資料）
- [ ] 權限檢查（確認資料屬於該使用者）

```php
// ✅ 正確範例
public function update($userId, $params)
{
    // 1. Read
    $existing = $this->xxxDao->getById($userId, $params['id']);
    if (!$existing) {
        throw new AppException('資料不存在', 404);
    }

    // 2. 合併資料
    $data = [
        'field1' => $params['field1'] ?? $existing['field1'],
        'field2' => $params['field2'] ?? $existing['field2'],
    ];

    // 3. Write
    $this->xxxDao->update($userId, $params['id'], $data);
    return ['id' => $params['id']];
}

// ❌ 錯誤範例：直接更新，沒有 Read-before-Write
public function update($userId, $params)
{
    $this->xxxDao->update($userId, $params['id'], $params); // 沒有先查詢
}
```

#### 3.4 異常處理

- [ ] 不使用 try-catch 捕捉 PDOException
- [ ] 讓 PDOException 自然冒泡到 ErrorHandlerMiddleware

```php
// ✅ 正確範例：不處理 PDOException
public function create($userId, $params)
{
    // 直接呼叫 DAO，PDOException 會自動冒泡
    $id = $this->xxxDao->insert($userId, $params);
    return ['id' => $id];
}

// ❌ 錯誤範例：捕捉 PDOException
try {
    $id = $this->xxxDao->insert($userId, $params);
} catch (PDOException $e) {
    throw new AppException('資料庫錯誤', 500); // 不應該捕捉
}
```

### 4. DAO 層審查

開啟 DAO 檔案，逐項檢查：

#### 4.1 基本結構

- [ ] Namespace 正確（`App\Dao`）
- [ ] Constructor 正確注入 `$dbh` 和 `$dbh_slave`
- [ ] 讀取使用 `$dbSlave`，寫入使用 `$db`

```php
// ✅ 正確範例
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
}
```

#### 4.2 SQL 安全性

- [ ] 使用 Prepared Statements
- [ ] 參數綁定正確（`:paramName`）
- [ ] 沒有 SQL Injection 風險

```php
// ✅ 正確範例：使用 Prepared Statements
$sql = "SELECT * FROM xxx_table WHERE user_id = :userId AND id = :id";
$stmt = $this->dbSlave->prepare($sql);
$stmt->execute([':userId' => $userId, ':id' => $id]);

// ❌ 錯誤範例：字串拼接，有 SQL Injection 風險
$sql = "SELECT * FROM xxx_table WHERE user_id = $userId AND id = $id";
$stmt = $this->dbSlave->query($sql);
```

#### 4.3 資料庫操作

- [ ] 讀取操作使用 `$this->dbSlave`
- [ ] 寫入操作使用 `$this->db`
- [ ] 使用 `PDO::FETCH_ASSOC` 回傳關聯陣列

```php
// ✅ 正確範例
public function getById($userId, $id)
{
    $sql = "...";
    $stmt = $this->dbSlave->prepare($sql); // 讀取用 dbSlave
    $stmt->execute([...]);
    return $stmt->fetch(\PDO::FETCH_ASSOC);
}

public function insert($userId, $data)
{
    $sql = "...";
    $stmt = $this->db->prepare($sql); // 寫入用 db
    $stmt->execute([...]);
    return $this->db->lastInsertId();
}
```

#### 4.4 職責分離

- [ ] 不包含業務邏輯
- [ ] 不處理異常
- [ ] 只負責資料庫 CRUD

```php
// ✅ 正確範例：單純的資料查詢
public function getById($userId, $id)
{
    $sql = "SELECT * FROM xxx_table WHERE user_id = :userId AND id = :id";
    $stmt = $this->dbSlave->prepare($sql);
    $stmt->execute([':userId' => $userId, ':id' => $id]);
    return $stmt->fetch(\PDO::FETCH_ASSOC);
}

// ❌ 錯誤範例：包含業務邏輯
public function getById($userId, $id)
{
    $sql = "...";
    $stmt = $this->dbSlave->prepare($sql);
    $stmt->execute([...]);
    $result = $stmt->fetch(\PDO::FETCH_ASSOC);
    
    if (!$result) {
        throw new AppException('資料不存在', 404); // 業務邏輯應在 Service
    }
    
    return $result;
}
```

### 5. 路由與命名審查

- [ ] URL 使用 kebab-case（`/user-data/load`）
- [ ] Controller 名稱使用 PascalCase（`UserDataController`）
- [ ] 方法名稱語義清晰（`list`, `detail`, `create`, `update`, `delete`）
- [ ] Namespace 與檔案路徑對應

```php
// ✅ 正確對應
URL: /user-data/load
Controller: App\Controller\UserDataController::load()
檔案路徑: App/Controller/UserDataController.php

// ❌ 錯誤對應
URL: /userData/load  // 應該用 kebab-case
Controller: userDataController  // 應該用 PascalCase
```

### 6. 整體審查

#### 6.1 依賴方向

- [ ] Controller 依賴 Service
- [ ] Service 依賴 DAO
- [ ] 沒有反向依賴（DAO 不依賴 Service）

#### 6.2 錯誤處理

- [ ] Controller 不使用 try-catch
- [ ] Service 使用 `AppException` 處理業務錯誤
- [ ] DAO 不處理異常

#### 6.3 程式碼品質

- [ ] 變數命名語義清晰
- [ ] 沒有註解掉的程式碼
- [ ] 沒有 `var_dump()`, `print_r()`, `die()` 等調試代碼
- [ ] 適當的註解（複雜邏輯）

### 7. 比對舊功能

與舊 PHP 檔案比對，確認：

- [ ] 業務邏輯一致
- [ ] 參數驗證一致
- [ ] 權限檢查一致
- [ ] 回傳資料格式一致（或有文件說明變更）

### 8. 測試驗證

// turbo
```powershell
# 使用 curl 或 Postman 測試 API
curl -X POST "http://localhost/v3/xxx/list" \
  -H "Content-Type: application/json" \
  -d '{"accesstoken": "test_token"}'
```

檢查：
- [ ] 正常情境回傳正確
- [ ] 錯誤情境回傳正確（缺少參數、資料不存在等）
- [ ] Token 驗證正確

## 審查檢查清單總覽

### Controller
- [ ] 繼承 `BaseController`
- [ ] 使用 `guard()` 驗證
- [ ] 使用 `validate()` 驗證參數
- [ ] 不包含業務邏輯
- [ ] 不使用 try-catch
- [ ] 使用 `success()` 回傳

### Service
- [ ] 包含業務邏輯
- [ ] Update/Delete 使用 Read-before-Write
- [ ] 使用 `AppException` 處理錯誤
- [ ] 不捕捉 PDOException

### DAO
- [ ] 使用 Prepared Statements
- [ ] 讀取用 `$dbSlave`，寫入用 `$db`
- [ ] 不包含業務邏輯
- [ ] 不處理異常

### 命名與路由
- [ ] URL 使用 kebab-case
- [ ] Namespace 正確
- [ ] 檔案路徑對應

## 常見問題與修正

### 問題 1: Controller 包含業務邏輯
**修正**：將業務邏輯移至 Service 層

### 問題 2: Service 捕捉 PDOException
**修正**：移除 try-catch，讓異常自然冒泡

### 問題 3: DAO 包含業務驗證
**修正**：將驗證邏輯移至 Service 層

### 問題 4: 缺少 Read-before-Write
**修正**：在 Update/Delete 前先查詢資料

## 完成審查後

- [ ] 記錄審查結果（通過/需修改）
- [ ] 如需修改，列出修改清單
- [ ] 修改完成後重新審查
- [ ] 更新對照表（如有必要）
