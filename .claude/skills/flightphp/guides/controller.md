# Controller 開發指南

> 參考來源：[FlightPHP 官方文檔](https://docs.flightphp.com/learn)、[Creating a RESTful API](https://dev.to/n0nag0n/creating-a-restful-api-with-flight-framework-56lj)

## 基本結構

Controller 的建構子會自動接收 `Engine` 實例（由 Dispatcher 的 `invokeCallable` 注入）。

```php
<?php

declare(strict_types=1);

namespace App\Controller;

use flight\Engine;

class UserController
{
    protected Engine $app;

    // Engine 自動注入，不需要手動 new
    public function __construct(Engine $app)
    {
        $this->app = $app;
    }

    public function index(): void
    {
        // 取得查詢參數
        $page = (int) ($this->app->request()->query['page'] ?? 1);
        $limit = (int) ($this->app->request()->query['limit'] ?? 20);

        // 呼叫 Service
        $result = $this->userService->getList($page, $limit);

        // 回應
        $this->app->json([
            'success' => true,
            'data'    => $result
        ]);
    }

    public function show(string $id): void
    {
        $user = $this->userService->findById($id);

        if (!$user) {
            // jsonHalt 會立即中止
            $this->app->jsonHalt([
                'success' => false,
                'error'   => ['message' => '找不到使用者']
            ], 404);
            return; // 實際不會執行到，但有助可讀性
        }

        $this->app->json(['success' => true, 'data' => $user]);
    }
}
```

## 取得請求資料

```php
// GET 查詢參數
$value = $this->app->request()->query['key'];

// POST/PUT body (JSON 已自動解析)
$data = $this->app->request()->data;      // Collection 物件
$name = $this->app->request()->data['name'];
$all  = $this->app->request()->data->getData(); // 轉陣列

// 原始 body
$raw = $this->app->request()->getBody();

// HTTP 方法
$method = $this->app->request()->method;    // 'GET', 'POST', ...
$method = $this->app->request()->getMethod(); // 同上（支援 _method 覆寫）

// URL 資訊
$url     = $this->app->request()->url;      // 路徑部分
$fullUrl = $this->app->request()->getFullUrl();
$baseUrl = $this->app->request()->getBaseUrl();

// Header
$auth = $this->app->request()->getHeader('Authorization');
$all  = $this->app->request()->getHeaders();

// IP
$ip = $this->app->request()->ip;
$ip = $this->app->request()->getProxyIpAddress(); // 透過代理

// 上傳檔案
$files = $this->app->request()->getUploadedFiles();

// Content Type
$type = $this->app->request()->type; // 如 'application/json'
```

> **重要**：官方強烈建議不要直接使用 `$_GET`、`$_POST`、`$_REQUEST` 等超全域變數。
> 應統一透過 `$this->app->request()` 物件來存取。

## 服務注入模式

Controller 透過建構子建立 Service 實例，保持三層架構分離：

```php
class ExampleController
{
    protected Engine $app;
    private ExampleService $service;

    public function __construct(Engine $app)
    {
        $this->app = $app;
        $this->service = new ExampleService();
    }

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

    public function create(): void
    {
        $data = $this->app->request()->data->getData();

        // 驗證必填欄位
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

## ⚠️ Controller 常見錯誤

```php
// ❌ 錯誤：使用 $this->request (Engine 不會設為屬性)
$data = $this->request->data;

// ✅ 正確：呼叫方法
$data = $this->app->request()->data;

// ❌ 錯誤：建構子沒有 Engine 參數
class BadController {
    public function __construct() { }  // 不接收 Engine
}

// ✅ 正確
class GoodController {
    protected Engine $app;
    public function __construct(Engine $app) {
        $this->app = $app;
    }
}

// ❌ 錯誤：在 Controller 中直接使用 Flight:: 靜態呼叫
Flight::json($data);

// ✅ 正確：使用實例方法，保持可測試性
$this->app->json($data);
```
