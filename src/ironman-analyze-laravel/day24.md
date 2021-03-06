# 分析 Auth（1）

Auth 是筆者認為，在 Laravel 開放的 Illuminate 套件包裡（Support 除外），前三名複雜的。

> 註一：Support 除外的原因是，它比較像是 helper，類別大部分都能獨立運作，如 Facade。
> 註二：筆者認為的前三名：`Database`、`Auth`、`Queue`

之前幾天在分析元件時，都是很容易知道如何開始，因為都有進入點（entry point）。無論是從初始化實例開始或是實際 Laravel 如何使用這個元件，都是有辦法找出開始分析的路徑。但 Auth，但類別本身較複雜，加上筆者並沒有使用過 Auth 套件，所以一開始也不知道該如何下手。

這時還有一種方法：看文件。文件通常會用容易理解的方法，讓開發者可以快速了解元件會有哪些角色，以及它們如何互動。從文件去了解套件如何開始分析，是有幫助的。

## Authentication

文件裡可以大致知道 Auth 提供兩種類型：[Authentication](https://laravel.com/docs/5.7/authentication) 和 [Authorization](https://laravel.com/docs/5.7/authorization)，今天就先從 Authentication 開始。

裡面有提到的關鍵角色是 Guard 與 Providers。Guard 處理驗證，而 Providers 是提供資料讓 Guard 驗證。相關的 UML 圖如下：

![](http://www.plantuml.com/plantuml/png/hLDDImGn3BtFhvW_qFa5oxBYn-9145TlUucTh1OxAIQf8FZZdLeFMnYO25w68SbxVQzvtOa2QvvY5qYHJ-2nluqnJu50yNYPI1cyol4Yw-lF1qc31uNdY2RCpVoR-DCqky_0esdoeA1uoj6EU1BaUquVOKJkEXz1fBygFa2mwTNMTKpl6KaNMdiavE1BvxVoWFEiQ1LJImSK2OdEIu_f3Pj2qNK712z5qUeDzfklOMWmwn2tNRrj0udRcZUnSjhimRb_8qlEdbM3ic5e5sb1d2_LPglYsdoONVGRPusudRFeQK9jEw7Y5fxGdmYM7zeEQSnJ_0O0)

    @startuml
    interface Illuminate\Contracts\Auth\Authenticatable
    interface Illuminate\Contracts\Auth\Factory
    interface Illuminate\Contracts\Auth\Guard
    interface Illuminate\Contracts\Auth\StatefulGuard
    interface Illuminate\Contracts\Auth\UserProvider
    
    class DatabaseUserProvider
    class EloquentUserProvider
    class GenericUser
    class RequestGuard
    class SessionGuard
    class TokenGuard
    class AuthManager
    
    Illuminate\Contracts\Auth\Factory <|.. AuthManager
    Illuminate\Contracts\Auth\Factory -> Illuminate\Contracts\Auth\Guard
    Illuminate\Contracts\Auth\Factory --> Illuminate\Contracts\Auth\StatefulGuard
    Illuminate\Contracts\Auth\Guard <|-- Illuminate\Contracts\Auth\StatefulGuard
    Illuminate\Contracts\Auth\Guard o- Illuminate\Contracts\Auth\Authenticatable
    Illuminate\Contracts\Auth\Guard <|.. RequestGuard
    Illuminate\Contracts\Auth\Guard <|.. TokenGuard
    Illuminate\Contracts\Auth\StatefulGuard <|.. SessionGuard
    Illuminate\Contracts\Auth\Authenticatable <- Illuminate\Contracts\Auth\UserProvider
    Illuminate\Contracts\Auth\Authenticatable <|.. GenericUser
    Illuminate\Contracts\Auth\UserProvider <|.. DatabaseUserProvider
    Illuminate\Contracts\Auth\UserProvider <|.. EloquentUserProvider
    @enduml

從圖可以知道，UserProvider 是建構 Authenticatable 的角色；Factory 是負責建構 Guard；而 Authenticatable 則是提供給 Guard 做驗證用。

## Service provider

Auth 套件主要的 service provider 是 [AuthServiceProvider][]：

```php
$this->app->singleton('auth', function ($app) {
    $app['auth.loaded'] = true;

    // 建構 UML 圖裡的 AuthManager 類別
    return new AuthManager($app);
});

$this->app->singleton('auth.driver', function ($app) {
    // 取得預設的 guard 作為 driver
    return $app['auth']->guard();
});

$this->app->bind(AuthenticatableContract::class, function ($app) {
    // 當需要取得 Authenticatable 物件時，會使用 AuthManager 的 userResolver 解析
    return call_user_func($app['auth']->userResolver());
});

$this->app->rebinding('request', function ($app, $request) {
    // 將全域的 $request 多設定 userResolver 的參數
    $request->setUserResolver(function ($guard = null) use ($app) {
        return call_user_func($app['auth']->userResolver(), $guard);
    });
});
```

另外，因為已經知道 Facade 的用法了，裡面有一個 Auth Facade 是對應到 Container 綁定的 `auth`，也就是 [AuthManager][]。

## Authenticate Middleware

知道類別如何綁定後，再來了解它如何被使用。[Authenticate Middleware][] 會檢查使用者是否有驗證，從 `handle()` 與 `authenticate()` 可以大概知道需要呼叫哪些參數

```php
// 額外參數使用 ...，代表 guards 是可以設定很多筆的
public function handle($request, Closure $next, ...$guards)
{
    $this->authenticate($request, $guards);

    return $next($request);
}

protected function authenticate($request, array $guards)
{
    // 如果 middleware 沒有設定的話，就使用 null，也就是預設值
    if (empty($guards)) {
        $guards = [null];
    }

    // 依續檢查每一個 guard
    foreach ($guards as $guard) {
        // 當有檢查過，就呼叫 shouldUse()
        if ($this->auth->guard($guard)->check()) {
            return $this->auth->shouldUse($guard);
        }
    }

    // 驗證失敗的例外
    throw new AuthenticationException(
        'Unauthenticated.', $guards, $this->redirectTo($request)
    );
}
```

來看 `guard()` 是如何取得 driver 的，其實跟 SessionManger 或 LogManager 都非常像：

```php
public function guard($name = null)
{
    // 是 null 就取預設的 driver
    $name = $name ?: $this->getDefaultDriver();

    // 如果不存在的話就解析一下
    return $this->guards[$name] ?? $this->guards[$name] = $this->resolve($name);
}

public function getDefaultDriver()
{
    // Laravel 預設的 driver 叫 `web` 
    return $this->app['config']['auth.defaults.guard'];
}
```

`resolve()` 原始碼：

```php
protected function resolve($name)
{
    // 取得設定，如 web 的預設設定是：['driver' => 'session', 'provider' => 'users']
    $config = $this->getConfig($name);

    if (is_null($config)) {
        throw new InvalidArgumentException("Auth guard [{$name}] is not defined.");
    }

    // 有自定義的建置方法就使用
    if (isset($this->customCreators[$config['driver']])) {
        return $this->callCustomCreator($name, $config);
    }

    // 使用預設的建置方法
    $driverMethod = 'create'.ucfirst($config['driver']).'Driver';

    if (method_exists($this, $driverMethod)) {
        return $this->{$driverMethod}($name, $config);
    }

    throw new InvalidArgumentException("Auth driver [{$config['driver']}] for guard [{$name}] is not defined.");
}
```

web 的 driver 是 session，對應的方法是 `createSessionDriver()`：

```php
public function createSessionDriver($name, $config)
{
    // 建構 UserProvider
    $provider = $this->createUserProvider($config['provider'] ?? null);

    // 建構 SessionGuard，也是 UML 有提到的實作
    $guard = new SessionGuard($name, $provider, $this->app['session.store']);

    // 如果 guard 有定義 setCookieJar 方法，就呼叫一下
    if (method_exists($guard, 'setCookieJar')) {
        $guard->setCookieJar($this->app['cookie']);
    }

    // 如果 guard 有定義 setDispatcher 方法，就呼叫一下
    if (method_exists($guard, 'setDispatcher')) {
        $guard->setDispatcher($this->app['events']);
    }

    // 如果 guard 有定義 setRequest 方法，就呼叫一下
    if (method_exists($guard, 'setRequest')) {
        // 這裡會觸發 rebind 事件，並把 request 設定給 $guard
        $guard->setRequest($this->app->refresh('request', $guard, 'setRequest'));
    }

    return $guard;
}
```

> 這裡的寫法其實是有點奇怪的。`$guard` 很明確使用 `new SessionGuard()` 建構，該實例會有什麼方法，是可以預期的，因此下面 set 的相關方法，使用 `method_exists()` 判斷就顯得多餘。

經過上面的過程，`guard()` 就能取得 [SessionGuard][] 實例了。

今天休息一下，明天再繼續看這個 guard 是如何檢查的。

[AuthManager]: https://github.com/laravel/framework/blob/v5.7.6/src/Illuminate/Auth/AuthManager.php
[AuthServiceProvider]: https://github.com/laravel/framework/blob/v5.7.6/src/Illuminate/Auth/AuthServiceProvider.php
[Authenticate Middleware]: https://github.com/laravel/framework/blob/v5.7.6/src/Illuminate/Auth/Middleware/Authenticate.php
[SessionGuard]: https://github.com/laravel/framework/blob/v5.7.6/src/Illuminate/Auth/SessionGuard.php
