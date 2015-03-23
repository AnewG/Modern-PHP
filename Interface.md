无规矩不成方圆。你如果有一组类有类似的“行为”，需要对他们进行约束的话，接口必不可少了。

定好规范，任第三份随意实现代码。效率可高、可低，实现内容也可随时间变化而变化，扩展性高。

但是调用关系是不变的。所以即使以后更换组件，只要实现了接口的组件，调用逻辑都不需要变化。

```php
# Example
class DocumentStore {
    protected $data = [];
    public function addDocument(Documentable $document) {
        $key = $document->getId();
        $value = $document->getContent();
        $this->data[$key] = $value;
    }
    public function getDocuments() {
        return $this->data; 
    }
}

interface Documentable {
    public function getId(); 
    public function getContent();
}

class HtmlDocument implements Documentable {
    protected $url;
    public function __construct($url) {
        $this->url = $url;
    }
    public function getId() {
        return $this->url; 
    }
    public function getContent() {
        $ch = curl_init();
        curl_setopt($ch, CURLOPT_URL, $this->url);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
        curl_setopt($ch, CURLOPT_CONNECTTIMEOUT, 3);
        curl_setopt($ch, CURLOPT_FOLLOWLOCATION, 1);
        curl_setopt($ch, CURLOPT_MAXREDIRS, 3);
        $html = curl_exec($ch);
        curl_close($ch);
        return $html; 
    }
}

class StreamDocument implements Documentable {
    protected $resource; protected $buffer;
    public function __construct($resource, $buffer = 4096) {
        $this->resource = $resource;
        $this->buffer = $buffer;
    }
    public function getId() {
        return 'resource-' . (int)$this->resource; 
    }
    public function getContent() {
        $streamContent = ''; rewind($this->resource);
        while (feof($this->resource) === false) {
            $streamContent .= fread($this->resource, $this->buffer);
        }
        return $streamContent; 
    }
}

class CommandOutputDocument implements Documentable {
    protected $command;
    public function __construct($command) {
        $this->command = $command;
    }
    public function getId() {
        return $this->command; 
    }
    public function getContent() {
        return shell_exec($this->command); 
    }
}

$documentStore = new DocumentStore();
// Add HTML document
$htmlDoc = new HtmlDocument('https://php.net'); 
$documentStore->addDocument($htmlDoc);
// Add stream document
$streamDoc = new StreamDocument(fopen('stream.txt', 'rb')); 
$documentStore->addDocument($streamDoc);


// Add terminal command document
$cmdDoc = new CommandOutputDocument('cat /etc/hosts'); 
$documentStore->addDocument($cmdDoc);
print_r($documentStore->getDocuments());
```

5.4 新加入了 Traits，它既不是接口也不是类。主要是为了解决单继承语言的限制。与 Ruby 的 composable modules 或 mixins 类似。

它能被加入到一个或多个已经存在的类中。它声明了类能做什么（接口特性），同时也包含了具体实现（类特性）

Example:

```php
<?php
trait Geocodable {
    /** @var string */
    protected $address;
    /** @var \Geocoder\Geocoder */
    protected $geocoder;
    /** @var \Geocoder\Result\Geocoded */
    protected $geocoderResult;
    public function setGeocoder(\Geocoder\GeocoderInterface $geocoder) {
        $this->geocoder = $geocoder;
    }
    public function setAddress($address) {
        $this->address = $address;
    }
    public function getLatitude(){
        if (isset($this->geocoderResult) === false) {
            $this->geocodeAddress();
        }
        return $this->geocoderResult->getLatitude(); 
    }
    public function getLongitude() {
        if (isset($this->geocoderResult) === false) { 
            $this->geocodeAddress();
        }
        return $this->geocoderResult->getLongitude(); 
    }
    protected function geocodeAddress() {
        $this->geocoderResult = $this->geocoder->geocode($this->address); 
        return true;
    } 
}
```

```php
<?php
class RetailStore {
    use Geocodable;
    // Class implementation goes here
    // Now each RetailStore instance can use the properties and methods provided by the Geocodable trait
}
```

```php
$geocoderAdapter = new \Geocoder\HttpAdapter\CurlHttpAdapter(); 
$geocoderProvider = new \Geocoder\Provider\GoogleMapsProvider($geocoderAdapter); 
$geocoder = new \Geocoder\Geocoder($geocoderProvider);
$store = new RetailStore();
$store->setAddress('420 9th Avenue, New York, NY 10001 USA'); 
$store->setGeocoder($geocoder);
$latitude = $store->getLatitude(); 
$longitude = $store->getLongitude(); 
echo $latitude, ':', $longitude;
```

PHP解释器将在编译时期将 Traits 的代码拷贝到类中。

从基类继承的成员被 traits 插入的成员所覆盖。优先顺序是来自当前类的成员覆盖了 trait 的方法，而 trait 则覆盖了被继承的方法。

如果两个 trait 都插入了一个同名的方法，如果没有明确解决冲突将会产生一个致命错误。

为了解决多个 trait 在同一个类中的命名冲突，需要使用 insteadof 操作符来明确指定使用冲突方法中的哪一个。

以上方式仅允许排除掉其它方法，as 操作符可以将其中一个冲突的方法以另一个名称来引入。

```
<?php
trait A {
    public function smallTalk() {
        echo 'a';
    }
    public function bigTalk() {
        echo 'A';
    }
}

trait B {
    public function smallTalk() {
        echo 'b';
    }
    public function bigTalk() {
        echo 'B';
    }
}

class Talker {
    use A, B {
        B::smallTalk insteadof A;
        A::bigTalk insteadof B;
    }
}

class Aliased_Talker {
    use A, B {
        B::smallTalk insteadof A;
        A::bigTalk insteadof B;
        B::bigTalk as talk;
    }
}
?>
```

使用 as 语法还可以用来调整方法的访问控制。

```php
<?php
trait HelloWorld {
    public function sayHello() {
        echo 'Hello World!';
    }
}

// 修改 sayHello 的访问控制
class MyClass1 {
    use HelloWorld { sayHello as protected; }
}

// 给方法一个改变了访问控制的别名
// 原版 sayHello 的访问控制则没有发生变化
class MyClass2 {
    use HelloWorld { sayHello as private myPrivateHello; }
}
?>
```
