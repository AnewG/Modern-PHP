# 过滤，验证与转义

任何用户数据都是不可信的。主要是以下几个来源

* `$_GET`
* `$_POST`
* `$_REQUEST`
* `$_COOKIE`
* `$argv`
* `php://stdin`
* `php://input`
* `file_get_contents()`
* `Remote databases`
* `Remote APIs`
* `Data from your clients`

## 过滤

### HTML

`htmlentities` 函数默认不转义引号与检测字符集，正确的用法应该是

```php
<?php
$input = '<p><script>alert("You won the Nigerian lottery!");</script></p>'; 
echo htmlentities($input, ENT_QUOTES, 'UTF-8');
```

过滤html输入，[HTML Purifier](http://htmlpurifier.org/) 是更好的选择。一些php模版引擎会自动完成过滤工作

不要自己写正则来过滤，正则又复杂又慢，html输入不一定是规范的，这一方法有很大的安全隐患。

### SQL

举一个例子

```php
$sql = sprintf(
    'UPDATE users SET password = "%s" WHERE id = %s',
    $_POST['password'],
    $_GET['id']
);
```

直接使用字符串作为语句简直就是灾难，直接一个伪造请求就能改掉用户密码。

```php
# POST /user?id=1 HTTP/1.1
#    Content-Length: 17
#    Content-Type: application/x-www-form-urlencoded
#    password=abc";--
```

PS：大部分数据库将 `--` 当作注释而忽略掉后面的内容。

更好的方式是使用 [PDO](http://php.net/manual/en/book.pdo.php)

### 杂项

过滤Email

```php
<?php
$email = 'john@example.com';
$emailSafe = filter_var($email, FILTER_SANITIZE_EMAIL);
```

移除 ASCII 32 之前的字符并转义 127 之后的

```php
<?php
$string = "\nIñtërnâtiônàlizætiøn\t"; $safeString = filter_var(
    $string,
    FILTER_SANITIZE_STRING,
    FILTER_FLAG_STRIP_LOW|FILTER_FLAG_ENCODE_HIGH
);
```

[查看更多filter_var用法](http://php.net/ manual/function.filter-var.php)

## 验证

与过滤不同，验证数据并不改变数据本身，只是检查数据是否符合你的期望。

可以使用 `filter_var` 配合 `FIL TER_VALIDATE_*` 标志位来验证数据。

```php
<?php
$input = 'john@example.com';
$isEmail = filter_var($input, FILTER_VALIDATE_EMAIL); 
if ($isEmail !== false) {
    echo "Success"; 
}else{
    echo "Fail"; 
}
```

以下是其他推荐的验证组件。

[aura/filter](https://packagist.org/packages/aura/filter)

[respect/validation](https://packagist.org/packages/respect/validation)

[symfony/validator](https://packagist.org/packages/symfony/validator)

# 密码安全

* 绝不明文存储
* 不通过邮件发送密码（一旦发送表示网站明文存储并且能读取用户密码），使用带有效时间并且需要验证的token附带在url中发送来代替发送密码。
* 使用 bcrypt 加密密码
* 尽可能使用 Password Hashing API

## 用户注册

请求：

```php
# POST /register.php HTTP/1.1
# Content-Length: 43
# Content-Type: application/x-www-form-urlencoded
# email=john@example.com&password=sekritshhh!
```

```php
<?php
try{
    // Validate email
    $email = filter_input(INPUT_POST, 'email', FILTER_VALIDATE_EMAIL);
    if (!$email) {
        throw new Exception('Invalid email');
    } 

    // Validate password
    $password = filter_input(INPUT_POST, 'password');
    if (!$password || mb_strlen($password) < 8) {
        throw new Exception('Password must contain 8+ characters');
    }

    // Create password hash
    $passwordHash = password_hash(
       $password,
       PASSWORD_DEFAULT,
       ['cost' => 12]
    );

    if ($passwordHash === false) {
        throw new Exception('Password hash failed');
    }

    // Create user account (THIS IS PSUEDO-CODE)
    $user = new User();
    $user->email = $email;
    $user->password_hash = $passwordHash;
    $user->save();

    // Redirect to login page
    header('HTTP/1.1 302 Redirect');
    header('Location: /login.php');

} catch (Exception $e) {
    // Report error
    header('HTTP/1.1 400 Bad request');
    echo $e->getMessage();
}
```

密码字段推荐 `varchar(255)`

