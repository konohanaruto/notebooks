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
3. 在config/app.php的`providers`数组添加一行

```
Dingo\Api\Provider\LaravelServiceProvider::class,
```

4. 发布相应的配置文件
```
php artisan vendor:publish --provider="Dingo\Api\Provider\LaravelServiceProvider"
```

## 安装JWT

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
3. 发布配置文件
```
php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\LaravelServiceProvider"
```
4. 在config/app.php的`providers`数组添加一行
```
Tymon\JWTAuth\Providers\JWTAuthServiceProvider::class,
```
5. 生成一个jwt key
```
php artisan jwt:generate
```
