---
title: PHP笔记(二)
date: 2021-11-27 15:10:19
keywords: 后端 php
categories: "后端"
tags:
  - php
---

# PHP 笔记(二)

## 1. 面向对象

<b style="color: red">已经学过 C++面向对象、Java 面向对象，这一部分属于是复习，以及熟悉 PHP 面向对象的不同之处，所以不会详讲</b>

### 1.1 基本使用

```php
<?php
  class Car {
    function __construct($color) { // 构造函数: 用来在创建对象时初始化对象，在创建对象的语句中与 new 运算符一起使用。
      $this->color = $color;
    }

    function __destruct() {  // 析构函数: 当对象结束它的生命周期时，系统会自动执行析构函数
      echo "<br>";
      echo "系统自动执行析构函数";
    }

    function setColor($color) {
      $this->color = $color;
    }

    function getColor() {
      return $this->color;
    }
  }

  $car = new Car("White");
  echo $car->getColor();
  echo "<br>";

  $car->setColor("Red");
  echo $car->getColor();

?>
```

### 1.2 继承

```php
<?php
  class Car {
    function getColor() {
      return "白色";
    }
  }

  class FCar extends Car {
    function getColor() { // 重写getColor()方法
      echo "红色";
    }

    function fly() {
      echo "I can Fly";
    }
  }

  $fcar = new FCar("White");
  echo $fcar->getColor();
  echo "<br>";

  $fcar->fly();

?>
```

### 1.3 访问控制

只有属性的访问控制，方法的类似

```php
<?php
  echo "<pre>";
  class MyClass1 {
    public $public = "Public";
    protected $protected = "Protected";
    private $private = "Private";

    function printM() {
      echo $this->public . "\n";
      echo $this->protected . "\n";
      echo $this->private . "\n";
    }
  }

  class MyClass2 extends MyClass1 {
    protected $protected = 'Protected2';  // 可以对public和protected重定义，但private不能，因为无法继承private属性
    function printM() {
      echo $this->public . "\n";
      echo $this->protected . "\n";
      echo $this->private . "\n";   // 会出现未定义private的警告
    }
  }

  $obj1 = new MyClass1();
  echo $obj1->public . "\n";
  // echo $obj1->protected;
  // echo $obj1->private;   // 类外部无法访问protected属性和private属性
  $obj1->printM();      // 对象方法可以访问所有属性

  $obj2 = new MyClass2();
  echo $obj2->public . "\n";
  $obj2->printM();
?>
```

### 1.4 接口

使用接口，可以指定某个类必须实现哪些方法，但不需要定义这些方法的具体内容

接口通过**interface**关键字来定义，定义所有的方法都是空的

<b style="color: red">接口中定义的所有方法都必须是公有(public)</b>, 这是接口的特性

实现接口，使用**implements**操作符。类中必须实现接口中定义的所有方法，否则会报错。

```php
<?php
  echo "<pre>";
  interface myName {
    public function setName($name);
    public function getName();
  }

  class Name implements myName {
    public function setName($name) {
      $this->name = $name;
    }
    public function getName() {
      return $this->name;
    }
  }

  $n = new Name;
  $n->setName("CLZ");
  echo $n->getName();
?>
```

### 1.5 抽象类

任何一个类，如果至少有一个方法被声明为抽象的，则这个类就必须声明为抽象的

被定义为抽象的方法只是声明了它的调用方式，不能定义具体的功能实现。

继承一个抽象类的时候，子类必须定义父类中的抽象方法，这些方法的访问控制必须和父类一样或比父类宽松。

```php
<?php
  echo "<pre>";
  abstract class AbstractClass {
    abstract protected function getValue();
    abstract protected function prefixValue($prefix);

    public function printOut() {
      print $this->getValue() . PHP_EOL;
    }
  }

  class ConcreateClass1 extends AbstractClass {
    protected function getValue() {
      return "ConcreateClass1";
    }

    public function prefixValue($prefix) {
      return "{$prefix}ConcreateClass1";
    }
  }

  class ConcreateClass2 extends AbstractClass {
    public function getValue() {
      return "ConcreateClass2";
    }

    public function prefixValue($prefix) {
      return "{$prefix}ConcreateClass2";
    }
  }

  $class1 = new ConcreateClass1();
  $class1->printOut();
  echo $class1->prefixValue("FOO_") . PHP_EOL;

  $class2 = new ConcreateClass2();
  $class2->printOut();
  echo $class2->prefixValue("FOO_") . PHP_EOL;
?>
```

