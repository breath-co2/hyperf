# 快速入门

为了让您更快的了解 `Hyperf` 的使用，本章节将以 `创建一个 HTTP Server` 为例，通过对路由、控制器的定义实现一个简单的 `Web` 服务，但 `Hyperf` 不止于此，完善的服务治理、`gRPC` 服务、注解、`AOP` 等功能将由具体的章节阐述。

## 定义访问路由

Hyperf 使用 [nikic/fast-route](https://github.com/nikic/FastRoute) 作为默认的路由组件并提供服务，您可以很方便的在 `config/routes.php` 中定义您的路由。   
不仅如此，框架还提供了极其强大和方便灵活的`注解路由`功能，关于路由的详情文档请查阅 [路由](zh/router.md) 章节

### 通过配置文件定义路由
路由的文件位于 [hyperf-skeleton](https://github.com/hyperf-cloud/hyperf-skeleton) 项目的 `config/routes.php` ，下面是一些常用的用法示例。
```php
<?php
use Hyperf\HttpServer\Router\Router;

// 此处代码示例为每个示例都提供了三种不同的绑定定义方式，实际配置时仅可采用一种且仅定义一次相同的路由

// 设置一个 GET 请求的路由，绑定访问地址 '/get' 到 App\Controller\IndexController 的 get 方法
Router::get('/get', 'App\Controller\IndexController::get');
Router::get('/get', 'App\Controller\IndexController@get');
Router::get('/get', [\App\Controller\IndexController::class, 'get']);

// 设置一个 POST 请求的路由，绑定访问地址 '/post' 到 App\Controller\IndexController 的 post 方法
Router::post('/post', 'App\Controller\IndexController::post');
Router::post('/post', 'App\Controller\IndexController@post');
Router::post('/post', [\App\Controller\IndexController::class, 'post']);

// 设置一个允许 GET、POST 和 HEAD 请求的路由，绑定访问地址 '/multi' 到 App\Controller\IndexController 的 multi 方法
Router::addRoute(['GET', 'POST', 'HEAD'], '/multi', 'App\Controller\IndexController::multi');
Router::addRoute(['GET', 'POST', 'HEAD'], '/multi', 'App\Controller\IndexController@multi');
Router::addRoute(['GET', 'POST', 'HEAD'], '/multi', [\App\Controller\IndexController::class, 'multi']);
```

### 通过注解来定义路由

`Hyperf` 提供了极其强大和方便灵活的 [注解](zh/annotation.md) 功能，在路由的定义上也毫无疑问地提供了注解定义的方式，Hyperf 提供了 `@Controller` 和 `@AutoController` 两种注解来定义一个 `Controller`，此处仅做简单的说明，更多细节请查阅 [路由](zh/router.md) 章节。

### 通过 `@AutoController` 注解定义路由
`@AutoController` 为绝大多数简单的访问场景提供路由绑定支持，使用 `@AutoController` 时则 Hyperf 会自动解析所在类的所有 `public` 方法并提供 `GET` 和 `POST` 两种请求方式。

> 使用 `@AutoController` 注解时需 `use Hyperf\HttpServer\Annotation\AutoController;` 命名空间；

```php
<?php
declare(strict_types=1);

namespace App\Controller;

use Hyperf\HttpServer\Contract\RequestInterface;
use Hyperf\HttpServer\Annotation\AutoController;

/**
 * @AutoController()
 */
class IndexController
{
    // Hyperf 会自动为此方法生成一个 /index/index 的路由，允许通过 GET 或 POST 方式请求
    public function index(RequestInterface $request)
    {
        // 从请求中获得 id 参数
        $id = $request->input('id', 1);
        return (string)$id;
    }
}
```

### 通过 `@Controller` 注解定义路由
`@Controller` 为满足更细致的路由定义需求而存在，使用 `@Controller` 注解用于表名当前类为一个 `Controller类`，同时需配合 `@RequestMapping` 注解来对请求方法和请求路径进行更详细的定义。   
我们也提供了多种快速便捷的 `Mapping注解`，如 `@GetMapping`、`@PostMapping`、`@PutMapping`、`@PatchMapping`、`@DeleteMapping` 5种便捷的注解用于表明允许不同的请求方法。

> 使用 `@Controller` 注解时需 `use Hyperf\HttpServer\Annotation\Controller;` 命名空间；   
> 使用 `@RequestMapping` 注解时需 `use Hyperf\HttpServer\Annotation\RequestMapping;` 命名空间；   
> 使用 `@GetMapping` 注解时需 `use Hyperf\HttpServer\Annotation\GetMapping;` 命名空间；   
> 使用 `@PostMapping` 注解时需 `use Hyperf\HttpServer\Annotation\PostMapping;` 命名空间；   
> 使用 `@PutMapping` 注解时需 `use Hyperf\HttpServer\Annotation\PutMapping;` 命名空间；   
> 使用 `@PatchMapping` 注解时需 `use Hyperf\HttpServer\Annotation\PatchMapping;` 命名空间；   
> 使用 `@DeleteMapping` 注解时需 `use Hyperf\HttpServer\Annotation\DeleteMapping;` 命名空间；  

```php
<?php
declare(strict_types=1);

namespace App\Controller;

use Hyperf\HttpServer\Contract\RequestInterface;
use Hyperf\HttpServer\Annotation\Controller;
use Hyperf\HttpServer\Annotation\RequestMapping;

/**
 * @Controller()
 */
class IndexController
{
    // Hyperf 会自动为此方法生成一个 /index/index 的路由，允许通过 GET 或 POST 方式请求
    /**
     * @RequestMapping(path="index", methods="get,post")
     */
    public function index(RequestInterface $request)
    {
        // 从请求中获得 id 参数
        $id = $request->input('id', 1);
        return (string)$id;
    }
}
```


## 处理 HTTP 请求

`Hyperf` 是完全开放的，本质上没有规定您必须基于某种模式下去实现请求的处理，您可以采用传统的 `MVC模式`，亦可以采用 `RequestHandler模式` 来进行开发。   
我们以 `MVC模式` 来举个例子：   
在 `app` 文件夹内创建一个 `Controller` 文件夹并创建 `IndexController.php` 如下，`index` 方法内从请求中获取了 `id` 参数，并转换为 `字符串` 类型返回到客户端。

```php
<?php
declare(strict_types=1);

namespace App\Controller;

use Hyperf\HttpServer\Contract\RequestInterface;
use Hyperf\HttpServer\Annotation\AutoController;

/**
 * @AutoController()
 */
class IndexController
{
    // Hyperf 会自动为此方法生成一个 /index/index 的路由，允许通过 GET 或 POST 方式请求
    public function index(RequestInterface $request)
    {
        // 从请求中获得 id 参数
        $id = $request->input('id', 1);
        // 转换 $id 为字符串格式并以 plain/text 的 Content-Type 返回 $id 的值给客户端
        return (string)$id;
    }
}
```

## 依赖自动注入

依赖自动注入是 `Hyperf` 提供的一个非常强大的功能，也是保持框架灵活性的根基。   
`Hyperf` 提供了两种注入方式，一种是大家常见的通过构造函数注入，另一种是通过 `@Inject` 注解注入，下面我们举个例子并分别以两种方式展示注入的实现；   
假设我们存在一个 `\App\Service\UserService` 类，类中存在一个 `getInfoById(int $id)` 方法通过传递一个 `id` 并最终返回一个用户实体，由于返回值并不是我们这里所需要关注的，所以不做过多阐述，我们要关注的是在任意的类中获取 `UserService` 并调用里面的方法，一般的方法是通过 `new UserService()` 来实例化该服务类，但在 `Hyperf` 下，我们有更优的解决方法。

### 通过构造函数注入
只需在构造函数的参数内声明参数的类型，`Hyperf` 会自动注入对应的对象或值。
```php
<?php
declare(strict_types=1);

namespace App\Controller;

use Hyperf\HttpServer\Contract\RequestInterface;
use Hyperf\HttpServer\Annotation\AutoController;
use App\Service\UserService;

/**
 * @AutoController()
 */
class IndexController
{
    /**
     * @var UserService
     */
    private $userService;
    
    // 在构造函数声明参数的类型，Hyperf 会自动注入对应的对象或值
    public function __construct(UserService $userService)
    {
        $this->userService = $userService;
    }
    
    // /index/info
    public function info(RequestInterface $request)
    {
        $id = $request->input('id', 1);
        return $this->userService->getInfoById((int)$id);
    }
}
```

### 通过 `@Inject` 注解注入
只需对对应的类属性通过 `@var` 声明参数的类型，并使用 `@Inject` 注解标记属性 ，`Hyperf` 会自动注入对应的对象或值。

> 使用 `@Inject` 注解时需 `use Hyperf\Di\Annotation\Inject;` 命名空间；  

```php
<?php
declare(strict_types=1);

namespace App\Controller;

use Hyperf\HttpServer\Contract\RequestInterface;
use Hyperf\HttpServer\Annotation\AutoController;
use Hyperf\Di\Annotation\Inject;
use App\Service\UserService;

/**
 * @AutoController()
 */
class IndexController
{
    /**
     * @Inject()
     * @var UserService
     */
    private $userService;
    
    // /index/info
    public function info(RequestInterface $request)
    {
        $id = $request->input('id', 1);
        return $this->userService->getInfoById((int)$id);
    }
}
```
   
通过上面的示例我们不难发现 `$userService` 在没有实例化的情况下， 属性对应的类对象被自动注入了。   
不过这里的案例并未真正体现出依赖自动注入的好处及其强大之处，我们假设一下 `UserService` 也存在很多的依赖，而这些依赖同时又存在很多其它的依赖时，`new` 实例化的方式就需要手动实例化很多的对象并调整好对应的参数位，而在 `Hyperf` 里我们就无须手动管理这些依赖，只需要声明一下最终使用的类即可。   
而当 `UserService` 需要发生替换等剧烈的内部变化时，比如从一个本地服务替换成了一个 RPC 远程服务，也只需要通过配置调整依赖中 `UserService` 这个键值对应的类为新的RPC服务类即可。

## 启动 Hyperf 服务

由于 `Hyperf` 内置了协程服务器，也就意味着 `Hyperf` 将以 `CLI` 的形式去运行，所以在定义好路由及实际的逻辑代码之后，我们需要在项目根目录并通过命令行运行 `php bin/hyperf.php start` 来启动服务。   
当 `Console` 界面显示服务启动后便可通过 `cURL` 或 浏览器对服务正常发起访问了，默认情况下上面的例子是访问 `http://127.0.0.1:9501/index/info?id=1`。

## 重新加载代码

由于 `Hyperf` 是持久化的 `CLI` 应用，也就意味着一旦进程启动，已解析的 `PHP` 代码会持久化在进程中，也就意味着启动服务后您再修改的 `PHP` 代码不会改变已启动的服务，如您希望服务重新加载您修改后的代码，您需要通过在启动的 `Console` 中键入 `CTRL + C` 终止服务，再重新执行启动命令完成重启和重新加载。

> Tips: 您也可以将启动 Server 的命令配置在 IDE 上，便可直接通过 IDE 的 `启动/停止` 操作快捷的完成 `启动服务` 或 `重启服务` 的操作。
