## Laravel中的api.php文件

1. 路由会自动附加前缀'api/'
2. 路由会使用`auth`和`api`中间件，验证token，api中间件附加了额外的安全保护协议，并且限制api的请求频率