### 1.6 静态方法

通过**static 关键字**声明类属性或方法，可以不是实例化类直接访问

**静态属性不能由对象通过->操作符访问，而应该使用::操作符访问**

```php
<?php
  echo "<pre>";
  class Test {
    public static $my_static = "静态变量";

    public function staticValue() {
      return self::$my_static;
    }

    static function sayHello() {
      echo "Hello!" . PHP_EOL;
    }
  }

  print Test::$my_static . PHP_EOL;
  Test::sayHello();       // 可以直接通过类名::访问静态变量和静态方法

  $test = new Test();
  print $test::$my_static . PHP_EOL;    // 也可以通过对象::访问静态变量和静态方法
  print $test->staticValue() . PHP_EOL;
?>
```

### 1.7 调用父类的构造方法

在子类的构造方法中调用**parent::\_\_construct()**

```php
<?php
  echo "<pre>";
  class Car {
    function __construct($color, $cost) {
      $this->color = $color;
      $this->cost = $cost;
    }
  }

  class WCar extends Car{
    function __construct($color, $cost, $weight) {
      parent::__construct($color, $cost);
      $this->weight = $weight;
    }
  }

  $wcar = new WCar("White", 99999, "999kg");
  var_dump($wcar);
?>
```

## 2. 表单

**简单使用**

form.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>

  <body>
    <form action="welcome.php" method="post">
      名字: <input type="text" name="fname" /> 年龄:
      <input type="text" name="age" />
      <input type="submit" value="提交" />
    </form>
  </body>
</html>
```

welcome.php

```php
<?php
  echo "欢迎" . $_POST["fname"] . "<br>";
  echo "你的年龄是 " . $_POST["age"] . "岁";
?>
```

## 3. PHP AJAX

### 3.1 AJAX

AJAX 是一种**无需重新加载整个页面的情况下，能够更新部分网页的技术**。

AJAX 通过在后台与服务器进行少量数据交换，使网页实现异步更新。使用 AJAX 可以实现在不重载整个页面的情况下，对页面的某些部分进行更新。

![](https://www.runoob.com/wp-content/uploads/2013/08/ajax.gif)

### 3.2 使用 PHP、AJAX 实现简单的前后端交互

websites 表如下:

![](https://pic.imgdb.cn/item/619b1c562ab3f51d916a0508.jpg)

前端：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>

  <body>
    <form>
      <select name="users" onchange="showSite(this.value)">
        <option value="">选择一个网站:</option>
        <option value="1">Google</option>
        <option value="2">淘宝</option>
        <option value="3">Facebook</option>
      </select>
    </form>

    <div id="txtHint"></div>
    <script>
      const txtHint = document.getElementById("txtHint");

      function showSite(str) {
        if (str == "") {
          txtHint.innerHTML = "";
          return;
        }

        let xmlhttp = new XMLHttpRequest();
        xmlhttp.open("GET", "getsite.php?v=" + str, true); // true表示异步执行请求
        xmlhttp.send(null);

        xmlhttp.onreadystatechange = function () {
          if (xmlhttp.readyState === 4 && xmlhttp.status === 200) {
            txtHint.innerHTML = xmlhttp.responseText;
          }
        };
      }
    </script>
  </body>
</html>
```

后端

```php
<?php
  // $v = isset($_GET['v']) ? intval($_GET['v']) : '';
  $v = $_GET['v'];

  if(empty($v)) {
    echo '请选择一个网站';
    exit;
  }

  $con = mysqli_connect('localhost', 'root', 'root1234');
  if(!$con) {
    die('连接数据库失败: ' . mysqli_error($con));
  }

  mysqli_select_db($con, 'php_ajax_mysql'); // 选择数据库
  mysqli_set_charset($con, 'utf8');

  $sql = 'select * from websites where id = ' . $v;

  $result = mysqli_query($con, $sql);

  echo '<table border="1" cellspacing="0" cellpadding="20" style="margin-top: 20px">
        <tr style="background-color: #555; color: #fff">
          <th>ID</th>
          <th>网站名</th>
          <th>网站url</th>
          <th>Alexa排名</th>
          <th>国家</th>
        </tr>';
  while($row = mysqli_fetch_assoc($result)) {
    echo '<tr>';
    echo '<td>' . $row['id'] . '</td>';
    echo '<td>' . $row['name'] . '</td>';
    echo '<td>' . $row['url'] . '</td>';
    echo '<td>' . $row['alexa'] . '</td>';
    echo '<td>' . $row['country'] . '</td>';
    echo '</tr>';
  }
  echo '</table>';

  mysqli_close($con);

?>
```

