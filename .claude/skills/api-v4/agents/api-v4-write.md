# Agent: api-v4-write

> 根據端點分析結果，逐一實作 v4 API 端點。

## 觸發方式

用戶完成 `/api-v4-analyze` 後，選擇一個端點開始實作。

## 執行流程

### 1. 確認實作範圍

在開始寫程式碼前，確認：
- 要實作哪個端點
- SQL 是否已由用戶確認
- 是否有外部依賴需要處理

### 2. 實作順序（由內而外）

嚴格按照以下順序：

```
① Repository（SQL 層）
    ↓
② Service（業務邏輯）
    ↓
③ Controller（HTTP 層）
    ↓
④ routes.php（路由註冊）
```

### 3. Repository 撰寫規範

```php
<?php
declare(strict_types=1);
namespace App\Repository;

use flight\Engine;

class XxxRepository
{
    private Engine $app;

    public function __construct(Engine $app)
    {
        $this->app = $app;
    }

    /**
     * 方法說明
     *
     * SQL 來源: prodb_xxx.php L45-78
     */
    public function findList(string $userId, int $limit, int $offset): array
    {
        $sql = "...";  // 用戶確認的 SQL

        // 讀取用 dbSlave，寫入用 db
        $stmt = $this->app->dbSlave()->prepare($sql);
        $stmt->bindValue(':user_id', $userId, \PDO::PARAM_STR);
        $stmt->execute();

        return $stmt->fetchAll(\PDO::FETCH_ASSOC);
    }
}
```

**規則**：
- 讀取查詢用 `$this->app->dbSlave()`
- 寫入操作用 `$this->app->db()`
- 用 `bindValue` 綁定參數，不用 `execute([$params])`
- 標註 SQL 來源行號
- 不處理例外，讓 PDOException 冒泡

### 4. Service 撰寫規範

```php
<?php
declare(strict_types=1);
namespace App\Service;

use App\Repository\XxxRepository;
use flight\Engine;

class XxxService
{
    private Engine $app;
    private XxxRepository $repo;

    public function __construct(Engine $app)
    {
        $this->app  = $app;
        $this->repo = new XxxRepository($app);
    }

    public function getList(string $userId, array $params): array
    {
        // 業務邏輯、驗證
        $rows = $this->repo->findList($userId, $params['per_page'], $offset);
        return $rows;
    }
}
```

**規則**：
- 業務驗證失敗用 `throw new \Exception('訊息', HTTP狀態碼)`
- Write 操作前先 Read 確認資料存在（Read-before-Write）
- 不碰 `$this->app->request()` 或 `$this->app->response()`

### 5. Controller 撰寫規範

```php
<?php
declare(strict_types=1);
namespace App\Controller;

use flight\Engine;
use App\Service\XxxService;

class XxxController
{
    private Engine $app;
    private XxxService $service;

    public function __construct(Engine $app)
    {
        $this->app     = $app;
        $this->service = new XxxService($app);
    }

    public function index(): void
    {
        $query = $this->app->request()->query;
        $result = $this->service->getList(
            $_SESSION['user_id'],
            ['per_page' => $query->per_page ?? 15]
        );
        $this->app->success($result);
    }
}
```

**規則**：
- Controller 盡量薄：取參數 → 呼叫 Service → 回傳結果
- 用 `$this->app->success()` / `$this->app->fail()` 回傳
- 不寫 SQL、不做複雜邏輯
- 檔案頭部加 legacy 來源註解

### 6. 路由註冊

在 `config/routes.php` 加入：

```php
// 在 AuthMiddleware group 內
Flight::group('/xxx', function () {
    Flight::route('GET /', [XxxController::class, 'index']);
    Flight::route('GET /@id:[0-9]+', [XxxController::class, 'show']);
});
```

**規則**：
- 加入 `use` 語句在檔案頂部
- 需要參數驗證的加 `ValidateQueryMiddleware` / `ValidateBodyMiddleware`
- 路由加註解說明來源

### 7. 完成確認

每個端點完成後：
1. 列出新增/修改的檔案清單
2. 標註測試建議（cURL 或 Postman 範例）
3. 詢問是否繼續下一個端點

## 注意事項

- **不主動新增中間件** — 除非端點有特殊的驗證需求
- **遵循現有模式** — 參考 CalendarController/Service/Repository 的寫法風格
- **SQL 來源標記** — 每個 Repository 方法的 docblock 都標記 SQL 來源
- **不過度設計** — 簡單端點不需要 Factory、DTO 等額外抽象
