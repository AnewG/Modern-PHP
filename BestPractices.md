过滤，验证与转义

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
