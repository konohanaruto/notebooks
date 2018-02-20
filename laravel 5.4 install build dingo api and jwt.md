在此篇文章中，我将展示怎样去整合使用DingoApi和JwtAuth包，怎样初步构建一个api应用，我将使用Laravel框架(基于5.4版本)，我们使用composer作为包管理器。最后，建议安装postman工具以便我们方便的测试我们的api。

## 安装 [Dingo API](https://github.com/dingo/api)

1. 编辑laravel的composer.json文件，在`require`代码段加入一行

```
"require": {
    "php": ">=5.6.4",
    "laravel/framework": "5.4.*",
    "laravel/tinker": "~1.0",
    "dingo/api": "1.0.*@dev"
},
```

2. composer安装它, 我们使用-vvv选项打印安装详细过程

```
composer update -vvv
```

3. 发布相应的配置文件
```
php artisan vendor:publish --provider="Dingo\Api\Provider\LaravelServiceProvider"
```

4. 在config/app.php的`providers`数组添加一行

```
Dingo\Api\Provider\LaravelServiceProvider::class,
```
将会输出  
```
Copied File [\vendor\dingo\api\config\api.php] To [\config\api.php]  
Publishing complete.
```


## 安装[JWT](https://github.com/tymondesigns/jwt-auth)

1. 编辑laravel的composer.json文件，在`require-dev`(尤其注意，和上面dingo不同)代码段加入一行
```
"require-dev": {
    "fzaninotto/faker": "~1.4",
    "mockery/mockery": "0.9.*",
    "phpunit/phpunit": "~5.7",
    "tymon/jwt-auth": "1.0.*@dev"
},
```
2. composer安装它, 我们使用-vvv选项打印安装详细过程

```
composer update -vvv
```
3. 在config/app.php的`providers`数组添加一行
```
Tymon\JWTAuth\Providers\LaravelServiceProvider::class,
```
4. 发布配置文件
```
php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\LaravelServiceProvider"
```
5. 生成一个jwt key
```
php artisan jwt:secret
```
## 整合
1. 配置auth guard, 因此，我们打开config/auth.php文件，确保如下两处更改
```
'defaults' => [
    'guard' => 'api',
    'passwords' => 'users',
],

...

'guards' => [
    'api' => [
        'driver' => 'jwt',
        'provider' => App\User::class # 使用app/User.php作为provider
    ],
],
```

2. 更新app/User.php文件，替换成如下内容
```
namespace App;

use Tymon\JWTAuth\Contracts\JWTSubject;
use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable implements JWTSubject
{
    use Notifiable;

    // Rest omitted for brevity

    /**
     * Get the identifier that will be stored in the subject claim of the JWT.
     *
     * @return mixed
     */
    public function getJWTIdentifier()
    {
        return $this->getKey();
    }

    /**
     * Return a key value array, containing any custom claims to be added to the JWT.
     *
     * @return array
     */
    public function getJWTCustomClaims()
    {
        return [];
    }
}
```
3. 修改一些配置，打开app/Providers/AppServiceProvider.php文件
```
use Illuminate\Support\Facades\Schema;
// 省略...
public function boot()
{
    // 添加如下行
    Schema::defaultStringLength(191);
}
```
4. 创建用户表
```
php artisan migrate
```
5. 使用thinker命令创建用户，确保你laravel的.env文件已正确的配置了数据库信息
```
php artisan tinker
// 粘贴如下内容
$user = new User;
$user->email = 'admin@test.com';
$user->name = 'admin';
$user->password = Hash::make('123456');
$user->save();
```
6. 编辑.env文件，设置dingo相关
```
API_PREFIX=api
API_VERSION=v2
API_SUBTYPE=myapp
API_STANDARDS_TREE=vnd
```

7. 创建Auth控制器

```
php artisan make:controller AuthController
```

