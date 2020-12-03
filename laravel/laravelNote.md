## laravel 笔记

#### 一、修改数据表字段，`php artisan migrate` 报错

1. 修改数据表字段属性，需要 **doctrine/dbal** 组件
 - 安装命令：
`composer require doctrine/dbal`

2. 执行过上面的命令后，如果执行 `php artisan migrate` 还是报错
```
Migrating: 2020_12_02_201423_add_phone_to_users_table

   Error

  Class 'Doctrine\DBAL\Driver\PDOMySql\Driver' not found

  at vendor/laravel/framework/src/Illuminate/Database/MySqlConnection.php:64
    60|      * @return \Doctrine\DBAL\Driver\PDOMySql\Driver
    61|      */
    62|     protected function getDoctrineDriver()
    63|     {
  > 64|         return new DoctrineDriver;
    65|     }
    66| }
    67|

      +8 vendor frames
  9   database/migrations/2020_12_02_201423_add_phone_to_users_table.php:14
      Illuminate\Support\Facades\Facade::__callStatic("table")

      +21 vendor frames
  31  artisan:37
      Illuminate\Foundation\Console\Kernel::handle(Object(Symfony\Component\Console\Input\ArgvInput), Object(Symfony\Component\Console\Output\ConsoleOutput))
```

 - 则可能原因是 **doctrine/dbal** 版本太高
 - 降低 **doctrine/dbal** 版本，执行命令：
`composer require doctrine/dbal:^2.12.1`

二、form 表单正则校验，返回首页代码

1. 在进行正则校验的时候，如果输入错误的手机号，则会跳转到首页面
![form_regex](/images/sms_1.jpg)
![form_regex](/images/sms_2.jpg)
2. 原因是没有设置请求的 **header** 头
3. 增加设置 **header** 头的中间件
 1. 执行命令：`php artisan make:middleware AcceptHeader`
 2. 在 AcceptHeader.php 增加下列代码
 ```
 class AcceptHeader
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        // 给请求增加一个默认的 header 头
        $request->headers->set('Accept', 'application/json');

        return $next($request);
    }
}
```

  3. 在 App\Http\Kernel.php api 中间件组中增加中间件
```
protected $middlewareGroups = [
        'web' => [

        ],

        'api' => [
            'throttle:60,1',
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
            \App\Http\Middleware\AcceptHeader::class,
        ],
    ];
```

  4. 重新请求，效果如下
  ![form_regex](/images/sms_3.jpg)

