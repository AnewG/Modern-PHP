namespace定义在php文件顶部紧挨着 `<?php` 的下一行，顶级namespace必须全局唯一。

```php
<?php
namespace Oreilly;
```

subname-space

```php
<?php
namespace Oreilly\ModernPHP
```

属于同个namespace的类不一定非得定义在一个文件下。可在每个文件顶部都使用同一个namespace以标记这些文件在同一个命名空间下。

在使用namespace之前，php开发者使用 Zend-style 的类命名方式（源自 Zend 框架）。

使用下划线来表示目录分隔，autoloader时自动替换下划线为OS的目录分隔符

```php
# Zend_Cloud_DocumentService_Adapter_WindowsAzure_Query => Zend/Cloud/DocumentService/Adapter/WindowsAzure/Query.php
```

这种方式对懒人来说的缺点就是太TM长了。

namespace提供了import和alias来解决这个问题。

