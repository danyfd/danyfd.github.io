---
title: php笔记
date: 2023-10-13 18:45:05
tags: "php"
categories: "编程语言"
---

### PHP符号

当解析一个文件时，PHP会寻找起始和结束标记，即\<?php和?>，这告诉PHP开始和停止解析二者之间的代码，此种解析方式使得PHP可以被嵌入到各种不同的文档中，任何起始和结束之外的部分都会被PHP解析器忽略；

若文件内容是纯PHP代码，最好在文件末尾删除PHP结束标记，避免在PHP结束标记之后意外加了空格或换行符导致PHP开始输出这些空白，而脚本中此时并无输出的意图

`->`是“插入式解引用操作符”，调用由==引用==传递参数的子程序的方法

`=>`用于数组中key指向value

`::`是范围解析操作符，又名域运算符，与C的`.`相似，php调用类的内部静态成员，或是类之间的调用就用`::`

`@`在@mysql_num_rows($res)中会忽略后面的表达式的错误

`{}`：

将多个语句合并为一个复合语句，如在if...else...中的使用；

变量间接引用中定界，避免歧义，如：${$my_var[8]}与${$my_var}[8]的区分

用于只是字符串变量中单个字符（下标从0开始），如：

```php
$my_str="1234";
$my_str{1}="5";		//现在$my_str内容为'1534'
```

#### 指针和引用的区别

`指针`：一个存储地址的变量，即指针是一个实体

`引用`：跟原来的变量实质上是同一个东西，是原变量的一个别名

```java
int a=1; int *p=&a;
int a=1; int &b=a;
```

`此用法为PHP5之后的特性，用于消除使用中括号引起的歧义`

- 可以有const指针，但是没有const引用
- 指针可以多级，引用只能一级
- 指针的值可以为空，但是引用的值不能为NULL
- 指针的值在初始化后可以改变，引用在初始化后就不会再改变了
- “sizeof引用“得到的是指向的变量的大小，而”sizeof指针“得到的是指针大小

### PDO

PHP数据对象（PHP Data Object）的缩写，用于统一各种数据库的访问接口

#### 创建PDO类的对象

***PDO::__construct ( string $dsn [, string $username [, string $password]] )***

- $dsn:表示数据源名称，包含了连接到数据库的信息，通常一个DSN由PDO驱动名、冒号以及具体PDO驱动的连接语法组成，例如：$dsn = "mysql : host=127.0.0.1; port=3306; dbname=db; charset=utf8"  

