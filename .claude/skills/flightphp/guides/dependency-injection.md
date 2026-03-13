# 依賴注入容器指南

> 參考來源：[FlightPHP DI Container 官方文檔](https://docs.flightphp.com/learn/dependency-injection-container)

## 概述

依賴注入容器 (DIC) 是一個管理應用程式依賴關係的強大工具。DIC 讓你在集中位置創建和管理類別實例，尤其在需要將同一個物件傳遞給多個類別時非常有用。

## 為什麼需要 DIC？

### 傳統方式（手動 DI）

```php
// 在 routes.php 中手動建立所有依賴
$db = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass');
$UserController = new UserController($db);
Flight::route('/user/@id', [$UserController, 'view']);
// 每增加一個 Controller，都需要重複傳遞 $db
```

### 使用 DIC

```php
// 在一個地方定義規則
$container = new \Dice\Dice;
$container = $container->addRule('PDO', [
    'shared' => true,  // 每次都返回相同實例
    'constructParams' => ['mysql:host=localhost;dbname=test', 'user', 'pass']
]);

// 註冊容器
Flight::registerContainerHandler(function ($class, $params) use ($container) {
    return $container->create($class, $params);
});

// 路由自動注入依賴！
Flight::route('/user/@id', [UserController::class, 'view']);
Flight::route('/company/@id', [CompanyController::class, 'view']);
Flight::route('/settings', [SettingsController::class, 'view']);
// 所有 Controller 的 PDO 依賴都自動解析！
```

## 本專案的 DI 模式

本專案目前使用 `Flight::register()` 做輕量級服務管理：

```php
// bootstrap.php
Flight::register('db', function () use ($dbh) { return $dbh; });
Flight::register('dbSlave', function () use ($dbh_slave) { return $dbh_slave; });

// Controller 中建立 Service
class ExampleController
{
    protected Engine $app;
    private ExampleService $service;

    public function __construct(Engine $app)
    {
        $this->app = $app;
        $this->service = new ExampleService(); // 手動建立
    }
}

// Service 中建立 Repository
class ExampleService
{
    private ExampleRepository $repo;

    public function __construct()
    {
        $this->repo = new ExampleRepository(); // 手動建立
    }
}
```

### 進階：使用 DIC 容器的 Controller

如果未來需要更好的可測試性，可以改用 DIC：

```php
// 使用 Dice DIC
$container = new \Dice\Dice;
$container = $container->addRule(ExampleService::class, [
    'shared' => true,
    'substitutions' => [
        ExampleRepository::class => new ExampleRepository()
    ]
]);

Flight::registerContainerHandler(function ($class, $params) use ($container) {
    return $container->create($class, $params);
});

// Controller 建構子可以直接接收依賴
class ExampleController
{
    protected Engine $app;
    private ExampleService $service;

    public function __construct(Engine $app, ExampleService $service)
    {
        $this->app = $app;
        $this->service = $service; // 自動注入
    }
}
```

## 可用的 DIC 庫

| 庫 | 特點 |
|---|------|
| [flightphp/container](https://github.com/flightphp/container) | FlightPHP 官方容器 |
| [Dice](https://r.je/dice) | 輕量、零配置 |
| [PHP-DI](https://php-di.org/) | 功能完整、支援 Annotation |
| [Pimple](https://pimple.symfony.com/) | Symfony 出品、簡單 |
| [league/container](https://container.thephpleague.com/) | PSR-11 相容 |

## Flight 內建的 DI 機制

Flight 在不使用 DIC 的情況下也有基本的依賴注入：

1. **Controller 建構子自動注入 Engine**：`public function __construct(Engine $app)`
2. **中間件建構子自動注入 Engine**：`public function __construct(Engine $app)`
3. **`Flight::register()`** 提供延遲載入單例模式

## 何時考慮使用 DIC？

- 有超過 5 個 Controller 且共享多個 Service
- 需要為測試替換依賴
- Service 之間有複雜的依賴關係圖
- 需要遵守 SOLID 原則中的 Dependency Inversion Principle
