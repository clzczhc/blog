---
title: PHP笔记(一)
date: 2021-11-27 15:10:03
keywords: 后端 php
categories: "后端"
tags:
  - php
---

# PHP 笔记(一)

数据库项目作业要团队开发 web，选择了尝试后端，又向做出自己的网站这个目标踏出了一步。

## 1. 简单使用

```php
<!DOCTYPE html>
<html>
<body>

<h1>h1</h1>

<?php
echo "Hello World!";
?>

<h1>h1</h1>

</body>
</html>
```

在服务器下运行才有效果

![](https://pic.imgdb.cn/item/619683ef2ab3f51d9165ef8f.jpg)

## 2. 变量

```php
<?php
  $x = 1;
  $y = 1;
  $z = $x + $y;
  echo $z;
?>
```

### 2.1 变量作用域

PHP 有四种作用域

- local
- global
- static
- parameter

```php
<?php
$x = 5; // 全局变量
function fn()
{
  $y = 10;  // 局部变量
  echo "<p>函数内: </p>";
  // echo "变量x: $x"; // 在函数中访问一个全局变量，需要使用global关键字
  echo "<br>";
  echo "变量y: $y";
}

fn();
echo "<p>函数外: </p>";
echo "变量x: $x";
// echo "变量y: $y"; // 无法访问$y, $y变量在函数中定义，属于局部变量

?>
```

**global 关键字**：

```php
<?php
$x = 1;
$y = 1;
function test()
{
    global $x, $y;
    $y = $x + $y;	// 使用global关键字后，会改变全局变量的值
}

test();
echo "变量y: $y";
?>

// 或者下面的形式：PHP将所有全局变量存储到一个名为$GLOBALS[index]的数组中，index保存变量的名称，这个数组可以在函数内访问，也可以用来更新全局变量
<?php
$x = 1;
$y = 2;
function test()
{
    $GLOBALS['y'] = $GLOBALS['x'] + $GLOBALS['y'];
}

test();
echo "变量y: $y";
?>
```

**static 关键字**：

当一个函数完成时，它的所有变量通常都会被删除，在第一次声明变量时使用**static 关键字**可以实现让特定变量不被删除。然后，每次调用该函数时，该变量都会保留函数前一次被调用时的值。**该变量还是函数的局部变量**

```php
<?php
function test() {
  static $x = 0;
  echo $x;
  $x++;
  echo "<br>";
}
test();
test();
test();
?>
```

## 3. 定界符 EOF

```php

<?php
$name = "clz";
echo <<<EOF
        <h1>测试</h1>  // html格式会被解析
        "abg\n"$name  // 双引号也会被解析，双引号内会保留转义符的转义效果
        "123"
EOF;
echo "Hello World!";
?>
```

## 4. 数据类型

```php
<?php
$x = 123;
var_dump($x); // 返回变量的数据类型和值
$x = -234;
var_dump($x);

$x = 017;     // 八进制
var_dump($x);

$x = 0xaa;    // 十六进制
var_dump($x);

$x = "test";
var_dump($x);

$x = 1.234;   // 浮点型
var_dump($x);
$x = 2.4e3;
var_dump($x);

$x = true;    // 布尔型
var_dump($x);
?>
```

```php
<?php
$arr = array(1, "Hello", true);   // 数组
var_dump($arr);
var_dump($arr[0]);

// 类
class Car
{
  var $color;
  function __construct($color="green") {  // 构造函数
    $this->color = $color;		// $this指向对象, 所以可以通过$this->color = $color设置颜色值
  }
  function what_color() {
    return $this->color;
  }
}

$c = new Car("white");
var_dump($c);

$d = null;
var_dump($d);
?>
```

## 5. 类型比较

和 JavaScript 一样，有松散比较和严格比较两种形式

- 松散比较(等于)："==", 只比较值，不比较类型
- 严格比较(绝对等于)："===", 既比较值，也比较类型

## 6. 常量

设置常量需要使用 define()函数

> ```php
> bool define ( string $name , mixed $value [, bool $case_insensitive = false ] )
> ```

- name: 必选参数，常量名称
- value: 必选参数，常量的值
- case_insensitive: 可选参数，如果是 true，则常量对大小写不敏感。默认是大小写敏感。

```php
<?php
  define("HW", "Hello World!");
  echo HW;
  echo "<br>";

  function test() {
    echo HW;  // 常量默认是全局变量
  }

  test();
?>
```

## 7. 字符串

### 7.1 并置运算符(.)

用于把两个字符串连接起来

```php
<?php
  $h = "Hello ";
  $w = "World";
  echo $h . $w;
?>
```

### 7.2 strlen()函数

strlen()函数返回字符串的长度(字节数), `echo strlen("中文");`会输出 6，因为一个中文占 3 个字节

```php
<?php
  $h = "Hello ";
  $w = "World";
  echo strlen($h . $w);
?>
```

### 7.3 strpos()函数

strpos()函数用于在字符串中查找字符串，如果找到匹配，则返回第一个匹配的字符位置，如果找不到，则返回 false

```php
<?php
  echo strpos("Hello World!", "!");
  echo "<br>";

  echo strpos("Hello World!", "World");
  echo "<br>";

  var_dump(strpos("Hello World!", "Worll"));
?>
```

## 8. 关联数组

```php
<?php
  $age = array("A" => 17, "B" => 33, "C" => 21);
  /*
   * 创建关联数组的另一种方法:
   * $age["A"] = 17;
   * $age["B"] = 33;
   * $age["C"] = 21;
   */
  echo "A is " . $age['A'] . " years old.";
  echo "<br>";

  // 遍历关联数组
  foreach($age as $k => $v)
  {
    echo "Key = " . $k . ", Value = " .$v;
    echo "<br>";
  }
?>
```

## 9. 支持可变参数的函数

```php
<?php
    echo "<pre>";
    function test(...$args)
    {
      $num = count($args);
      echo "函数调用参数个数: " . $num . PHP_EOL;
      /* PHP_EOL是文本换行，直接使用的话，可能会是空格，需要在前面加echo "<pre>";，
       * 做文本格式化处理，才能当成换行使用
       */
      echo "函数参数详情: ";

      foreach($args as $args)
      {
        echo $args . " ";
      }
      echo PHP_EOL;

    }
    test("a");
    test("a", "b");
    test("a", "b", "c");
  ?>
```

## 10. 数据库

### 10.1 连接数据库

- MySQLi 面向对象

```php
<?php
$servername = "localhost";
$username = "root";
$password = "密码";

$conn = new mysqli($servername, $username, $password);

if($conn->connect_error) {
  die("连接失败: " . $conn->connect_error);	// die() 函数输出一条消息，并退出当前脚本。
}

echo "连接成功";

$conn->close();	  // 关闭连接
?>
```

- MySQLi 面向过程

```php
<?php
$servername = "localhost";
$username = "root";
$password = "root1234";

$conn = mysqli_connect($servername, $username, $password);

if(!$conn) {
  die("连接失败: " . mysqli_connect_error());
}
echo "连接成功";

mysqli_close($conn);	// 关闭连接
?>
```

- PDO

```php
<?php
$servername = "localhost";
$username = "root";
$password = "root1234";

try
{
  $conn = new PDO("mysql:host=$servername;", $username, $password);
  echo "连接成功";
}
catch(PDOException $e)
{
  echo $e->getMessage();
}

$conn = null;  // 关闭连接
?>
```

### 10.2 创建数据库

- MySQLi 面向对象

```php
<?php
  $servername = "localhost";
  $username = "root";
  $password = "root1234";

  $conn = new mysqli($servername, $username, $password);
  if($conn->connect_error) {
    die("连接失败: " . $conn->connect_error);
  }

  $sql = "create database php_db";
  if($conn->query($sql) === true) {   // 通过mysqli对象的query方法实现使用mysql语句，成功则返回true
    echo "数据库创建成功";
  } else {
    echo "数据库创建失败: " . $conn->error;
  }

  $conn->close();
?>
```

![](https://pic.imgdb.cn/item/6197c29c2ab3f51d910fcd75.jpg)

- MySQLi 面向过程

```php
<?php
  $servername = "localhost";
  $username = "root";
  $password = "root1234";

  $conn = mysqli_connect($servername, $username, $password);
  if(!$conn) {
    die("连接失败: " . mysqli_connect_error());
  }

  $sql = "create database php_db_1";
  if(mysqli_query($conn, $sql)) {
    echo "数据库创建成功";
  } else {
    echo "数据库创建失败: " . mysqli_error($conn);
  }

  mysqli_close($conn);
?>
```

- PDO

```php
<?php
  $servername = "localhost";
  $username = "root";
  $password = "root1234";

  try {
    $conn = new PDO("mysql:host=$servername", $username, $password);

     // 设置 PDO 错误模式为异常
     $conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
     $sql = "create database php_db_2";

     $conn->exec($sql);
     echo "数据库创建成功";
  }
  catch(PDOException $e)
  {
    echo $sql . " " . $e->getMessage();
  }

  $conn = null;
?>
```

### 10.3 创建表

MySQLi 面向对象

```php
<?php
  $servername = "localhost";
  $username = "root";
  $password = "root1234";
  $dbname = "php_db";

  $conn = new mysqli($servername, $username, $password, $dbname);

  if($conn->connect_error) {
    die("连接失败: " . $conn->connect_error);
  }

  $sql = "create table guests_1(
    id int(6) unsigned auto_increment primary key,
    firname varchar(30) not null,
    lastname varchar(30) not null,
    email varchar(50),
    reg_date timestamp
  )";

  if($conn->query($sql) === true) {
    echo "创建表成功";
  } else {
    echo "创建表失败: " . $conn->error;
  }

  $conn->close();
?>
```

其他操作，基本类似，具体实现可查[PHP 教程 | 菜鸟教程 (runoob.com)](https://www.runoob.com/php/php-tutorial.html)

参考：[PHP 教程 | 菜鸟教程 (runoob.com)](https://www.runoob.com/php/php-tutorial.html)
