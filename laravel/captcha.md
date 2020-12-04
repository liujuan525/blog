#### 图片验证码

- `composer` 安装 `gregwar/captcha`

 `composer require gregwar/captcha`

- 控制器

   - 创建控制器

   `php artisan make:controller Api/CaptchasController`

   - 编辑控制器
   
    ```
    <?php

    namespace App\Http\Controllers\Api;

    use App\Http\Controllers\Controller;
    use App\Http\Requests\Api\CaptchaRequest;
    use Gregwar\Captcha\CaptchaBuilder;
    use Illuminate\Support\Str;

    class CaptchasController extends Controller
    {
        public function store(CaptchaRequest $request, CaptchaBuilder $captchaBuilder)
        {
            $key = 'captcha-' . Str::random(15);
            $phone = $request->phone;
            // build() 创建图片验证码
            $captcha = $captchaBuilder->build();
            $expireAt = now()->addMinutes(2);
            // getPhrase() 获取验证码文本
            \Cache::put($key, ['phone' => $phone, 'code' => $captcha->getPhrase()], $expireAt);

            $result = [
                'captcha_key' => $key,
                'expired_at'  => $expireAt->toDateTimeString(),
                // inline() 获取 base64 的图片验证码
                'captcha_image_content' => $captcha->inline()
            ];
            return response()->json($result)->setStatusCode(201);
        }
    }
    ```

- 表单验证类

   - 创建表单验证类

   `php artisan make:request Api/CaptchaRequest`

   - 编辑 表单验证类

    ```
    <?php

    namespace App\Http\Requests\Api;

    use Illuminate\Foundation\Http\FormRequest;

    class CaptchaRequest extends FormRequest
    {
        /**
         * Get the validation rules that apply to the request.
         *
         * @return array
         */
        public function rules()
        {
            return [
                'phone' => [
                    'required',
                    'regex:/^((13[0-9])|(14[5,7])|(15[0-3,5-9])|(17[0,3,5-8])|(18[0-9])|166|198|199|(147))\d{8}$/',
                    'unique:users'
                ]
            ];
        }
    }
    ```

- 添加路由

   - 在 routes/api.php 添加：
   
   `Route::post('captchas', 'CaptchasController@store')->name('captchas.store');`

- 通过 `postman` 测试接口

   - 请求路由：

   `{{may}}/api/v1/captchas`

   - 请求参数：
   ```
   {
     "phone":"15830754764"
   }
   ```

   - 返回数据：
   ```
   {
    "captcha_key": "captcha-BhSP0BbrlKD7l2Z",
    "expired_at": "2020-12-03 20:38:11",
    "captcha_image_content": "data:image/jpeg;base64,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"
   }
   ```

备注： 参考 `L03 Laravel 教程 - 实战构架 API 服务器 ( Laravel 7.x ) `








