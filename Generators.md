php 5.5之后引入了 generators。

不像标准的php迭代器，generators不仅不需要你实现繁重的迭代器接口，而且还能按需求值，提高了性能，特别是在数据量大的时候。

generators 也并非灵丹妙药。除非你调用，否则它并不知道下一次迭代的值。并且整体只能迭代一次。

Example:

```php
<?php
function myGenerator() {
yield 'value1'; yield 'value2'; yield 'value3';
}
```