![](https://img-blog.csdnimg.cn/20200928230132532.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTkzNDUyMA==,size_16,color_FFFFFF,t_70)

#### PDO对象常用方法

##### PDO::exec()方法

- 表示执行一条SQL语句并返回受影响的行数
- 语法：int PDO::exec(string $sql)；$sql表示要被预处理和执行的SQL语句，<font color=red>不会从SELECT语句返回结果</font>
- 返回受修改或删除SQL语句影响的行数，若没有受影响的行，则返回0

![](https://img-blog.csdnimg.cn/20200930162053250.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTkzNDUyMA==,size_16,color_FFFFFF,t_70)

##### PDO::query()方法

- 表示执行一条SQL语句，返回一个结果集对象（**PDOStatement** ）
- 语法：public PDOStatement PDO::query ( string $statement )
- 主要用于SELECT、SHOW语句
- 执行成功返回PDOstatement对象，执行失败返回FALSE

![](https://img-blog.csdnimg.cn/20200930162138986.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTkzNDUyMA==,size_16,color_FFFFFF,t_70)

##### PDO::lastInsertId()方法

- 返回最后插入成功记录的id或序列值
- 语法：string PDO::lastInsertId ( void )

![](https://img-blog.csdnimg.cn/20200930162219957.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTkzNDUyMA==,size_16,color_FFFFFF,t_70)

##### PDO::setAttribute()方法

- 设置数据库句柄属性
- 语法：bool PDO::setAttribute ( int $attribute , mixed $value )
- PDO内置的一些可用的通用属性
  - PDO::ATTR_CASE：强制列名为指定的大小写
  - PDO::ATTR_ERRMODE：错误报告
  - PDO::ATTR_DEFAULT_FETCH_MODE： 设置默认的提取模式

![](https://img-blog.csdnimg.cn/20200930162258378.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTkzNDUyMA==,size_16,color_FFFFFF,t_70)

#### Statement对象常用方法

##### PDOStatement::fetch()方法

- 从结果集中获取一行，并向下移动指针
- 语法：mixed PDOStatement::fetch ([ int $fetch_style ] )
- 参数：$fetch_style控制下一行如何返回给调用者
  - PDO::FETCH_ASSOC，返回一个索引为结果集列名的数组
  - PDO::FETCH_BOTH(默认)，返回一个索引为结果集列名和以0开始的列号的数组
  - PDO::FETCH_NUM：返回一个索引为以0开始的结果集列号的数组

![](https://img-blog.csdnimg.cn/20200930162354370.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTkzNDUyMA==,size_16,color_FFFFFF,t_70)

##### PDOStatement::fetchAll()方法

- 返回一个包含结果集中所有行的数组
- 语法：array PDOStatement::fetchAll ([ int $fetch_style ] (同上))

![](https://img-blog.csdnimg.cn/20200930162432177.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTkzNDUyMA==,size_16,color_FFFFFF,t_70)

##### PDOStatement::rowCount()方法

- 返回受上一个SQL语句影响的行数
- 语法：int PDOStatement::rowCount ( void )
- 想要使用该函数，必须使用$pdo->query()返回PDOStatement对象

#### PDO错误处理

##### PDO支持三种错误模式

- 静默模式（Slient）：错误发生后不会主动报错，是默认的模式；此时不会讲错误显示在页面上，可以通过PDO的PDO::errorCode()和PDO::errorInfo()两个方法，来获取错误信息

  ![](https://img-blog.csdnimg.cn/20200930162617497.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTkzNDUyMA==,size_16,color_FFFFFF,t_70)

- 警告模式（Warning）：错误发生后通过PHP标准来报告错误；使用setAttribute()方法提前设置

  ![](https://img-blog.csdnimg.cn/20200930162807547.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTkzNDUyMA==,size_16,color_FFFFFF,t_70)

- 异常模式（Exception)：错误发生后抛出异常，需要捕捉和处理，需要setAttribute()方法提前设置

![](https://img-blog.csdnimg.cn/2020093016283844.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTkzNDUyMA==,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20200930162903378.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTkzNDUyMA==,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20200930162940298.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTkzNDUyMA==,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20200930162953217.png)

#### SQL语句预处理

##### SQL语句执行过程

SQL语句执行分为编译和执行两个阶段；第一次执行先编译再执行，耗用系统资源，相对不安全；若是第二次执行直接从缓存中读取，执行效率高且安全，有效避免SQL注入等安全问题

##### 预处理步骤

先提取并编译相同部分，将编译结果保存，再将不同数据部分替换，执行即可

![](https://img-blog.csdnimg.cn/20200930163212958.png)

![](https://img-blog.csdnimg.cn/20200930163243759.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTkzNDUyMA==,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20200930163411181.png)

`PDO的SQL语句预处理实例`

![](https://img-blog.csdnimg.cn/20200930163444103.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTkzNDUyMA==,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20200930163500100.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTkzNDUyMA==,size_16,color_FFFFFF,t_70)

#### 单例模式封装PDO

##### 单例模式

只会最多产生一个对象的设计思想，单例模式是为了保护资源的唯一性

- 单例模式为了解决如何只产生一个对象用到以下解决方案，简称`三私一公`
  - **私有化构造方法**：不让在外部产生多个对象
  - **私有化克隆方法**：不允许对象被克隆产生新的对象
  - **公有化静态方法**：运行进入类内部产生对象
  - **私有化静态属性**：保存已经产生的对象

`实例`

```php
<?php
class Singleton {
    //对象的产生是通过实例化产生的，而实例化是一种不可控行为，即可以产生无限多个对象，所以应该“禁止实例化”，即仅禁止在类外部实例化对象
    private function __construct(){ echo 'hello world';}
    //此时外部不能实例化对象，所以应在还未产生对象的时候进入“类内部”，通过“静态方法”让类直接进入到类内部
    public static function getInstance(){
        //此时依然没有对象，需要在内部进行对象实例化，且将对象返回给外部
        # 判断内部属性是否存在对象（is_object函数）：最好的判定是存的对象是当前类的instanceof
        if(!(self::$object instanceof self)){
            # 当前保存的内容不是当前类的对象
            self::$object = new self();
        }
        # 返回对象给外部
        return self::$object;
    }
    //上一步开启了实例化对象的窗口，但无限调用静态方法依然可以得到多个对象，若想只返回一个对象，就得保证类内部有办法存某个产生的对象，第一次产生新的，后面返回旧的，此时需要使用静态属性
    private static $object = NULL;	#初始化为NULL，没有对象
	//判断内部是否存在对象，根据$object修改getInstance()中返回的内容
    
    //此时可以保证外部无论多少次调用公有静态方法获取实例都会只得到一个对象，但此时外部对象依然可以产生新的对象：因为克隆，因此必须禁止对象的克隆，即在类内部私有化克隆方法
    private function __clone(){}
}

// 尝试外部实例化
// $demo = new Singleton();    //报错
// 外部获取对象
$s = Singleton::getInstance();
```

`使用单例模式PDO连接数据库`

```php
<?php
class MySQLPDO{
    //数据库默认连接信息
	private $dbConfig = array(
		'db'   => 'mysql', //数据库类型
		'host' => 'localhost', //服务器地址
		'port' => '3306', //端口
		'user' => 'root', //用户名
		'pass' => 'weicunbin123', //密码
		'charset' => 'utf8', //字符集
		'dbname' => 'testguest', //默认数据库
	);
	//单例模式 本类对象引用
	private static $instance;
    //PDO实例
	private $db;
    /**
	 * 私有构造方法
	 * @param $params array 数据库连接信息
	 */
    private function __construct($params){
        //初始化属性
        $this->dbConfig = array_merge($this->dbConfig,$params);
        //连接服务器
        $this->connect();
    }
    /**
	 * 获得单例对象
	 * @param $params array 数据库连接信息
	 * @return object 单例的对象
	 */
    public static function getInstance($params = array()){
        if(!self::$instance instanceof self){
            self::$instance = new self($params);
        }
        return self::$instance;	//返回对象
    }
    /**
	 * 私有克隆
	 */
    private function __clone() {}
    /**
	 * 连接目标服务器
	 */
    private function connect(){
        try{
            //连接信息
            $dsn = "{$this->dbConfig['db']}:host={$this->dbConfig['host']};port={$this->dbConfig['host']};dbname={$this->dbConfig['dbname']};charset={$this->dbConfig['charset']}";";
            //实例化PDO
            $this->db = new PDO($dsn,$this->dbConfig['user'],$this->dbConfig['pass'])
            //设定字符集
            $this->db->query("set names {$this->dbConfig['charset']}");
            echo 1;
        }catch (PDOException $e){
            //错误提示
            die("数据库操作失败：{$e->getMessage()}");
        }
    }
    /**
	 * 执行SQL
	 * @param $sql string 执行的SQL语句
	 * @return object PDOStatement
	 */
	 public function query($sql){
	 	$rst = $this->db->query($sql);
	 	if($rst===false){
	 		$error = $this->db->errorInfo();
	 		die("数据库操作失败：ERROR {$error[1]}({$error[0]}): {$error[2]}");
	 	}
	 	return $rst;
	 }
	 /**
	 * 取得一行结果
	 * @param $sql string 执行的SQL语句
	 * @return array 关联数组结果 
	 */
	 public function fetchRow($sql){
	 	return $this->query($sql)->fetch(PDO::FETCH_ASSOC);
	 }
	 /**
	 * 取得所有结果
	 * @param $sql string 执行的SQL语句
	 * @return array 关联数组结果 
	 */
	 public function fetchAll($sql){
	 	return $this->query($sql)->fetchAll(PDO::FETCH_ASSOC);
	 }
}
```

##### 对象的克隆

即对象的复制操作，分为浅克隆和深克隆

`浅克隆和深克隆的区别`

- 浅克隆：若源对象成员变量是值类型将复制一份给克隆对象，若是引用类型则将引用对象的地址复制给目标对象，即将源对象和目标对象成员变量指向相同的内存地址；只复制对象本身和包含的值类型，引用类型的成员对象并没有复制

  ![](https://img-blog.csdnimg.cn/bfa467e4c146470d9660e621b9aeba72.png)

  

- 深克隆：

  ![](https://img-blog.csdnimg.cn/11d8d2b4f89246a290d2b86d8e2d5642.png)

  

###### php中的浅克隆

只能克隆对象中的“非对象非资源”的数据

```php
class A{
    public $o = 1;
}
$obj = new A();
$obj1 = clone $obj;
$obj->o = 2;
var_dump($obj);
echo '<br />';
var_dump($obj1);
```

输出结果如下图：

![](https://img-blog.csdnimg.cn/20200330222553144.png)

```php
class B{
	public $b = 2;
}
class A{
	public $a = 1;
	public $c;
	function __construct(){
		$this->c = new B(); // 此时c中存储的是对象
	}
}
$obj = new A();
$obj1 = clone $obj;
$obj->a = 10;
$obj->c->b = 12;
var_dump($obj);
echo "<br />";
var_dump($obj1);
```

![](https://img-blog.csdnimg.cn/2020033022453997.png)

修改了obj中c的值发现obj1的c的值也跟着变了，这就是克隆不完全

###### php中的深克隆

一个对象所有属性都实现“复制”，在PHP中默认浅克隆，实现深克隆需对该类使用魔术方法__clone()，并在里面实现深克隆：

```php
class B{
	public $b = 2;
}
class A{
	public $a = 1;
	public $c;
	function __construct(){
		$this->c = new B(); // 此时c中存储的是对象
	}
	function __clone(){
		$this->c = clone $this->c; // 此方法会在对象克隆时调用
	}
}
$obj = new A();
$obj1 = clone $obj;
$obj->a = 10;
$obj->c->b = 12;
var_dump($obj);
echo "<br />";
var_dump($obj1);
```

![](https://img-blog.csdnimg.cn/20200330230013943.png)

### XML

可扩展标记语言，主要用来存储数据、作为配置文件、在网络中传输

#### XML语法

```
（1）xml文档的后缀为.xml
（2）xml第一行必须定义为文档声明
（3）xml文档中有且仅有一个根标签
（4）属性值必须使用单引号（单双都可）引起来且属性值是唯一的
（5）xml标签必须正确关闭，即要么是自闭和标签，要么是围堵标签
（6）xml标签名称区别大小写
```

#### XML的约束

##### DTD约束

```
一个完整的DTD声明主要由三个基本部分组成：元素声明、属性声明、实体声明
```

元素声明基本语法为：

```
<!ELEMENT 元素名 元素内容模型>
```

XML元素中可以有子元素，可以通过DTD定义某个元素中包含哪些子元素，若department标签下有dname和address标签，即：

```xml
<!ELEMENT department (dname,address)>
```

若hr标签下有多个employee标签，可以在（）后加个*代表有0个或多个employee标签：

```xml
<!ELEMENT hr (employee)*>
```

使用`,`分隔代表子元素必须按照这样的顺序出现，否则报错，若对于顺序无要求，可以使用`|`分隔子元素，如下：

```xml
<!ELEMENT employee (name,age,salary,department)>
<!ELEMENT employee (name|age|salary|department)>
```

##### XSD约束

XSD的作用：

```
（1）定义可出现在文档中的元素
（2）定义可出现在文档中的属性
（3）定义哪个元素是子元素
（4）定义子元素的次序
（5）定义子元素的数目
（6）定义元素是否为空，或者是否可包含文本
（7）定义元素和属性的数据类型
（8）定义元素和属性的默认值及固定值
```

定义简易元素的语法：

```
<element name="标签名" type="元素类型"/>
```

定义属性的语法：

```
<attribute name="属性名" type="属性的数据类型"/>
```

#### PHP XML Expat 解析器（基于事件的XML解析器）

```xml
<from>Jani</from>
```

基于事件的解析器将上面的XML报告为一连串的三个事件：

- 开始元素：from
- 开始CDATA部分值：Jani
- 关闭元素：from

工作原理：

1. 通过 xml_parser_create() 函数初始化XML解析器
2. 创建配合不同事件处理程序的函数
3. 添加xml_set_element_handler() 函数来定义，当解析器遇到开始和结束标签时执行哪个函数
4. 添加xml_set_character_data_handler() 函数来定义，当解析器遇到字符数据时执行哪个函数
5. 通过xml_parse()函数解析文件“test.html”
6. 添加xml_error_string() 函数将XML错误转换为文本说明
7. 调用xml_parser_free()函数释放分配给 xml_parser_create()函数的内存

#### PHP XML DOM 解析器（基于树的解析器）

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<from>Jani</from>
```

XML DOM将上面的XML视为一个树形结构：

- level 1：XML文档
- level 2：根元素\<from>
- level 3：文本元素Jani

`示例`

`note.xml`

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<note>
<to>Tove</to>
<from>Jani</from>
<heading>Reminder</heading>
<body>Don't forget me this weekend!</body>
</note>
```

`遍历XML`

初始化XML解析器，加载XML，遍历\<note>元素的所有元素

```php
<?php
$xmlDoc = new DOMDocument();
$xmlDoc->load("note.xml");

$x = $xmlDoc->documentElement;
foreach ($x->childNotes AS $item){
    print $item->nodeName . " = " . $item->nodeValue . "<br>";
}
```

```
#text =
to = Tove
#text =
from = Jani
#text =
heading = Reminder
#text =
body = Don't forget me this weekend!
#text =
```

在上面的实例中每个元素间存在空的文本节点，当XML生成时，通常会在节点间包含空白。XML DOM解析器将它们当做普通的元素

#### PHP SimpleXML

处理最普遍的XML任务，其余任务交由其它扩展处理；SimpleXML提供了一种获取XML元素的名称和文本的简单方式；与 DOM 或 Expat 相比，SimpleXML 更容易从 XML 元素中读取文本数据，它可以将XML文档转换为对象：

- 元素被转换为SimpleXMLElement 对象的单一属性，当同级别存在多个元素时，它们会被置于数组中
- 属性通过关联数组进行访问，索引对应属性名称
- 元素内部文本被转换为字符串，若一个元素拥有多个文本节点，则按照它们被找到的顺序进行排列

当执行类似以下基础任务时，SimpleXML用起来非常快捷：

- 读取XML文件的数据
- 编辑文本节点或属性

`示例`

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<note>
<to>Tove</to>
    <from>Jani</from>
    <heading>Reminder</heading>
    <body>Don't forget me this weekend!</body>
</note>
```

```php
<?php
$xml=simplexml_load_file("note.xml");
print_r($xml);
?>
```

```
SimpleXMLElement Object ( [to] => Tove [from] => Jani [heading] => Reminder [body] => Don't forget me this weekend! )
```

### AJAX

无需重新加载整个网页的情况下更新部分网页的技术，通过后台与服务器进行少量数据交换，使网页实现异步更新

![](http://run.jb51.net/uploadfile/2017/0524/20170524053007405.gif)

AJAX基于因特网标准，并使用以下技术组合：

- XMLHttpRequest 对象（与服务器异步交互数据）
- JavaScript/DOM（显示/取回信息）
- CSS（设置数据的样式）
- XML（常用作数据传输的格式）

`AJAX PHP实例`

![](https://s1.ax1x.com/2023/02/03/pSs3AVx.png)

当用户在上面的输入框中键入字符时会执行“showHint()”函数，该函数由“onkeyup”事件触发：

```html
<html>
    <head>
        <script>
        	function showHint(str){
                if (str.length == 0){
                    document.getElementById("txtHint").innerHTML="";
                    return;
                }
                if (window.XMLHttpRequest){
                    //IE7+, Firefox, Chrome, Opera, Safari 浏览器执行的代码
                    xmlhttp = new XMLHttpRequest();
                }else{
                    //IE6, IE5 浏览器执行的代码
                    xmlhttp = new ActiveXObject("Microsoft.XMLHTTP");
                }
                xmlhttp.onreadystatechange=function(){
                    if (xmlhttp.readyState==4 && xmlhttp.status==200){
                       document.getElementById("txtHint").innerHTML=xmlhttp.responseText;
                    }
                }
                xmlhttp.open("GET","gethint.php?q="+str,true);
                xmlhttp.send();
            }
        </script>
    </head>
</html>

<body>
    <p><b>在输入框中输入一个姓名：</b></p>
    <from>姓名：<input type="text" onkeyup="showHint(this.value)"></from>
    <p>返回值：<span id="txtHint"></span></p>
</body>
```

若输入框为空（str.length==0）,该函数会清空txtHint占位符的内容并退出，若非空，则showHint()会执行以下步骤：

- 创建XMLHttpRequest对象
- 创建在服务器响应就绪时执行的函数
- 向服务器上的文件发送请求
- 请注意添加URL末端的参数（q）（包含输入框的内容）

`gethint.php`

检查姓名数组，然后向浏览器（上述html）返回对应的姓名：

```php
<?php
// 将姓名填充到数组中
$a[]="Anna";
$a[]="Brittany";
$a[]="Cinderella";
$a[]="Diana";
$a[]="Eva";
$a[]="Fiona";
$a[]="Gunda";
$a[]="Hege";
$a[]="Inga";
$a[]="Johanna";
$a[]="Kitty";
$a[]="Linda";
$a[]="Nina";
$a[]="Ophelia";
$a[]="Petunia";
$a[]="Amanda";
$a[]="Raquel";
$a[]="Cindy";
$a[]="Doris";
$a[]="Eve";
$a[]="Evita";
$a[]="Sunniva";
$a[]="Tove";
$a[]="Unni";
$a[]="Violet";
$a[]="Liza";
$a[]="Elizabeth";
$a[]="Ellen";
$a[]="Wenche";
$a[]="Vicky";

//从请求URL地址中获取q参数
$q=$_GET["q"];
//查找是否有匹配值，若q>0
if (strlen($q) > 0) {
    $hint = "";
    for($i=0;$i<count($a);$i++){
        if (strtolower($q)==strtolower(substr($a[$i],0,strlen($q)))){
            if ($hint==""){
                $hint=$a[$i];
            }else{
                $hint=$hint.",".$a[$i];
            }
        }
    }
}

//若没有匹配值设置输出为“no suggestion”
if ($hint == ""){
    $response = "no suggestion";
}else{
    $response = $hint;
}
//输出返回值
echo $response;
```

若JS发送了任何文本，则：

1. 查找匹配JS发送的字符的姓名
2. 若未找到匹配，则将响应字符串设置为 "no suggestion"
3. 若找到一个或多个匹配姓名，则用所有姓名设置响应字符串
4. 把响应发送到“txtHint”占位符

#### 回调函数

通过函数指针调用的函数，若将函数的指针作为参数传递给另一个函数，当该指针被用来调用所指向的函数时称为回调函数（将一段可执行的代码像参数传递一样传给其它代码，且其会在某个时刻被调用执行就叫做回调；若立即被执行称为同步回调，如果在之后某个时间再执行则称为异步回调）

##### 回调函数与普通函数的区别

1. 将回调函数像参数一样传入库函数，改变传进库函数的参数可以实现不同的功能，且无需修改库函数的实现，实现了解耦
2. 主函数和回调函数在同一层，库函数在另外一层，如果库函数不可见，则只能传入不同的回调函数

### Session

在`php.ini`配置文件中修改三个配置：

- session.use_cookies

  把该值设置为1，利用cookie传递sessionid

- session.cookie_lifetime

  表示`SessionID`在客户端`Cookie`储存的时间（秒），默认为0，代表浏览器一关闭SessionID就作废，将它设置为自定义如：`86400`即`1天`

- session.gc_maxlifetime

  该值为Session在服务器端储存的时间，若超过这个时间，则Session数据自动删除，默认为1440即24分钟，也可以设置为86400即1天

- session.save_path

  session文件保存路径，linux默认`var/lib/php/session`，Windows需要配置并创建对应目录，如`D:/php/php7.4/tmp`

超过设置的时间后gc并不一定会工作，gc启动的概率为`session.gc_probaility / session.gc_divisor`，默认为`1/100`

#### 通过php修改失效时间

```php
<?php
$lifeTime = 24 * 3600;
session_set_cookie_params($lifeTime);
    
session_start();
$_SESSION["username"] = "peter";
echo "登记的用户名为：".$_SESSION["username"];
?>
```

#### 通过php修改gc_maxlifetime

```php
<?php
$LifeTime = 3600;
$DirectoryPath = "./tmp";
is_dir($DirectoryPath) or mkdir($DirectoryPath,0777);

//是否开启基于url传递sessionid,这里是不开启，发现开启也要关闭掉
if(ini_get("session.use_trans_sid") == true){
    ini_set("url_rewriter.tags","");
    ini_set("session.use_trans_sid",false);
}
ini_set("session.gc_maxlifetime",$Lifetime);	//设置session生存时间
ini_set("session.gc_divisor","1");
ini_set("session.gc_probability","1");
ini_set("session.cookie_lifetime", "0");//sessionID在cookie中的生存时间
ini_set("session.save_path", $DirectoryPath);//session文件存储的路径
session_start();
?>
```

若网站自定义了`session_save_path`，则需要给`session.gc_probability`设置值，否则的话，session产生的sessionID永远不会被删除