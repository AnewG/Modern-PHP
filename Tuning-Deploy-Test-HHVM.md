# 调优

## php.ini

### 内存

`memory_limit`

Q：总共能分配多少内存给PHP ? 

A：假设一台 2GB 内存的 Linode 主机，除 php 外的其他进程，比如 nginx 或 MySQL 都部署在同一台机子上。配置 512M 给 php 是一个不错的选择。

Q：平均来说，一个php进程占用多少内存？

A：命令行使用 `top` ，或者调用php函数 `memory_get_peak_usage()` 来查看内存使用情况。以我的观察，大概一个占 5-20MB左右内存。（个人情况，仅供参考）

而且如果正在进行文件上传等操作时，内存占用会持续上升。

Q：PHP-FPM 开多少进程合适?

A：以分配 512 MB 内存来说。大概分配 34 个 PHP-FPM 进程（同样个人情况，仅供参考）

可以使用 [Apache Bench](https://httpd.apache.org/docs/2.2/programs/ab.html) 或 [seige](http://www.joedog.org/siege-home/) 来进行压力测试

### 文件上传

需要文件上传吗？如果不需要，直接关掉它提高安全系数。

```php
file_uploads = 1 // Whether or not to allow HTTP file uploads
upload_max_filesize = 10M
max_file_uploads = 3
```

### 脚本最大执行时间

建议 `max_execution_time = 5` 或 `set_time_limit()` 函数, 长时间运行的任务请丢给[队列](https://github.com/chrisboulton/php-resque)慢慢消化。

```php
<?php
exec('echo "create-report.php" | at now');  // The standalone create-report.php script runs in a separate background process;
echo 'Report pending...';
```

### 设置session解决方案

```php
session.save_handler = 'memcached'
session.save_path = '127.0.0.2:11211'
```

### 输出缓存

php默认开启了输出缓存（非命令行下）

```php
output_buffering = 4096
implicit_flush = false
// This is equivalent to calling the PHP function flush() after each and every call to print or echo and each and every HTML block. false by default
```

如果更改了 output_buffering ，确保它是4的倍数 (32位系统) 或8的倍数 (64位系统).

### Realpath Cache

php会对文件路径做缓存来避免每次在 include path 中搜索。默认大小为 16k。

```php
realpath_cache_size = 64k
```

# 部署

[Capistrano](http://capistranorb.com/)

[Magallanes](http://magephp.com/)

[Rocketeer](http://rocketeer.autopergamene.eu/#/docs/rocketeer/README)

# 测试

## 单元测试

单元测试主要针对独立的类，方法和函数，最常用的单元测试套件为 [PHPUnit](https://phpunit.de/) 。

## 测试驱动开发（Test-driven development）TDD

是极限编程中倡导的程序开发方法，以其倡导先写测试程序，然后编码实现其功能得名。

测试驱动开发是戴两顶帽子思考的开发方式：先戴上实现功能的帽子，在测试的辅助下，快速实现其功能；再戴上重构的帽子，在测试的保护下，通过去除冗余的代码，提高代码质量。测试驱动着整个开发过程：首先，驱动代码的设计和功能的实现；其后，驱动代码的再设计和重构。

正面评价:

可以有效的避免过度设计带来的浪费。但是也有人强调在开发前需要有完整的设计再实施可以有效的避免重构带来的浪费。
可以让开发者在开发中拥有更全面的视角。

负面评价:

开发者可能只完成满足了测试的代码，而忽略了对实际需求的实现。有实践者认为用结对编程的方式可以有效的避免这个问题。
会放慢开发实际代码的速度，特别对于要求开发速度的原型开发造成不利。这里需要考虑开发速度需要包含功能和品质两个方面，单纯的代码速度可能不能完全代表开发速度。

对于GUI,资料库和Web应用而言。构造单元测试比较困难，如果强行构造单元测试，反而给维护带来额外的工作量。有开发者认为这个是由于设计方法，而不是开发方法造成的困难。

使得开发更为关注用例和测试案例，而不是设计本身。目前，对于这个观点有较多的争议。

测试驱动开发会导致单元测试的覆盖度不够，比如可能缺乏边界测试。在实际的操作中，和非测试驱动开发一样，当代码完成以后还是需要补充单元测试，提高测试的覆盖度。

## 行为驱动开发（Behavior-driven development）BDD

BDD的重点是通过与利益相关者的讨论取得对预期的软件行为的清醒认识。它通过用自然语言书写非程序员可读的测试用例扩展了测试驱动开发方法。行为驱动开发人员使用混合了领域中统一的语言的母语语言来描述他们的代码的目的。这让开发者得以把精力集中在代码应该怎么写，而不是技术细节上，而且也最大程度的减少了将代码编写者的技术语言与商业客户、用户、利益相关者、项目管理者等的领域语言之间来回翻译的代价。

[Behat](http://docs.behat.org/en/v2.5/)

[PHPspec](http://www.phpspec.net/en/latest/)

