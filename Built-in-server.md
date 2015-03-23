php 5.4之后引入了内建的http服务器。

使用方式：

```php
# 在命令行界面切换到你的项目目录，在此启动该目录将作为服务器的根目录。
# php -S localhost:4000
# 在 localhost 上绑定 4000 端口
# 也可用 0.0.0.0 监听任意ip
# 带自定义配置项的启动 php -S localhsot:8000 -c app/config/php.ini
```

内建的服务器并不支持 `.htaccess`。php使用 router scripts 来解决这一问题。

```php
php -S localhost:8000 router.php
```

该脚本将在每一次请求前调用。如果脚本return false，与请求相匹配的静态文件将返回，否则将返回router.php返回的内容。

检测内建服务器的方式：

```php
<?php
if (php_sapi_name() === 'cli-server') {
    // PHP web server
}else{
    // Other web server
}
```

缺点：

1.只能同时处理一个请求。
2.mimetypes有限。
3.router script的rewrite规则是有限的。
