# 專案架構與三層分層

> 參考來源：[FlightPHP 官方文檔](https://docs.flightphp.com)、[Creating a RESTful API](https://dev.to/n0nag0n/creating-a-restful-api-with-flight-framework-56lj)

## 目錄結構

```
ADLAPI/v4/
├── public/index.php           # 唯一入口
├── config/
│   ├── bootstrap.php          # 初始化 (autoload, db, helpers, 錯誤處理)
│   └── routes.php             # 路由定義 (純路由，無邏輯)
├── src/App/
│   ├── Controller/            # 控制器 (薄層, 只處理 HTTP 輸入/輸出)
│   ├── Service/               # 業務邏輯層
│   ├── Repository/            # 資料存取層 (SQL)
│   ├── Factory/               # 工廠類別
│   └── Middleware/            # 中間件
├── libs/
│   └── core/                  # FlightPHP 框架原始碼
└── skills/                    # Skill 文件
```

## 三層架構

```
Controller (HTTP 層)
    ↓ 呼叫
Service (業務邏輯)
    ↓ 呼叫
Repository (SQL 查詢)
```

### 每層的職責

| 層 | 職責 | 不應做的事 |
|----|------|-----------|
| **Controller** | 取得/驗證請求參數 → 呼叫 Service → 輸出 JSON | 不寫 SQL、不處理複雜邏輯 |
| **Service** | 業務邏輯 → 呼叫 Repository → 資料轉換 | 不處理 HTTP 相關（request/response） |
| **Repository** | 純 SQL 查詢 → 回傳原始資料 | 不做資料轉換、不處理 HTTP |

## 類別自動載入

```php
// bootstrap.php 設定載入路徑
Flight::path(__DIR__ . '/../src');

// Flight 的 Loader 依照命名空間映射檔案路徑:
// App\Controller\UserController → src/App/Controller/UserController.php
// App\Service\MissionService   → src/App/Service/MissionService.php
```

## 統一回應格式

```json
// 成功 (單一資料)
{
    "success": true,
    "data": { ... }
}

// 成功 (列表 + 分頁)
{
    "success": true,
    "data": [ ... ],
    "meta": {
        "page": 1,
        "limit": 20,
        "total": 100
    }
}

// 錯誤
{
    "success": false,
    "error": {
        "code": 400,
        "message": "錯誤訊息"
    }
}
```

使用 bootstrap.php 中已映射的 helper：

```php
// 成功回應
Flight::success($data);
Flight::success(['items' => $list, 'meta' => $meta]);

// 錯誤回應
Flight::fail('參數錯誤', 400);
Flight::fail('找不到', 404, ['field' => 'id']);

// ⚠️ 錯誤處理慣例:
// Controller / Service → throw new \Exception('msg', httpCode)
//   bootstrap 全域 Flight::map('error') 自動攔截 → 統一 JSON
// Middleware → $app->jsonHalt() + return false
//   因為需要立即中止路由執行
```

## 完整範例模板

### Controller 模板

```php
<?php

declare(strict_types=1);

namespace App\Controller;

use flight\Engine;
use App\Service\ExampleService;

class ExampleController
{
    protected Engine $app;
    private ExampleService $service;

    public function __construct(Engine $app)
    {
        $this->app = $app;
        $this->service = new ExampleService($app);
    }

    /**
     * 取得列表
     * GET /api/v4/examples?page=1&limit=20
     */
    public function index(): void
    {
        $page  = max(1, (int) ($this->app->request()->query['page'] ?? 1));
        $limit = min(100, max(1, (int) ($this->app->request()->query['limit'] ?? 20)));

        $result = $this->service->getList($page, $limit);

        $this->app->json([
            'success' => true,
            'data'    => $result['items'],
            'meta'    => $result['meta']
        ]);
    }

    /**
     * 取得單一
     * GET /api/v4/examples/@id
     */
    public function show(string $id): void
    {
        $item = $this->service->findById($id);

        if ($item === null) {
            $this->app->jsonHalt([
                'success' => false,
                'error'   => ['code' => 404, 'message' => '找不到資料']
            ], 404);
            return;
        }

        $this->app->json(['success' => true, 'data' => $item]);
    }

    /**
     * 新增
     * POST /api/v4/examples
     */
    public function create(): void
    {
        $data = $this->app->request()->data->getData();

        $required = ['name', 'type'];
        foreach ($required as $field) {
            if (empty($data[$field])) {
                $this->app->jsonHalt([
                    'success' => false,
                    'error'   => ['code' => 400, 'message' => "缺少必填欄位: $field"]
                ], 400);
                return;
            }
        }

        $id = $this->service->create($data);

        $this->app->json([
            'success' => true,
            'data'    => ['id' => $id]
        ], 201);
    }
}
```

### Service 模板

```php
<?php

declare(strict_types=1);

namespace App\Service;

use App\Repository\ExampleRepository;
use flight\Engine;

class ExampleService
{
    private Engine $app;
    private ExampleRepository $repo;

    public function __construct(Engine $app)
    {
        $this->app  = $app;
        $this->repo = new ExampleRepository($app);
    }

    public function getList(int $page, int $limit): array
    {
        $offset = ($page - 1) * $limit;
        $items = $this->repo->findAll($limit, $offset);
        $total = $this->repo->count();

        return [
            'items' => $items,
            'meta'  => [
                'page'  => $page,
                'limit' => $limit,
                'total' => $total
            ]
        ];
    }

    /**
     * Guard clause 模式: 減少巢狀，提早 return 或 throw
     * @throws \Exception 403
     */
    private function resolveUserId(string $loginId, ?string $childId): string
    {
        if ($childId === null) {
            return $loginId;
        }
        if (!$this->parentRepo->isMyChild($loginId, $childId)) {
            throw new \Exception('無權查看', 403);
        }
        return $childId;
    }
}
```

### Repository 模板

```php
<?php

declare(strict_types=1);

namespace App\Repository;

use flight\Engine;
use PDO;

class ExampleRepository
{
    private Engine $app;

    public function __construct(Engine $app)
    {
        $this->app = $app;
    }

    /**
     * 查詢列表
     *
     * @return array<int, array>
     */
    public function findAll(int $limit, int $offset): array
    {
        $sql = "SELECT * FROM examples ORDER BY created_at DESC LIMIT ? OFFSET ?";
        $stmt = $this->app->dbSlave()->prepare($sql);
        $stmt->execute([$limit, $offset]);
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }

    public function count(): int
    {
        $sql = "SELECT COUNT(*) as cnt FROM examples";
        $stmt = $this->app->dbSlave()->prepare($sql);
        $stmt->execute();
        $row = $stmt->fetch(PDO::FETCH_ASSOC);
        return (int) ($row ? $row['cnt'] : 0);
    }

    public function findById(string $id): ?array
    {
        $sql = "SELECT * FROM examples WHERE id = ?";
        $stmt = $this->app->dbSlave()->prepare($sql);
        $stmt->execute([$id]);
        $row = $stmt->fetch(PDO::FETCH_ASSOC);
        return $row ?: null;
    }

    public function insert(array $data): string
    {
        $columns = implode(', ', array_keys($data));
        $placeholders = implode(', ', array_fill(0, count($data), '?'));
        $sql = "INSERT INTO examples ({$columns}) VALUES ({$placeholders})";

        $stmt = $this->app->db()->prepare($sql);
        $stmt->execute(array_values($data));

        return $this->app->db()->lastInsertId();
    }
}
```

### Middleware 模板

```php
<?php

declare(strict_types=1);

namespace App\Middleware;

use flight\Engine;

class RoleMiddleware
{
    private Engine $app;

    public function __construct(Engine $app)
    {
        $this->app = $app;
    }

    public function before(): ?bool
    {
        $userType = (int) ($_SESSION['user_type'] ?? 0);

        if ($userType === 0) {
            $this->app->jsonHalt([
                'success' => false,
                'error'   => ['code' => 403, 'message' => '權限不足']
            ], 403);
            return false;
        }

        return true;
    }
}
```

## PHP 版本相容性

- 框架要求 **PHP >= 7.4**
- 可以使用：型別宣告、箭頭函式 `fn()`、null 合併 `??`、屬性型別
- **不要使用**：PHP 8.0+ 的 named arguments、match 表達式、enum、readonly 屬性、交集型別

## 效能考量

- `Flight::register()` 是**延遲載入**（第一次呼叫才實例化），善加利用
- `Flight::map()` 在 init 時就註冊，適合輕量工具函式
- 使用 `Flight::dbSlave()` 做讀取查詢，分散資料庫壓力

## 安全性

- **永遠使用參數綁定**，不要拼接 SQL
- 中間件內做身份驗證使用 `$this->app->jsonHalt()` 中止
- CORS 設定在 `bootstrap.php` 的 `before('start')` hook 中

## 錯誤處理慣例

```php
// bootstrap.php 中統一設定（實際程式碼）
Flight::map('error', function (Throwable $e) {
    $code = $e->getCode();

    // 確保 HTTP 狀態碼有效 (100-599)
    if ($code < 100 || $code >= 600) {
        $code = 500;
    }

    // 記錄錯誤到 log
    error_log(sprintf(
        "[API v4 Error] %s in %s:%d\n%s",
        $e->getMessage(),
        $e->getFile(),
        $e->getLine(),
        $e->getTraceAsString()
    ));

    Flight::json([
        'success' => false,
        'error' => [
            'code'    => $code,
            'message' => $e->getMessage()
        ]
    ], $code);
});

Flight::map('notFound', function () {
    Flight::json([
        'success' => false,
        'error' => [
            'code'    => 404,
            'message' => 'API endpoint not found'
        ]
    ], 404);
});
```

各層的錯誤處理策略：

| 層 | 策略 | 範例 |
|----|------|------|
| **Controller** | 參數驗證失敗 → `jsonHalt()` | `$this->app->jsonHalt(['error' => '缺少欄位'], 400)` |
| **Service** | 業務邏輯錯誤 → `throw new \Exception()` | `throw new \Exception('找不到', 404)` |
| **Middleware** | 驗證失敗 → `jsonHalt()` + `return false` | `$this->app->jsonHalt([...], 401); return false;` |

## Session 使用慣例

本專案**直接使用 `$_SESSION`** 是正常的設計選擇，因為因材網的 Session 結構由舊有的 `auth_chk.php` 統一管理。

```php
// Controller / Service 中直接使用
$userId   = $_SESSION['user_id'];                       // 當前使用者 ID
$userType = (int) $_SESSION['user_data']->access_level; // 使用者類型
$userData = $_SESSION['user_data'];                     // 完整資料物件
```

bootstrap.php 另外提供 helper 方法（封裝相同的 Session 存取）：

```php
Flight::currentUserId()   // 等同 $_SESSION['user_id']
Flight::currentUserType() // 等同 (int) $_SESSION['user_data']->access_level
Flight::currentUserData() // 等同 $_SESSION['user_data']
Flight::isAuthenticated() // 等同 !empty($_SESSION['user_id'])
```

> **兩種方式都可以用**，直接用 `$_SESSION` 更簡潔明確。

## 命名規範

| 項目 | 規範 | 範例 |
|------|------|------|
| Controller 方法 | RESTful 動詞 | `index`, `show`, `create`, `update`, `destroy` |
| Service 方法 | 業務動詞 | `getList`, `findById`, `createXxx` |
| Repository 方法 | 資料動詞 | `findAll`, `findById`, `insert`, `update` |
| 路由 | RESTful 風格 | `GET /users`, `POST /users`, `GET /users/@id` |
