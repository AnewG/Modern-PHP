闭包与匿名函数在 php 5.3 之后被引入。

闭包指的是一种函数保留外部变量（创建该函数时外部变量的值）的技术。一般作为回调函数使用。

Example:

```php
<?php
$message = 'hello';

$example = function () {
    var_dump($message); // NULL
};
echo $example();

$example = function () use ($message) {
    var_dump($message); // hello
};
echo $example();

// Inherited variable's value is from when the function
// is defined, not when called
$message = 'world';
echo $example(); // also hello

// Reset message
$message = 'hello';

// Inherit by-reference
$example = function () use (&$message) {
    var_dump($message);
};
echo $example();

// The changed value in the parent scope
// is reflected inside the function call
$message = 'world';
echo $example(); // world

// Closures can also accept regular arguments
$example = function ($arg) use ($message) {
    var_dump($arg . ' ' . $message);
};
$example("hello"); // "hello world"
?>
```

闭包和匿名函数（其实就是没定义名字的函数）其实是不同的东西，但在PHP里他们是一样的。

匿名函数赋值给变量后，创建了closure对象(每个闭包都是一个 closure 对象实例)，其实现了 invoke 魔术方法，所以可以通过直接调用变量名来调用该匿名函数。
