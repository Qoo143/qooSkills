# 單元測試指南

> 參考來源：[FlightPHP 單元測試官方指南](https://docs.flightphp.com/guides/unit-testing)、[GitHub 範例](https://github.com/n0nag0n/flight-unit-tests-guide)

## 為什麼要寫單元測試？

單元測試確保程式碼按預期行為運作，在問題到達生產環境之前捕獲 bug。在 Flight 中，這尤其有價值，因為輕量級路由和靈活性可能導致複雜的交互。

- 充當安全網，記錄預期行為
- 防止回歸（Regression）
- 改善設計：**難以測試的程式碼通常表明類過於複雜或耦合緊密**

## PHPUnit 設定

### 1. 安裝

```bash
composer require --dev phpunit/phpunit
```

### 2. 建立 tests 目錄

```
ADLAPI/v4/
├── tests/
│   ├── Controller/
│   ├── Service/
│   └── Repository/
```

### 3. 在 composer.json 加入測試腳本

```json
{
    "scripts": {
        "test": "phpunit --configuration phpunit.xml"
    }
}
```

### 4. 建立 phpunit.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit bootstrap="vendor/autoload.php">
    <testsuites>
        <testsuite name="Flight Tests">
            <directory>tests</directory>
        </testsuite>
    </testsuites>
</phpunit>
```

### 5. 執行測試

```bash
composer test
```

## 測試 Controller

### 基本測試模式

核心概念：直接建立 `Engine` 實例，設定模擬資料，呼叫 Controller 方法，檢查回應。

```php
<?php

use PHPUnit\Framework\TestCase;
use flight\Engine;

class UserControllerTest extends TestCase
{
    public function testValidEmailReturnsSuccess()
    {
        $app = new Engine();
        $request = $app->request();
        $request->data->email = 'test@example.com'; // 模擬 POST 資料

        $controller = new UserController($app);
        $controller->register();

        $response = $app->response()->getBody();
        $output = json_decode($response, true);

        $this->assertEquals('success', $output['status']);
    }

    public function testInvalidEmailReturnsError()
    {
        $app = new Engine();
        $request = $app->request();
        $request->data->email = 'invalid-email';

        $controller = new UserController($app);
        $controller->register();

        $response = $app->response()->getBody();
        $output = json_decode($response, true);

        $this->assertEquals('error', $output['status']);
    }
}
```

### 關鍵要點

- 使用 `$app->request()` 模擬資料，**不要使用** `$_POST`、`$_GET` 等全域變數
- 所有 Controller 預設都會注入 `flight\Engine` 實例
- 測試中完全不使用 `Flight::`，使程式碼更容易測試

## 使用依賴注入的可測試 Controller

對於更複雜的情境，透過建構子注入依賴，避免全域狀態：

```php
use flight\database\PdoWrapper;

class UserController
{
    protected $app;
    protected $db;
    protected $mailer;

    public function __construct(Engine $app, PdoWrapper $db, MailerInterface $mailer)
    {
        $this->app = $app;
        $this->db = $db;
        $this->mailer = $mailer;
    }

    public function register()
    {
        $email = $this->app->request()->data->email;

        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            return $this->app->jsonHalt(['status' => 'error', 'message' => 'Invalid email']);
        }

        $this->db->runQuery('INSERT INTO users (email) VALUES (?)', [$email]);
        $this->mailer->sendWelcome($email);

        return $this->app->json(['status' => 'success', 'message' => 'User registered']);
    }
}
```

### 使用 Mock 測試

```php
class UserControllerDICTest extends TestCase
{
    public function testValidEmailSavesAndSendsEmail()
    {
        $statementMock = $this->createMock(PDOStatement::class);
        $statementMock->method('execute')->willReturn(true);

        // 使用匿名類別 mock PdoWrapper
        $mockDb = new class($statementMock) extends PdoWrapper {
            protected $statementMock;
            public function __construct($statementMock) {
                $this->statementMock = $statementMock;
            }
            public function runQuery(string $sql, array $params = []): PDOStatement {
                return $this->statementMock;
            }
        };

        $mockMailer = new class implements MailerInterface {
            public $sentEmail = null;
            public function sendWelcome($email): bool {
                $this->sentEmail = $email;
                return true;
            }
        };

        $app = new Engine();
        $app->request()->data->email = 'test@example.com';

        $controller = new UserController($app, $mockDb, $mockMailer);
        $controller->register();

        $response = $app->response()->getBody();
        $result = json_decode($response, true);

        $this->assertEquals('success', $result['status']);
        $this->assertEquals('test@example.com', $mockMailer->sentEmail);
    }
}
```

## ⚠️ 避免過度 Mock

**壞的做法**：Mock 掉所有業務邏輯方法（如 `isEmailValid`、`registerUser`），導致測試只在驗證程式碼的"骨架"而非行為。

```php
// ❌ 不要這樣做
$controller = new class($app) extends UserController {
    protected function isEmailValid($email) {
        return true; // 跳過真實驗證 → 測試毫無意義
    }
    protected function registerUser($email) {
        return false; // 跳過真實操作 → 無法捕獲 bug
    }
};
```

**好的做法**：Mock 第三方依賴（DB、Email），讓 Controller 的業務邏輯正常執行。

## 常見陷阱

1. **過度模擬**：不要 mock 每個依賴；讓驗證等邏輯正常執行以測試真實行為
2. **全域狀態**：大量使用 `$_SESSION`、`$_COOKIE`、`Flight::` 會使測試脆弱。重構為顯式傳遞依賴
3. **複雜設置**：如果測試設置繁瑣，你的類可能有太多依賴或職責，違反 SOLID 原則

## 延伸閱讀

- [完整範例 GitHub Repo](https://github.com/n0nag0n/flight-unit-tests-guide)
- [PHPUnit 文檔](https://docs.phpunit.de/)
- [FlightPHP SOLID 原則](https://docs.flightphp.com/learn/unit-testing-and-solid-principles)
