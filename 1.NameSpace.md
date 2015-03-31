namespace 定义在 php文件顶部紧挨着 `<?php` 的下一行，顶级namespace必须全局唯一。

```php
<?php
namespace Oreilly;
```

sub-namespaces

```php
<?php
namespace Oreilly\ModernPHP
```

属于同个namespace的类不一定非得定义在一个文件下。可在每个文件顶部使用同一个namespace以标记这些文件在同一个命名空间下。

在php 引入 namespace之前，php开发者们使用 Zend-style 的类命名方式（源自 Zend 框架）。

即使用下划线来表示目录分隔，autoloader时自动替换下划线为当前OS的目录分隔符

```php
# Zend_Cloud_DocumentService_Adapter_WindowsAzure_Query => Zend/Cloud/DocumentService/Adapter/WindowsAzure/Query.php
```

这种方式对懒人来说的缺点就是写很多类名时太TM长了。

namespace提供了 import和 alias来解决这个问题。

import，alias 在5.3版本下支持类，接口与命名空间导入。5.6开始支持函数与常量导入。

```php
# namespace without alias
<?php
$response = new \Symfony\Component\HttpFoundation\Response('Oops',400);
$response->send();
$response2 = new \Symfony\Component\HttpFoundation\Response('Success',200);
```

```php
# namespace with alias 
use Symfony\Component\HttpFoundation\Response;
$response ＝ new Response('Oops',400);
$response->send();
```

```php
# namespace with custom alias 
use Symfony\Component\HttpFoundation\Response as Res;
$response ＝ new Res('Oops',400);
$response->send();
```

import与 alias的use与namespace的规则一样要在 php文件顶部紧挨着 `<?php` 的下一行或namespace的定义之后。

你没有必要在导入的命名空间的开始处加上`\`，php默认导入的命名空间为完全限定名称。

use关键字必须存在于全局作用域，因为它是在编译时被处理。

5.6之后可以导入函数与常量示例：

```php
<?php
use func Namespace\functionName;
functionName();
```

```php
<?php
use constant Namespace\CONST_NAME;
echo CONST_NAME;
```
多个namespace在一个文件中（这种做法极度不推荐）

```php
<?php
namespace Foo{

}

namespace Bar{

}
```

若没有附带namespace的调用类，方法或常量。php会默认这些属于当前命名空间下。

如果想指定namespace，则要使用完全限定名称来调用，或使用use来导入。

```php
<?php
namespace My\App
＃ Unqualified class name inside another namespace
class Foo {
    public function doSomething() {
        $exception = new Exception(); // php will search \My\App\Exception
    }
}
```

```php
<?php
namespace My\App
＃ Qualified class name inside another namespace
class Foo {
    public function doSomething() {
        $exception = new \Exception(); // php will search \Exception
    }
}
```

[namespace 官方手册参考](http://php.net/manual/en/language.namespaces.php) 

PS：中文版手册该章节有错误，请以英文版为准。
