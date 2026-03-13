# FlightPHP 反模式檢查清單

> AI 自我檢查：在產出程式碼前，逐一核對以下清單。

## 🔴 致命錯誤 (必須修正)

- [ ] **使用 `{id}` 而不是 `@id`**
  - ❌ `Flight::get('/users/{id}', ...)`
  - ✅ `Flight::get('/users/@id', ...)`

- [ ] **以為 `json()` 會中止執行**
  - ❌ `Flight::json(['error' => $msg], 400);` 後面沒有 `return`
  - ✅ 使用 `Flight::jsonHalt(...)` 或加上 `return`

- [ ] **Controller 沒有接收 Engine**
  - ❌ `public function __construct() {}`
  - ✅ `public function __construct(Engine $app) { $this->app = $app; }`

- [ ] **SQL 字串拼接**
  - ❌ `"WHERE name = '$name'"`
  - ✅ `"WHERE name = ?"` + `[$name]`

- [ ] **使用 PdoWrapper**
  - ❌ `flight\database\PdoWrapper`
  - ✅ `flight\database\SimplePdo`

- [ ] **使用 PHP 8.0+ 語法 (伺服器為 PHP 7.4)**
  - ❌ Union types: `function foo(): array|false`
  - ❌ `match()` 表達式
  - ❌ `enum`、`readonly` 屬性、Named arguments
  - ✅ 使用 PHPDoc `@return array|false` 替代 union type
  - ✅ 使用 `switch` 替代 `match()`

## 🟡 常見錯誤 (應該修正)

- [ ] **在 Controller 中直接呼叫 `Flight::xxx()`**
  - 應使用 `$this->app->xxx()`，保持可測試性

- [ ] **fetchRow 結果沒有 null 檢查**
  - `SimplePdo::fetchRow()` 回傳 `?Collection`，可能為 null

- [ ] **中間件 before() 沒有回傳 false 就做了回應**
  - 未授權時要 `return false` 才會真正中止路由

- [ ] **group callback 帶參數**
  - ❌ `Flight::group('/api', function ($router) { ... })`
  - ✅ `Flight::group('/api', function () { ... })`

- [ ] **直接 echo 輸出**
  - ❌ `echo json_encode($data);`
  - ✅ `$this->app->json($data);`

## 🟢 品質建議 (建議遵守)

- [ ] **三層架構分離**
  - Controller 不應直接寫 SQL
  - Repository 不應處理 HTTP 相關邏輯

- [ ] **使用 `declare(strict_types=1)`**
  - 每個 PHP 檔案開頭

- [ ] **參數驗證**
  - 數值參數用 `(int)` 轉型並做範圍檢查
  - 分頁: `$page = max(1, (int) $val);`
  - Limit: `$limit = min(100, max(1, (int) $val));`

- [ ] **回應格式一致**
  - 成功: `['success' => true, 'data' => ...]`
  - 錯誤: `['success' => false, 'error' => ['code' => N, 'message' => '...']]`

- [ ] **命名規範**
  - Controller 方法: `index`, `show`, `create`, `update`, `destroy`
  - Service: 業務動詞 `getList`, `findById`, `createXxx`
  - Repository: 資料動詞 `findAll`, `findById`, `insert`, `update`
  - 路由: RESTful 風格 `GET /users`, `POST /users`, `GET /users/@id`