结果：

![](https://pic.imgdb.cn/item/619b1d232ab3f51d916a66d1.jpg)

## 4. 二维数组

```php
<?php
  echo "<pre>";
  $sites = array
  (
    "google" => array
    (
      "Google搜索",
      "http://www.google.com"
    ),
    "baidu" => array
    (
      "Baidu搜索",
      "http://www.baidu.com"
    )
    );
    echo $sites['google'][0] . '地址为: ' . $sites['google'][1];
    echo "</pre>";
?>
```

## 5. 日期

date()用于格式化时间/日期

语法：

> ```
> string date ( string $format [, int $timestamp ] )
> ```

- format：**必需**。规定如何格式化当前的日期和时间
- timestamp：可选。规定时间戳，默认是当前的时间和日期。

```php
<?php
  echo "<pre>";
  echo date("Y/m/d") . PHP_EOL;
  echo date("Y-m-d") . PHP_EOL;
  echo date("Y-m-d", mktime(23, 59, 59, 12, 31, 2020));
  echo "</pre>";
?>
```

mktime()返回一个日期的 UNIX 时间戳

语法：

> mktime(hour, minute, second, month, day, year);

## 6. 插入文件

require 和 include 除了处理错误的方式不同外，在其他方面都相同

- require 生成一个致命错误，在错误发生后脚本会停止执行
- include 生成一个警告，在错误发生后脚本会继续执行

* require 一般放在 PHP 文件的最前面，程序在执行前就会先导入要引用的文件；
* include 一般放在程序的流程控制中，当程序执行时碰到才会引用，简化程序的执行流程

**上一段文字引自**[PHP include 和 require | 菜鸟教程 (runoob.com)](https://www.runoob.com/php/php-includes.html)

```php
<?php
  echo "<h1>测试开始<h1>";
  require "./form.html";
  // include "./form.html";
  echo "<h1>测试结束</h1>"
?>
```

![](https://pic.imgdb.cn/item/6199b1ab2ab3f51d91e38ce8.jpg)

另一种用法：

test.php：

```php
<?php
  $tt = "Hello World!";
?>
```

practise.php

```php
<?php
  require './test.php';
  echo $tt;
?>
```

![](https://pic.imgdb.cn/item/6199b4dd2ab3f51d91e4bc20.jpg)

## 7. 文件上传

form.html

```html
<html>
  <head>
    <meta charset="utf-8" />
    <title>菜鸟教程(runoob.com)</title>
  </head>

  <body>
    <form action="upload_file.php" method="post" enctype="multipart/form-data">
      <label for="file">文件名：</label>
      <input type="file" name="file" id="file" /><br />
      <input type="submit" name="submit" value="提交" />
    </form>
  </body>
</html>
```

upload_file.php`

```php
<?php
  if($_FILES['file']['error'] > 0) {
    echo '错误: ' . $_FILES['file']['error'] . '<br>';
  } else {
    echo '上传文件名: ' . $_FILES['file']['name'] . '<br>';
    echo '文件类型: ' . $_FILES['file']['type'] . '<br>';
    echo '文件大小: ' . ($_FILES['file']['size'] / 1024) . ' kB<br>';
    echo '文件临时存储的位置: ' . $_FILES['file']['tmp_name'];
  }
?>
```

![](https://pic.imgdb.cn/item/6199c8d62ab3f51d91ecb868.jpg)

**上传限制和保存上传的文件**:

```php
<?php
  $allowedExts = array('gif', 'jpeg', 'jpg', 'png');
  $temp = explode('.', $_FILES['file']['name']);    // 文件名按 . 分为字符串数组
  $extension = end($temp);                  // 得到数组中最后一个元素的值，即文件后缀名
  if($_FILES['file']['size'] < 204800       // 上传文件大小要求小于200kb
  && in_array($extension, $allowedExts)) {  // 上传文件类型必须符合要求
    if($_FILES['file']['error'] > 0) {
      echo '错误: ' . $_FILES['file']['error'] . '<br>';
    } else {
      echo '上传文件名: ' . $_FILES['file']['name'] . '<br>';
      echo '文件类型: ' . $_FILES['file']['type'] . '<br>';
      echo '文件大小: ' . ($_FILES['file']['size'] / 1024) . ' kB<br>';
      echo '文件临时存储的位置: ' . $_FILES['file']['tmp_name'] . '<br>';

      if(file_exists('upload/' . $_FILES['file']['name'])) {   // 判断文件已经存在
        echo $_FILES['file']['name'] . '文件已经存在';
      } else {
        move_uploaded_file($_FILES['file']['tmp_name'], 'upload/' . $_FILES['file']['name']);   // 必须有upload文件夹，否则会出错
        echo '文件存储在: ' . 'upload/' . $_FILES['file']['name'];
      }
    }
  } else {
    echo "非法的文件格式";
  }
?>
```

## 8. Cookie

当 web 服务器向浏览器发送 web 页面时，在连接关闭后，服务端不会记录用户的信息

Cookie 的作用就是用于解决**如何记录客户端的用户信息**

- 当用户访问 web 页面时，他的名字可以记录在 Cookie 中
- 在用户下一次访问该页面时，可以在 Cookie 中读取用户访问记录

Cookie 以键值对的形式存储。

### 8.1 创建 Cookie

语法：

```php
setcookie(name, value, expire, path, domain);
```

- name: 必需。规定 cookie 的名称
- value: 必需。规定 cookie 的值
- expire: 可选。规定 cookie 的有效值
- path: 可选。规定 cookie 的服务器路径
- domain: 可选。规定 cookie 的域名
- secure: 可选。规定是否通过安全的 HTTPS 连接来传输 cookie

```php
<?php
  $expire = time() + 60*60*24*30; // 设置过期时间为1个月, 60秒*60分*24小时*30天
  setcookie('user', 'clz', $expire);
?>
```

### 8.2 取回 Cookie 的值

通过$\_COOKIE 变量取回 Cookie 的值

```php
<?php
 echo $_COOKIE['user'] . "<br>";
 print_r($_COOKIE);   // 查看所有的cookie
?>
```

简单判断是不是登录的用户

```php
<?php
 if(isset($_COOKIE['user'])) {  //判断cookie是否存在
  echo "欢迎 " . $_COOKIE['user'] . '<br>';
 } else {
   echo '普通访客<br>';
 }
?>
```

### 8.3 删除 Cookie

有点巧妙，重新设置 cookie，不过把过期时间设置为过去的时间

```php
<?php
  setcookie('user', '', time() - 60);
?>
```

## 9. PHP JSON

**json_encode()函数**用于对变量进行 JSON 编码，执行成功则返回 JSON 数据，否则返回 false

```php
<?php
  $arr = array('a' => 1, 'b' => 2, 'c' => 3);
  echo json_encode($arr);
?>
```

> 结果：{"a":1,"b":2,"c":3}

```php
<?php
   $arr = array('test' => '测试', 'baidu' => '百度');
   echo json_encode($arr);  // 编码中文
   echo "<br>";
   echo json_encode($arr, JSON_UNESCAPED_UNICODE); // 不编码中文
?>
```

结果：

> {"test":"\u6d4b\u8bd5","baidu":"\u767e\u5ea6"}
> {"test":"测试","baidu":"百度"}

**json_decode() 函数**用于对 JSON 格式的字符串进行解码，并转换为 PHP 变量。

```php
<?php
  $json = '{"a": 1, "b": 2, "c": 3}';

  print_r(json_decode($json));
  print '<br>';
  print_r(json_decode($json, true));  // 第二个参数为true时，将返回数组，为false时，返回对象，默认为false
?>
```

参考：[PHP 教程 | 菜鸟教程 (runoob.com)](https://www.runoob.com/php/php-tutorial.html)
