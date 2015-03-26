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