8. 编辑我们刚生成的控制器app/Http/Controllers/AuthController.php文件，将如下代码粘贴
```
namespace App\Http\Controllers;

use Illuminate\Support\Facades\Auth;
use App\Http\Controllers\Controller;

class AuthController extends Controller
{
    /**
     * Create a new AuthController instance.
     *
     * @return void
     */
    public function __construct()
    {
        $this->middleware('auth:api', ['except' => ['login']]);
    }

    /**
     * Get a JWT via given credentials.
     *
     * @return \Illuminate\Http\JsonResponse
     */
    public function login()
    {
        $credentials = request(['email', 'password']);

        if (! $token = auth()->attempt($credentials)) {
            return response()->json(['error' => 'Unauthorized'], 401);
        }

        return $this->respondWithToken($token);
    }

    /**
     * Get the authenticated User.
     *
     * @return \Illuminate\Http\JsonResponse
     */
    public function me()
    {
        return response()->json(auth()->user());
    }

    /**
     * Log the user out (Invalidate the token).
     *
     * @return \Illuminate\Http\JsonResponse
     */
    public function logout()
    {
        auth()->logout();

        return response()->json(['message' => 'Successfully logged out']);
    }

    /**
     * Refresh a token.
     *
     * @return \Illuminate\Http\JsonResponse
     */
    public function refresh()
    {
        return $this->respondWithToken(auth()->refresh());
    }

    /**
     * Get the token array structure.
     *
     * @param  string $token
     *
     * @return \Illuminate\Http\JsonResponse
     */
    protected function respondWithToken($token)
    {
        return response()->json([
            'access_token' => $token,
            'token_type' => 'bearer',
            'expires_in' => auth()->factory()->getTTL() * 60
        ]);
    }
}
```

9. 打开routes/api.php, 在尾部粘贴如下内容，添加路由
```
$api = app('Dingo\Api\Routing\Router');
$api->version('v2', function ($api) {
    $api->group(['middleware' => 'auth:api', 'namespace' => 'App\Http\Controllers\Api\v2'], function ($api) {
        $api->get('/test', 'HomeController@test');
    });

    $api->group(['middleware' => 'auth:api', 'namespace' => 'App\Http\Controllers'], function ($api) {
        $api->get('/refresh', 'AuthController@refresh');
        $api->get('/me', 'AuthController@me');
        $api->get('/logout', 'AuthController@logout');
    });

    $api->get('/login', '\App\Http\Controllers\AuthController@login');
});
```

10. 创建必要的控制器
```
php artisan make:controller app/Http/Controllers/Api/v2/HomeController
```

11. 打开app/Http/Controllers/Api/v2/HomeController.php, 添加一个test方法

```
namespace App\Http\Controllers\Api\v2;

use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class HomeController extends Controller
{
    public function test()
    {
        return ['name' => 'konohanaruto'];
    }
}
```

## 使用postman测试

1. 访问yourdomain/api/test, 得到

```
{"message":"Unauthenticated.","status_code":500}
```

2. 我们需要登录，于是我们访问yourdomain/api/login, 并在postman的参数加入我们之前创建用户所用的email和password, 得到

```
{"access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOlwvXC9sYXJhdmVsLndlYlwvYXBpXC9sb2dpbiIsImlhdCI6MTUxOTA4NzU1NCwiZXhwIjoxNTE5MDkxMTU0LCJuYmYiOjE1MTkwODc1NTQsImp0aSI6IjRUUDNPeUFRUFBRVWQ0OTMiLCJzdWIiOjEsInBydiI6Ijg3ZTBhZjFlZjlmZDE1ODEyZmRlYzk3MTUzYTE0ZTBiMDQ3NTQ2YWEifQ.cYhzrgtxNOi6sKHh-R76tH9oZfXJGs3HgaIyytbha8M","token_type":"bearer","expires_in":3600}
```

3. 再次访问yourdomain/api/test， 并使用postman携带参数token, 也就是上方得到的这个access_token的值，得到

```
{"name":"konohanaruto"}
```

这里仅仅测试了登录后的api调用，暂时没有演示token失效以及其它功能， 后续将会补充， 敬请期待~



