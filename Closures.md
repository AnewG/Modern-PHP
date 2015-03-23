闭包与匿名函数在 php 5.3 之后被引入。

闭包指的是一种函数保留外部变量（创建该函数时外部变量的值）的技术。

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
