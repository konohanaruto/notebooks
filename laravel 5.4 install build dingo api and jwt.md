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
php artisan jwt:generate
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
php artisan thinker
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
