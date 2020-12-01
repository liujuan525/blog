# 短信插件安装

## 短信运营商使用 aliyun

 1、进入管理后台，添加签名

国内消息 -> 签名管理 -> 添加签名

 2、进入管理后台，添加模板

国内消息 -> 模板管理 -> 添加模板

3、获取 AccessKey ID 与 AccessKey Secret

在 阿里云 -> 短信服务 -> 概览 -> AccessKey 中 获取

4、在 .env 文件中增加短信配置项(涉及到私人信息，不可提交到 git)

```
# aliyun 短信
SMS_ALIYUN_ACCESS_KEY_ID=AccessKey ID
SMS_ALIYUN_ACCESS_KEY_SECRET=AccessKey Secret
```

5、在 .env.example 中添加上面的配置项（可同步到 git）
```
# aliyun 短信
SMS_ALIYUN_ACCESS_KEY_ID=
SMS_ALIYUN_ACCESS_KEY_SECRET=
```

## 代码层面

备注：本插件目前是基于 laravel 框架进行安装的

1、composer 安装 easysms 

`composer require "overtrue/easy-sms"`

注意：composer 最好使用最新版本，如果使用版本过老会安装失败，且会报错：

`Warning from https://repo.packagist.org: You are using an outdated version of Composer. Composer 2.0 is now available and you should upgrade. See https://getcomposer.org/2`

2、升级 composer 版本

升级命令：`composer self-update`

版本回滚：`composer self-update --rollback`

3、由于该组件还没有 Laravel 的 ServiceProvider，对其进行封装

在 config 目录中创建 easysms.php

`touch config/easysms.php`

代码内容如下：
```
<?php

return [
    // HTTP 请求的超时时间（秒）
    'timeout' => 10.0,

    // 默认发送配置
    'default' => [
        // 网关调用策略，默认：顺序调用
        'strategy' => \Overtrue\EasySms\Strategies\OrderStrategy::class,

        // 默认可用的发送网关
        'gateways' => [
            'aliyun',
        ],
    ],
    // 可用的网关配置
    'gateways' => [
        'errorlog' => [
            'file' => '/tmp/easy-sms.log',
        ],
        'aliyun' => [
            'access_key_id' => env('SMS_ALIYUN_ACCESS_KEY_ID'),
            'access_key_secret' => env('SMS_ALIYUN_ACCESS_KEY_SECRET'),
            'sign_name' => 'Larabbs',
        ],
    ],
];
```

4、创建 EasySmsServiceProvider.php 

`php artisan make:provider EasySmsServiceProvider`

修改文件 app/providers/EasySmsServiceProvider.php 

```
<?php

namespace App\Providers;

use Overtrue\EasySms\EasySms;
use Illuminate\Support\ServiceProvider;

class EasySmsServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap the application services.
     *
     * @return void
     */
    public function boot()
    {
        //
    }

    /**
     * Register the application services.
     *
     * @return void
     */
    public function register()
    {
        $this->app->singleton(EasySms::class, function ($app) {
            return new EasySms(config('easysms'));
        });
        $this->app->alias(EasySms::class, 'easysms');
    }
}
```

5、在 config/app.php 文件中的 providers 中注入 EasySmsServiceProvider 

```
/** send sms provider */
App\Providers\EasySmsServiceProvider::class,
```

## 短信测试
通过 `php artisan tinker` 进行测试

```
$sms = app('easysms');
try {
    $sms->send(手机号, [
         'template' => '模版CODE',
         'data' => [
             'code' => 1234
         ],
    ]);
} catch (\Overtrue\EasySms\Exceptions\NoGatewayAvailableException $exception) {
    $message = $exception->getException('aliyun')->getMessage();
    dd($message);
}
```

备注：部分代码来源于： L03 Laravel 教程 - 实战构架 API 服务器 



