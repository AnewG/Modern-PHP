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

5.4 新加入了 Traits，它既不是接口也不是类。与 Ruby 的 composable modules 或 mixins 类似。

它能被加入到一个或多个已经存在的类中。它声明了类能做什么（接口特性），同时也包含了具体实现（类特性）
