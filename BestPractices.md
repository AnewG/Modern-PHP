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
* 尽可能使用 Password Hashing API(如无法使用 php 5.5 以上，可以使用 [password-compat](https://packagist.org/packages/ircmaxell/password-compat) )

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

## 用户登陆

请求：

```php
# POST /login.php HTTP/1.1
# Content-Length: 43
# Content-Type: application/x-www-form-urlencoded
# email=john@example.com&password=sekritshhh!
```

```php
<?php
session_start(); 

try{
    // Get email address from request body
    $email = filter_input(INPUT_POST, 'email');

    // Get password from request body
    $password = filter_input(INPUT_POST, 'password');

    // Find account with email address (THIS IS PSUEDO-CODE)
    $user = User::findByEmail($email);

    // Verify password with account password hash
    if (password_verify($password, $user->password_hash) === false) {
        throw new Exception('Invalid password');
    }

    // Re-hash password if necessary(see not below)
    $currentHashAlgorithm = PASSWORD_DEFAULT;
    $currentHashOptions = array('cost' => 15);
    $passwordNeedsRehash = password_needs_rehash(
        $user->password_hash,
        $currentHashAlgorithm,
        $currentHashOptions
    );

    if ($passwordNeedsRehash === true) {
        // Save new password hash (THIS IS PSUEDO-CODE)

        $user->password_hash = password_hash(
            $password,
            $currentHashAlgorithm,
            $currentHashOptions
        );
        $user->save();
    }

    // Save login status to session
    $_SESSION['user_logged_in'] = 'yes';
    $_SESSION['user_email'] = $email;

    // Redirect to profile page
    header('HTTP/1.1 302 Redirect');
    header('Location: /user-profile.php');
} catch (Exception $e) {
    header('HTTP/1.1 401 Unauthorized');
    echo $e->getMessage();
}
```

# 时间处理

php 5.2 之后推荐使用 `DateTime`,`DateInterval`,`DateTimeZone`来处理时间相关的操作。

第一步，设置时区。两种方式

```php
# declare the default time zone in the php.ini file like this:date.timezone = 'America/New_York';
date_default_timezone_set('America/New_York');
```

第二步，实例化

```php
# $datetime = new DateTime(); 
# Without arguments, the DateTime class constructor creates an instance that repre‐ sents the current date and time.
$datetime = new DateTime('2014-04-27 5:03 AM');
```

时间的来源格式不尽相同，使用 `DateTime::createFromFormat()` 静态方法来格式化并创建实例

```php
$datetime = DateTime::createFromFormat('M j, Y H:i:s', 'Jan 2, 2014 23:04:12');
# strtotime ('Jan 2, 2014 23:04:12');
```

使用 `DateInterval` ，`DatePeriod` 来处理时间间隔。

* Y (years)
* M (months)
* D (days)
* W (weeks)
* H (hours)
* M (minutes) 
* S (seconds)

举例，P2D 表示两天。P2DT5H2M 两天五小时两分钟。

```php
<?php
// Create DateTime instance
$datetime = new DateTime('2014-01-01 14:00:00');
// Create two weeks interval
$interval = new DateInterval('P2W');
// Modify DateTime instance
$datetime->add($interval);
echo $datetime->format('Y-m-d H:i:s');
```

```php
$dateStart = new \DateTime();
$dateInterval = \DateInterval::createFromDateString('-1 day'); 
$datePeriod = new \DatePeriod($dateStart, $dateInterval, 3); 
foreach ($datePeriod as $date) {
    echo $date->format('Y-m-d'), PHP_EOL; 
}
# This outputs:
#     2014-12-08
#     2014-12-07
#     2014-12-06
#     2014-12-05
```

```php
<?php
$start = new DateTime();
$interval = new DateInterval('P2W');
$period = new DatePeriod($start, $interval, 3);
foreach ($period as $nextDateTime) {
    echo $nextDateTime->format('Y-m-d H:i:s'), PHP_EOL;
}
```

第三方组件 [Cabon](https://github.com/briannesbitt/Carbon)

切换不同时区的话，使用 `DateTimeZone`

```php
<?php
$timezone = new DateTimeZone('America/New_York'); 
$datetime = new DateTime('2014-08-20', $timezone);

$timezone = new DateTimeZone('America/New_York'); 
$datetime = new \DateTime('2014-08-20', $timezone); 
$datetime->setTimezone(new DateTimeZone('Asia/Hong_Kong'));
```

# 数据库

使用 PDO 来处理数据库操作

通过配置 DSN 使得 PDO 将 php 与数据库连接起来

DSN 配置由数据库驱动名开头 (e.g., mysql or sqlite),每种数据库格式不尽相同，但大体都包含以下信息:

* Hostname or IP address
* Port number
* Database name
* Character set

[DSN格式参考](http://php.net/manual/pdo.drivers.php)

```php
<?php 
try {
    $pdo = new PDO( 'mysql:host=127.0.0.1;dbname=books;port=3306;charset=utf8', 'USERNAME','PASSWORD');
} catch (PDOException $e) {
    // Database connection failed
    echo "Database connection failed";
    exit; 
}
```

连接信息的隐私性要注意保护，不要加到版本控制里，更别放到公共仓库里。

使用 prepare 语句来绑定查询变量，保证安全性

```php
<?php
$sql = 'SELECT id FROM users WHERE email = :email';
$statement = $pdo->prepare($sql);
$email = filter_input(INPUT_GET, 'email');
$statement->bindValue(':email', $email); // default string type

$sql = 'SELECT email FROM users WHERE id = :id';
$statement = $pdo->prepare($sql);
$userId = filter_input(INPUT_GET, 'id');
$statement->bindValue(':id', $userId, PDO::PARAM_INT);
```

* PDO::PARAM_BOOL
* PDO::PARAM_NULL
* PDO::PARAM_INT
* PDO::PARAM_STR (default)
    
[一些PDO预定义常量](http://php.net/manual/en/pdo.constants.php)

## 获取查询结果

通过 execute 执行 SQL 之后，若是非 INSERT，UPDATE或DELETE操作，你还需获取数据库返回的纪录。

```php
<?php
$pdo = new PDO("mysql:host=localhost;dbname=world", 'my_user', 'my_pass');
$pdo->setAttribute(PDO::MYSQL_ATTR_USE_BUFFERED_QUERY, false);
// 缓存结果集到php端将有更多的操作可用，但内存占用也会加大
// 不缓存到php端，则使用连接资源向数据库端每次获取数据，虽然php端内存压力小了，但是会明显加大数据库端的负载。
// 对于同一个连接来说，除非结果集被服务端获取完，否则将无法返回其他查询结果。

$uresult = $pdo->query("SELECT Name FROM City");
if ($uresult) {
   while ($row = $uresult->fetch(PDO::FETCH_ASSOC)) {
       echo $row['Name'] . PHP_EOL;
   }
}
?>
```

[其他的FETCH_STYLE](http://php.net/manual/zh/pdostatement.fetch.php)

```php
<?php
// Build and execute SQL query
$sql = 'SELECT id, email FROM users WHERE email = :email'; 
$statement = $pdo->prepare($sql);
$email = filter_input(INPUT_GET, 'email'); 
$statement->bindValue(':email', $email, PDO::PARAM_INT); $statement->execute();
// Iterate results
$results = $statement->fetchAll(PDO::FETCH_ASSOC); 
// results 已经包含全部结果集
foreach ($results as $result) {
    echo $result['email']; 
}
```

获取指定列结果集

```php
<?php
// Build and execute SQL query
$sql = 'SELECT id, email FROM users WHERE email = :email'; 
$statement = $pdo->prepare($sql);
$email = filter_input(INPUT_GET, 'email'); 
$statement->bindValue(':email', $email, PDO::PARAM_INT); 
$statement->execute();
// Iterate results
// The query result column order matches the column order specified in the SQL query.
while (($email = $statement->fetchColumn(1)) !== false) { 
    echo $email;
}
```

## 事务

PDO 同样支持事务

```php
<?php
require 'settings.php';
// PDO connection
try {
    $pdo = new PDO(
            sprintf(
                'mysql:host=%s;dbname=%s;port=%s;charset=%s',
                $settings['host'],
                $settings['name'],
                $settings['port'],
                $settings['charset']
            ),
            $settings['username'],
            $settings['password']
    );
} catch (PDOException $e) {
    // Database connection failed
    echo "Database connection failed";
    exit; 
}
// Statements
$stmtSubtract = $pdo->prepare('
    UPDATE accounts
    SET amount = amount - :amount
    WHERE name = :name
');
    $stmtAdd = $pdo->prepare('
    UPDATE accounts
    SET amount = amount + :amount
    WHERE name = :name
');
// Start transaction
$pdo->beginTransaction();
// Withdraw funds from account 1
$fromAccount = 'Checking';
$withdrawal = 50;
$stmtSubtract->bindParam(':name', $fromAccount);
$stmtSubtract->bindParam(':amount', $withDrawal, PDO::PARAM_INT);
$stmtSubtract->execute();
// Deposit funds into account 2
$toAccount = 'Savings';     
$deposit = 50;
$stmtAdd->bindParam(':name', $toAccount);
$stmtAdd->bindParam(':amount', $deposit, PDO::PARAM_INT);
$stmtAdd->execute();
// Commit transaction
$pdo->commit();
```
