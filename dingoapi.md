
## Dingo/api在Laravel5.4的安装

1. 在composer.json的require中添加一行：

```
"dingo/api": "1.0.*@dev"
```

2. config/app.php添加：

```
Dingo\Api\Provider\LaravelServiceProvider::class
```

3. 在.env文件加上：

```
API_PREFIX=api
```
