## PHP

### 简介

[PHP手册](https://secure.php.net/manual/zh/)

- PHP（全称：PHP：Hypertext Preprocessor，即"PHP：超文本预处理器"）是一种通用开源脚本语言。
- PHP 脚本在服务器上执行。
- PHP 可免费下载使用。
- PHP 文件可包含文本、HTML、JavaScript代码和 PHP 代码
- ·PHP 代码在服务器上执行，结果以纯 HTML 形式返回给浏览器
- PHP 文件的默认文件扩展名是 ".php"



### php安装

推荐使用：[wampserver](http://www.wampserver.com/)，备选[phpstudy](http://phpstudy.php.cn/)



### 基础

#### echo与print

`echo`、`print`：输出信息（echo 输出的速度比 print 快， echo 没有返回值，print有返回值1。）

- echo - 可以输出一个或多个字符串
- print - 只允许输出一个字符串，返回值总为 1

```php
<?php
echo "hello " , "world";
echo "<br>";
print "hi world";
?>
```

`$`：创建变量（变量名对大小写敏感、变量名必须以字母或下划线开头、变量名只能包含字母数字以及下划线）

```php
<?php
$a = 1;
$B = 2;
$c = $a + $B;
echo $c;
?>
```



#### 作用域

**变量作用域**：所有在函数外部定义的变量，拥有全局作用域。在函数内部想要访问外部的变量需要使用`global`关键字。

```php
<?php
$x = 5;
function func(){
  global $x;
  echo $x;
}
func();
?>
```

**static作用域**：当一个函数完成时，它的所有变量通常都会被删除。如果需要保留，则需要使用`static`关键字

```php
<?php
  function myTest()
{
    static $x = 1;
    echo $x;
    $x++;
}

myTest();
myTest();
myTest();
myTest();
?>
```



#### EOF

PHP EOF(heredoc)是一种在命令行shell（如sh、csh、ksh、bash、PowerShell和zsh）和程序语言（像Perl、PHP、Python和Ruby）里定义一个字串的方法。

- 1. 必须后接分号，否则编译通不过。
- 2. **EOF** 可以用任意其它字符代替，只需保证结束标识与开始标识一致。
- 3. **结束标识必须顶格独自占一行(即必须从行首开始，前后不能衔接任何空白和字符)。**
- 4. 开始标识可以不带引号或带单双引号，不带引号与带双引号效果一致，解释内嵌的变量和转义符号，带单引号则不解释内嵌的变量和转义符号。

```php
<?php
$name="qty";
$a= <<<EOF
    "abc"$name
    "123"
EOF;
?>
```



#### var_dump()

`var_dump()` 函数打印**变量**的数据类型和值。

```php
<?php
$b = 3.1;
$c = true;
var_dump($b, $c);
?>
```



#### 数据类型

##### 字符串（String）

位于单引号或双引号里面的文本

##### 整型（Integer）

整数是一个没有小数的数字。

整数规则:

- 整数必须至少有一个数字 (0-9)
- 整数不能包含逗号或空格
- 整数是没有小数点的
- 整数可以是正数或负数
- 整型可以用三种格式来指定：十进制， 十六进制（ 以 0x 为前缀）或八进制（前缀为 0）。

```php
<?php 
$x = 5985;
var_dump($x);
echo "<br>"; 
$x = -345; // 负数 
var_dump($x);
echo "<br>"; 
$x = 0x8C; // 十六进制数
var_dump($x);
echo "<br>";
$x = 047; // 八进制数
var_dump($x);
?>
```

##### 浮点型（Float）

浮点数是带小数部分的数字，或是指数形式。

```php
<?php 
$x = 10.365;
var_dump($x);
echo "<br>"; 
$x = 2.4e3;
var_dump($x);
echo "<br>"; 
$x = 8E-5;
var_dump($x);
?>
```

##### 布尔型（Boolean）

布尔值只有两个值：`ture` 、`false`

##### 数组（Array）

数组可以在一个变量中存储多个值。

```php
<?php
  $arr = array("one","two","three");
?>
```

##### 对象（Object）

先使用`class`关键字声明类对象再通过`new`语句实例化

```php
<?php
class foo
{
    function do_foo()
    {
        echo "Doing foo."; 
    }
}

$bar = new foo;
$bar->do_foo();
?>
```



##### 空值（NULL）

`NULL` 值表示变量没有值。`NULL` 是数据类型为 `NULL` 的值，`NULL` 值指明一个变量是否为空值。 同样可用于数据空值和`NULL`值的区别，**可以通过设置变量值为 `NULL` 来清空变量数据**。



#### 常量

常量是一个简单值的标识符。该值在脚本中不能改变，一个常量由英文字母、下划线、和数字组成,但数字不能作为首字母出现。 (常量名不需要加 $ 修饰符)， **常量在整个脚本中都可以使用**。

```php
<?php

// 合法的常量名
define("FOO",     "something");
define("FOO2",    "something else");
define("FOO_BAR", "something more");

// 非法的常量名
define("2FOO",    "something");

// 下面的定义是合法的，但应该避免这样做：(自定义常量不要以__开头)
// 也许将来有一天PHP会定义一个__FOO__的魔术常量
// 这样就会与你的代码相冲突
define("__FOO__", "something");

?>
```



#### 运算符

##### 算术运算符

| 运算符 | 名称             | 描述            | 实例                | 结果  |
| ------ | ---------------- | --------------- | ------------------- | ----- |
| x + y  | 加               | x 和 y 的和     | 2 + 2               | 4     |
| x - y  | 减               | x 和 y 的差     | 5 - 2               | 3     |
| x * y  | 乘               | x 和 y 的积     | 5 * 2               | 10    |
| x / y  | 除               | x 和 y 的商     | 15 / 5              | 3     |
| x % y  | 模（除法的余数） | x 除以 y 的余数 | 5 % 2 10 % 8 10 % 2 | 1 2 0 |
| - x    | 取反             | x 取反          | -2                  | -2    |
| a . b  | 并置             | 连接两个字符串  | "Hi" . "Ha"         | HiHa  |



##### 赋值运算符

| 运算符 | 等同于    | 描述                           |
| ------ | --------- | ------------------------------ |
| x = y  | x = y     | 左操作数被设置为右侧表达式的值 |
| x += y | x = x + y | 加                             |
| x -= y | x = x - y | 减                             |
| x *= y | x = x * y | 乘                             |
| x /= y | x = x / y | 除                             |
| x %= y | x = x % y | 模（除法的余数）               |
| a .= b | a = a . b | 连接两个字符串                 |



##### 递增递减运算符

| 运算符 | 名称   | 描述                |
| ------ | ------ | ------------------- |
| ++ x   | 预递增 | x 加 1，然后返回 x  |
| x ++   | 后递增 | 返回 x，然后 x 加 1 |
| -- x   | 预递减 | x 减 1，然后返回 x  |
| x --   | 后递减 | 返回 x，然后 x 减 1 |



##### 比较运算符

| 运算符  | 名称       | 描述                                           | 实例               |
| ------- | ---------- | ---------------------------------------------- | ------------------ |
| x == y  | 等于       | 如果 x 等于 y，则返回 true                     | 5==8 返回 false    |
| x === y | 绝对等于   | 如果 x 等于 y，且它们类型相同，则返回 true     | 5==="5" 返回 false |
| x != y  | 不等于     | 如果 x 不等于 y，则返回 true                   | 5!=8 返回 true     |
| x <> y  | 不等于     | 如果 x 不等于 y，则返回 true                   | 5<>8 返回 true     |
| x !== y | 绝对不等于 | 如果 x 不等于 y，或它们类型不相同，则返回 true | 5!=="5" 返回 true  |
| x > y   | 大于       | 如果 x 大于 y，则返回 true                     | 5>8 返回 false     |
| x < y   | 小于       | 如果 x 小于 y，则返回 true                     | 5<8 返回 true      |
| x >= y  | 大于等于   | 如果 x 大于或者等于 y，则返回 true             | 5>=8 返回 false    |
| x <= y  | 小于等于   | 如果 x 小于或者等于 y，则返回 true             | 5<=8 返回 true     |



##### 逻辑运算符

| 运算符   | 名称 | 描述                                         | 实例                                  |
| -------- | ---- | -------------------------------------------- | ------------------------------------- |
| x and y  | 与   | 如果 x 和 y 都为 true，则返回 true           | x=6 y=3  (x < 10 and y > 1) 返回 true |
| x or y   | 或   | 如果 x 和 y 至少有一个为 true，则返回 true   | x=6 y=3  (x==6 or y==5) 返回 true     |
| x xor y  | 异或 | 如果 x 和 y 有且仅有一个为 true，则返回 true | x=6 y=3  (x==6 xor y==3) 返回 false   |
| x && y   | 与   | 如果 x 和 y 都为 true，则返回 true           | x=6 y=3 (x < 10 && y > 1) 返回 true   |
| x \|\| y | 或   | 如果 x 和 y 至少有一个为 true，则返回 true   | x=6 y=3 (x==5 \|\| y==5) 返回 false   |
| ! x      | 非   | 如果 x 不为 true，则返回 true                | x=6 y=3 !(x==y) 返回 true             |



##### 数组运算符

| 运算符  | 名称   | 描述                                                         |
| ------- | ------ | ------------------------------------------------------------ |
| x + y   | 集合   | x 和 y 的集合                                                |
| x == y  | 相等   | 如果 x 和 y 具有相同的键/值对，则返回 true                   |
| x === y | 恒等   | 如果 x 和 y 具有相同的键/值对，且顺序相同类型相同，则返回 true |
| x != y  | 不相等 | 如果 x 不等于 y，则返回 true                                 |
| x <> y  | 不相等 | 如果 x 不等于 y，则返回 true                                 |
| x !== y | 不恒等 | 如果 x 不等于 y，则返回 true                                 |



##### 组合比较符（PHP7+）

PHP7+ 支持组合比较符（combined comparison operator）也称之为太空船操作符，符号为 `<=>`。组合比较运算符可以轻松实现两个变量的比较，当然不仅限于数值类数据的比较。

语法格式如下：

```php
$c = $a <=> $b;
```

解析如下：

- 如果 **$a > $b**, 则 **$c** 的值为 **1**。
- 如果 **$a == $b**, 则 **$c** 的值为 **0**。
- 如果 **$a < $b**, 则 **$c** 的值为 **-1**。



### 进阶

#### 数组

数组能够在单个变量中存储多个值。

在 PHP 中，有三种类型的数组：

- **数值数组** - 带有数字 ID 键的数组
- **关联数组** - 带有指定的键的数组，每个键关联一个值
- **多维数组** - 包含一个或多个数组的数组



##### 数值数组

使用`array()`创建数值数据：

```php
<?php
$cars=array("Volvo","BMW","Toyota");
echo "I like " . $cars[0] . ", " . $cars[1] . " and " . $cars[2] . ".";
?>
```



##### 关联数组

创建关联数组

```php
$age=array("Peter"=>"35","Ben"=>"37","Joe"=>"43");
```

获取关联数组里的值

```php
<?php
$age=array("Peter"=>"35","Ben"=>"37","Joe"=>"43");
echo "Peter is " . $age['Peter'] . " years old.";
// 遍历获取
foreach($age as $x=>$x_value)
{
    echo "Key=" . $x . ", Value=" . $x_value;
    echo "<br>";
}
?>
```



##### 多维数组

一个数组的值是另一个数组，以此类推可以创建二维、三维...的数组。

```php
// 二维数组:
$cars = array
(
    array("Volvo",100,96),
    array("BMW",60,59),
    array("Toyota",110,100)
);
// 三维数组
$sites = array 
( 
    "runoob"=>array 
    ( 
        "菜鸟教程", 
        "http://www.runoob.com" 
    ), 
    "google"=>array 
    ( 
        "Google 搜索", 
        "http://www.google.com" 
    ), 
    "taobao"=>array 
    ( 
        "淘宝", 
        "http://www.taobao.com" 
    ) 
); 
```



#### 超全局变量

PHP 超级全局变量列表:

- $GLOBALS
- $_SERVER
- $_REQUEST
- $_POST
- $_GET
- $_FILES
- $_ENV
- $_COOKIE
- $_SESSION



##### $GLOBALS

`$GLOBALS` 是PHP的一个超级全局变量组，在一个PHP脚本的全部作用域中都可以访问。

`$GLOBALS` 是一个包含了全部变量的全局组合数组。变量的名字就是数组的键。

```php
<?php 
$x = 75; 
$y = 25;
 
function addition() 
{ 
    $GLOBALS['z'] = $GLOBALS['x'] + $GLOBALS['y']; 
}
 
addition(); 
echo $z; 
?>
```



##### $_SERVER

`$_SERVER` 是一个包含了诸如头信息(header)、路径(path)、以及脚本位置(script locations)等等信息的数组。

| 元素/代码                       | 描述                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| $_SERVER['PHP_SELF']            | 当前执行脚本的文件名，与 document root 有关。例如，在地址为 http://example.com/test.php/foo.bar 的脚本中使用 $_SERVER['PHP_SELF'] 将得到 /test.php/foo.bar。__FILE__ 常量包含当前(例如包含)文件的完整路径和文件名。 从 PHP 4.3.0 版本开始，如果 PHP 以命令行模式运行，这个变量将包含脚本名。之前的版本该变量不可用。 |
| $_SERVER['GATEWAY_INTERFACE']   | 服务器使用的 CGI 规范的版本；例如，"CGI/1.1"。               |
| $_SERVER['SERVER_ADDR']         | 当前运行脚本所在的服务器的 IP 地址。                         |
| $_SERVER['SERVER_NAME']         | 当前运行脚本所在的服务器的主机名。如果脚本运行于虚拟主机中，该名称是由那个虚拟主机所设置的值决定。(如: www.runoob.com) |
| $_SERVER['SERVER_SOFTWARE']     | 服务器标识字符串，在响应请求时的头信息中给出。 (如：Apache/2.2.24) |
| $_SERVER['SERVER_PROTOCOL']     | 请求页面时通信协议的名称和版本。例如，"HTTP/1.0"。           |
| $_SERVER['REQUEST_METHOD']      | 访问页面使用的请求方法；例如，"GET", "HEAD"，"POST"，"PUT"。 |
| $_SERVER['REQUEST_TIME']        | 请求开始时的时间戳。从 PHP 5.1.0 起可用。 (如：1377687496)   |
| $_SERVER['QUERY_STRING']        | query string（查询字符串），如果有的话，通过它进行页面访问。 |
| $_SERVER['HTTP_ACCEPT']         | 当前请求头中 Accept: 项的内容，如果存在的话。                |
| $_SERVER['HTTP_ACCEPT_CHARSET'] | 当前请求头中 Accept-Charset: 项的内容，如果存在的话。例如："iso-8859-1,*,utf-8"。 |
| $_SERVER['HTTP_HOST']           | 当前请求头中 Host: 项的内容，如果存在的话。                  |
| $_SERVER['HTTP_REFERER']        | 引导用户代理到当前页的前一页的地址（如果存在）。由 user agent 设置决定。并不是所有的用户代理都会设置该项，有的还提供了修改 HTTP_REFERER 的功能。简言之，该值并不可信。) |
| $_SERVER['HTTPS']               | 如果脚本是通过 HTTPS 协议被访问，则被设为一个非空的值。      |
| $_SERVER['REMOTE_ADDR']         | 浏览当前页面的用户的 IP 地址。                               |
| $_SERVER['REMOTE_HOST']         | 浏览当前页面的用户的主机名。DNS 反向解析不依赖于用户的 REMOTE_ADDR。 |
| $_SERVER['REMOTE_PORT']         | 用户机器上连接到 Web 服务器所使用的端口号。                  |
| $_SERVER['SCRIPT_FILENAME']     | 当前执行脚本的绝对路径。                                     |
| $_SERVER['SERVER_ADMIN']        | 该值指明了 Apache 服务器配置文件中的 SERVER_ADMIN 参数。如果脚本运行在一个虚拟主机上，则该值是那个虚拟主机的值。(如：someone@runoob.com) |
| $_SERVER['SERVER_PORT']         | Web 服务器使用的端口。默认值为 "80"。如果使用 SSL 安全连接，则这个值为用户设置的 HTTP 端口。 |
| $_SERVER['SERVER_SIGNATURE']    | 包含了服务器版本和虚拟主机名的字符串。                       |
| $_SERVER['PATH_TRANSLATED']     | 当前脚本所在文件系统（非文档根目录）的基本路径。这是在服务器进行虚拟到真实路径的映像后的结果。 |
| $_SERVER['SCRIPT_NAME']         | 包含当前脚本的路径。这在页面需要指向自己时非常有用。__FILE__ 常量包含当前脚本(例如包含文件)的完整路径和文件名。 |
| $_SERVER['SCRIPT_URI']          | URI 用来指定要访问的页面。例如 "/index.html"。               |



##### $_REQUEST

`$_REQUEST` 用于收集HTML表单提交的数据。

```php+HTML
<html>
<body>

<form method="post" action="<?php echo $_SERVER['PHP_SELF'];?>">
Name: <input type="text" name="fname">
<input type="submit">
</form>

<?php 
$name = $_REQUEST['fname']; 
echo $name; 
?>

</body>
</html>
```



##### $\_POST 与 $\_GET

`$_POST`用于收集通过post方式提交的表单数据，`$_GET`用于收集url中的数据。

```php+HTML
<html>
<body>

<form method="post" action="<?php echo $_SERVER['PHP_SELF'];?>">
Name: <input type="text" name="fname">
<input type="submit">
</form>
<a href="test_get.php?subject=PHP&web=runoob.com">Test $GET</a>

<?php 
// 通过post方式收集
$name = $_POST['fname']; 
echo $name; 
// 通过get方式收集
echo "Study " . $_GET['subject'] . " at " . $_GET['web'];
?>

</body>
</html>
```



#### 循环

##### foreach循环

在php中，foreach循环用于遍历数组。

```php
<?php
$x=array("one","two","three");
foreach ($x as $value)
{
    echo $value . "<br>";
}
?>
```



#### 魔术变量

`__LINE__`：返回文件中当前行号；

`__FILE__`：返回文件的完整路径和文件名；

`__DIR__`：返回文件所在目录；

`__FUNCTION__`、`__METHOD__`：返回该函数被定义时的名称；

`__CLASS__`：返回该类被定义时的名字；

`__NAMESPACE__`：返回当前命名空间的名称。



#### 命名空间

命名空间可以解决以下两类问题：

1. 用户编写的代码与PHP内部的类/函数/常量或第三方类/函数/常量之间的名字冲突。
2. 为很长的标识符名称(通常是为了缓解第一类问题而定义的)创建一个别名（或简短）的名称，提高源代码的可读性。



##### 定义命名空间

通过`namespace`关键字声明

```php
<?php
declare(encoding='UTF-8'); // 声明命名空间之前唯一合法的代码
// 定义代码在 'MyProject' 命名空间中  
namespace MyProject;  
// ... 代码 ...  

// 推荐使用大括号括起来
namespace MyProject {
    const CONNECT_OK = 1;
    class Connection { /* ... */ }
    function connect() { /* ... */  }
}

// 如果声明的时候没有带有名字，则声明的为全局代码
namespace { // 全局代码
session_start();
$a = MyProject\connect();
echo MyProject\Connection::start();
}
?>
```

**注意：在声明命名空间之前唯一合法的代码是用于定义源文件编码方式的 declare 语句。所有非 PHP 代码包括空白符都不能出现在命名空间的声明之前。**



##### 子命名空间

命名空间通过分层级的方式定义：

```php
<?php
namespace MyProject\Sub\Level;  //声明分层次的单个命名空间

const CONNECT_OK = 1;
class Connection { /* ... */ }
function Connect() { /* ... */  }

?>
```



##### 命名空间使用

命名空间的三种访问方式：

1. 非限定名称访问方式；
2. 限定名称访问方式；
3. 完全限定名称访问方式。

```php
<?php
namespace Foo\Bar;
include 'file1.php';

const FOO = 2;
function foo() {}
class foo
{
    static function staticmethod() {}
}

/* 非限定名称 */
foo(); // 解析为 Foo\Bar\foo resolves to function Foo\Bar\foo
foo::staticmethod(); // 解析为类 Foo\Bar\foo的静态方法staticmethod。resolves to class Foo\Bar\foo, method staticmethod
echo FOO; // resolves to constant Foo\Bar\FOO

/* 限定名称 */
subnamespace\foo(); // 解析为函数 Foo\Bar\subnamespace\foo
subnamespace\foo::staticmethod(); // 解析为类 Foo\Bar\subnamespace\foo,
                                  // 以及类的方法 staticmethod
echo subnamespace\FOO; // 解析为常量 Foo\Bar\subnamespace\FOO
                                  
/* 完全限定名称 */
\Foo\Bar\foo(); // 解析为函数 Foo\Bar\foo
\Foo\Bar\foo::staticmethod(); // 解析为类 Foo\Bar\foo, 以及类的方法 staticmethod
echo \Foo\Bar\FOO; // 解析为常量 Foo\Bar\FOO
?>
```



##### 命名空间的导入与别名

使用`use`关键字导入

```php
<?php
namespace foo;
use My\Full\Classname as Another;

// 下面的例子与 use My\Full\NSname as NSname 相同
use My\Full\NSname;

// 导入一个全局类
use ArrayObject;

// importing a function (PHP 5.6+)
use function My\Full\functionName;

// aliasing a function (PHP 5.6+)
use function My\Full\functionName as func;

// importing a constant (PHP 5.6+)
use const My\Full\CONSTANT;

$obj = new namespace\Another; // 实例化 foo\Another 对象
$obj = new Another; // 实例化 My\Full\Classname　对象
NSname\subns\func(); // 调用函数 My\Full\NSname\subns\func
$a = new ArrayObject(array(1)); // 实例化 ArrayObject 对象
// 如果不使用 "use \ArrayObject" ，则实例化一个 foo\ArrayObject 对象
func(); // calls function My\Full\functionName
echo CONSTANT; // echoes the value of My\Full\CONSTANT
?>
```

[命名空间解析规则](https://secure.php.net/manual/zh/language.namespaces.rules.php)



### 函数

php所有內建函数参考链接：[php手册](https://secure.php.net/manual/zh/funcref.php)、[菜鸟教程](http://www.runoob.com/php/php-ref-array.html)



#### 面向对象编程

##### 构造函数

在一个类中定义一个方法作为构造函数

```php
function __construct ([ mixed $args [, $... ]] ){
  // ...
}
```



##### 析构函数

析构函数(destructor) 与构造函数相反，当对象结束其生命周期时（例如对象所在的函数已调用完毕），系统自动执行析构函数。

```php
function __destruct ( void ){
  // ...
}
```



##### 继承

使用`extends`关键字来继承一个类

```php
class Child extends Parent {
   // 代码部分
}
```



##### 访问权限

PHP 对属性或方法的访问控制，是通过在前面添加关键字 public（公有），protected（受保护）或 private（私有）来实现的。默认是**公有属性**。

- `public`（公有）：公有的类成员可以在任何地方被访问。
- `protected`（受保护）：受保护的类成员则可以被其自身以及其子类和父类访问。
- `private`（私有）：私有的类成员则只能被其定义所在的类访问（派生类**无法重写**私有属性）。



##### 接口

使用`interface`关键字定义接口，指定某个类必须要实现某些方法，但是不需要定义这些方法的具体内容。

要实现接口，使用`implements`操作符，类可以实现多个接口，使用逗号分隔开多个接口的名称。

```php
<?php

// 声明一个'iTemplate'接口
interface iTemplate
{
    public function setVariable($name, $var);
    public function getHtml($template);
}


// 实现接口
class Template implements iTemplate
{
    private $vars = array();
  
    public function setVariable($name, $var)
    {
        $this->vars[$name] = $var;
    }
  
    public function getHtml($template)
    {
        foreach($this->vars as $name => $value) {
            $template = str_replace('{' . $name . '}', $value, $template);
        }
 
        return $template;
    }
}
```



##### 抽象类

使用`abstract`关键字定义抽象类与抽象方法。

一个类里只要用一个方法被声明为抽象，那么这个类就必须声明为抽象类。被定义为抽象的方法只需要声明其调用方式（类似接口），继承抽象类的时候，子类必须定义父类中的所有抽象方法。另外，这些方法的访问权限必须和父类定义的访问权限相同或**更宽松**（从严谨到宽松的排序：`private`、`protected`、`public`）。

```php
<?php
abstract class AbstractClass
{
 // 强制要求子类定义这些方法
    abstract protected function getValue();
    abstract protected function prefixValue($prefix);

    // 普通方法（非抽象方法）
    public function printOut() {
        print $this->getValue() . PHP_EOL;
    }
}

class ConcreteClass1 extends AbstractClass
{
  // 子类必须定义父类中的所有抽象方法
    protected function getValue() {
        return "ConcreteClass1";
    }

    public function prefixValue($prefix) {
        return "{$prefix}ConcreteClass1";
    }
}

class ConcreteClass2 extends AbstractClass
{
  // 子类必须定义父类中的所有抽象方法
    public function getValue() {
        return "ConcreteClass2";
    }

    public function prefixValue($prefix) {
        return "{$prefix}ConcreteClass2";
    }
}

$class1 = new ConcreteClass1;
$class1->printOut();
echo $class1->prefixValue('FOO_') . PHP_EOL;

$class2 = new ConcreteClass2;
$class2->printOut();
echo $class2->prefixValue('FOO_') . PHP_EOL;
?>
```



##### 静态属性

使用`static`关键字声明一个类的属性或者方法为静态，静态方法不需要实例化类即可直接访问。

```php
<?php
class Foo {
  public static $my_static = 'foo';
  
  public function staticValue() {
     return self::$my_static;
  }
}

print Foo::$my_static . PHP_EOL;
$foo = new Foo();

print $foo->staticValue() . PHP_EOL;
?>    
```



##### `final`关键字

如果父类中的方法被声明为 `final`，则子类无法覆盖该方法。如果一个类被声明为 `final`，则不能被继承。

```php
<?php
class BaseClass {
   public function test() {
       echo "BaseClass::test() called" . PHP_EOL;
   }
   // 该方法无法被重写和继承
   final public function moreTesting() {
       echo "BaseClass::moreTesting() called"  . PHP_EOL;
   }
}

class ChildClass extends BaseClass {
   public function moreTesting() {
       echo "ChildClass::moreTesting() called"  . PHP_EOL;
   }
}
// 报错信息 Fatal error: Cannot override final method BaseClass::moreTesting()
?>
```



##### 子类调用父类构造方法

PHP 不会在子类的构造方法中自动的调用父类的构造方法。要执行父类的构造方法，需要在子类的构造方法中调用 `parent::__construct()` 。

```php
<?php
class BaseClass {
   function __construct() {
       print "BaseClass 类中构造方法" . PHP_EOL;
   }
}
class SubClass extends BaseClass {
   function __construct() {
       parent::__construct();  // 子类构造方法不能自动调用父类的构造方法
       print "SubClass 类中构造方法" . PHP_EOL;
   }
}
class OtherSubClass extends BaseClass {
    // 继承 BaseClass 的构造方法
}

// 调用 BaseClass 构造方法
$obj = new BaseClass();    // 输出“BaseClass 类中构造方法”

// 调用 BaseClass、SubClass 构造方法
$obj = new SubClass();    // 输出：“BaseClass 类中构造方法  SubClass 类中构造方法”

// 调用 BaseClass 构造方法
$obj = new OtherSubClass();    // 输出：“BaseClass 类中构造方法”
?>
```



### 高级

#### 数据库操作

[菜鸟MYSQL教程](http://www.runoob.com/mysql/mysql-tutorial.html)

```php
<?php
$servername = "localhost";
$username = "username";
$password = "password";
 
/* 创建连接 */
$conn = new mysqli($servername, $username, $password);
 
// 检测连接
if ($conn->connect_error) {
    die("连接失败: " . $conn->connect_error);
} 
echo "连接成功";
// 设置编码，防止中文乱码
mysqli_set_charset($con, "utf8");

/* 创建数据库 */
$createSql = "CREATE DATABASE myDB";
if ($conn->query($createSql) === TRUE) {
    echo "数据库创建成功";
} else {
    die("Error creating database: " . $conn->error);
}

/* 使用 sql 创建数据表 */
$sql = "CREATE TABLE MyGuests (
id INT(6) UNSIGNED AUTO_INCREMENT PRIMARY KEY, 
firstname VARCHAR(30) NOT NULL,
lastname VARCHAR(30) NOT NULL,
email VARCHAR(50),
reg_date TIMESTAMP
)";
 
if ($conn->query($sql) === TRUE) {
    echo "Table MyGuests created successfully";
} else {
    die("创建数据表错误: " . $conn->error);
}

/* 插入多条数据 */
$sql .= "INSERT INTO MyGuests (firstname, lastname, email)
VALUES ('John', 'Doe', 'john@example.com')";
$sql .= "INSERT INTO MyGuests (firstname, lastname, email)
VALUES ('John', 'Doe', 'john@example.com')";

if ($conn->query($sql) === TRUE) {
    echo "新记录插入成功";
} else {
    die("Error: " . $sql . "<br>" . $conn->error);
}

/* 读取数据 */
$sql = "SELECT id, firstname, lastname FROM MyGuests";
$result = $conn->query($sql);
 
if ($result->num_rows > 0) {
    // 输出数据
    while($row = $result->fetch_assoc()) {
        echo "id: " . $row["id"]. " - Name: " . $row["firstname"]. " " . $row["lastname"]. "<br>";
    }
} else {
    echo "0 结果";
}


// 关闭连接
$conn->close();
?>
```



更新数据

我们创建了一个名为 "Persons" 的表，如下所示：

| FirstName | LastName | Age  |
| --------- | -------- | ---- |
| Peter     | Griffin  | 35   |
| Glenn     | Quagmire | 33   |

下面的例子更新 "Persons" 表的一些数据：

```php
<?php
$con=mysqli_connect("localhost","username","password","database");
// 检测连接
if (mysqli_connect_errno())
{
    echo "连接失败: " . mysqli_connect_error();
}

mysqli_query($con,"UPDATE Persons SET Age=36
WHERE FirstName='Peter' AND LastName='Griffin'");

mysqli_close($con);
?>
```

在这次更新后，"Persons" 表如下所示：

| FirstName | LastName | Age  |
| --------- | -------- | ---- |
| Peter     | Griffin  | 36   |
| Glenn     | Quagmire | 33   |



##### 使用事务进行数据库操作（提高写入速度）

```php
<?php
$mysqli = new mysqli("127.0.0.1","root","111111","test");
if ($mysqli->connect_errno){
	echo "Failed to connect to MySQL: " . $mysqli->connect_error;
}
//默认auto-commit 是打开的，可直接提交sql语句
$b1 = $mysqli->query("INSERT INTO test (id,name) values (0,'000')");
//若要执行事务，先关闭auto-commit
//关闭 auto-commit
$mysqli->autocommit(false);
$res1 = $mysqli->query("INSERT INTO test (id,n1ame) values (1,'111')");
$b2 = $mysqli->query("INSERT INTO test (id,name) values (2,'222')");
//有一条失败则回滚，全部成功则提交
if(!$res1 || !$b2){
    echo '回滚！';
    $mysqli->rollback();
} else {
    echo '提交！';
    $mysqli->commit();
}
//若想再次自动提交非事务语句，要打开auto-commit
$mysqli->autocommit(true);
$b3 = $mysqli->query("INSERT INTO test (id,name) values (3,'333')");
//关闭连接
$mysqli->close();
```

1. `SQL语句是有长度限制`，在进行数据合并在同一SQL中务必不能超过SQL长度限制，通过max_allowed_packet配置可以修改，默认是1M，测试时修改为8M。
2. `事务需要控制大小`，事务太大可能会影响执行的效率。MySQL有innodb_log_buffer_size配置项，超过这个值会把innodb的数据刷到磁盘中，这时，效率会有所下降。所以比较好的做法是，在数据达到这个这个值前进行事务提交。



##### 设置utf-8字符串

在数据库连接完成后，添加以下代码

```php
/**
 * sql连接初始化
 */
function sql_init()
{
    $mysqli = new mysqli(HOST, SQL_USERNAME, SQL_PASSWORD, SQL_DBNAME, SQL_PORT);
    // 我使用下面这个方式设置字符集，但是返回的中文数据仍然是乱码
    $mysqli->set_charset("utf-8");
    $mysqli->query("set names 'utf8' ");
    // $mysqli->query("set character_set_client=utf8");
    $mysqli->query("set character_set_results=utf8");
    return $mysqli;
}
```



##### 数据更新

想对数据库里的某一条数据的某一项或多项进行更新，需要用到`ON DUPLICATE KEY UPDATE`

| id   | num         | date          | remarks |
| ---- | ----------- | ------------- | ------- |
| 1    | 13488889999 | 1550131999000 | test    |

> 需求是：当原来数据库里没有这一个`num`的数据时，新增这一条数据，否则更新`data`和`remarks`。

`id`是设置了主键的，保证了它的唯一性，然后需要给`num`设置**唯一**索引，当需要增加或者更新数据时，执行下面的语句:

```sql
INSERT INTO db (num,date,remarks) VALUES (13488889999,1549873770000,"msg") ON DUPLICATE KEY UPDATE date=1549873770000, remarks= "msg";
```

新增时，受影响的行数为1，更新时受影响的行数为2。



#### session

##### 存储与获取

在进行session操作的时候，需要先通过`session_start()`启动会话。

```php
<?php
session_start();
// 存储 session 数据
$_SESSION['views']=1;

// 获取数据
echo $_SESSION['views'];
?>
```

##### 销毁

如果您希望删除某些 session 数据，可以使用 `unset()` 或 `session_destroy()` 函数。

`unset()` 函数用于释放指定的 session 变量：

```php
<?php
session_start();
if(isset($_SESSION['views']))
{
    unset($_SESSION['views']);
}
?>
```

`unset($_SESSION)` 此函数千万不可使用，它会将全局变量$\_SESSION销毁，而且还没有可行的办法将其恢复。用户也不再可以注册$_session变量。

您也可以通过调用 `session_destroy()` 函数彻底销毁 session,结束当前的会话，并清空会话中的所有资源：

```php
<?php
session_destroy();
?>
```

**注释：**`session_destroy()` 将重置 session，您将失去所有已存储的 session 数据。



#### 文件操作

前端使用ajax将文件上传后，后端先将文件从缓存文件夹移动到你要存放文件的文件夹

```js
/* js */
$.ajax({
    url:"/studyPHP/php/updata.php",
    type:"post",
    data:data,
    processData:false,
    contentType:false,
    dataType:"json",
    success:function(res) {
      console.log(res);
    }
})
```



```php
/* php */
if (is_uploaded_file($_FILES['file']['tmp_name'])) {
    $root_dir = "/studyPHP/upData_files/";
    $filename = $_FILES['file']['name'];
    $filename = time() . $filename;
    $uploadfile = $root_dir . $filename;

    if (move_uploaded_file($_FILES['file']['tmp_name'], $uploadfile)) {
      $res = array('code' => 200, 'msg' => "文件已保存");
      echo json_encode($res,JSON_UNESCAPED_UNICODE);
    }
  }
```

**请注意**：在后端处理文件的时候，需要留意文件夹的权限问题，否则将会在执行`move_uploaded_file`时将会报错。

此时，若要读取文件，可以使用`fgets()`加`feof()`来分行读取文件

```php
/* php */
$myFile = fopen($uploadfile, "r");
$arr = array();
$j = 0;
while (!feof($myFile)) {
  $a = mb_convert_encoding(fgets($myFile), "UTF-8", "GBK,ASCII,ANSI,UTF-8");
  $arr[$j] = trim($a);
  // dosomething...
  $j++;
}
fclose($myFile);
```



#### 跨域处理

需要在后端接口入口页面设置允许跨域

```php
header('Access-Control-Allow-Origin:*');
header('Access-Control-Allow-Headers:content-type,token,id');
header("Access-Control-Request-Headers: Origin, X-Requested-With, content-Type, Accept, Authorization");
```

如果遇到set-cookies无法存储的问题时，需要做如下设置

```php
// 后端设置
header('Access-Control-Allow-Credentials:true');
// 设置了这个之后，Access-Control-Allow-Origin需要设置具体域名，不能使用*
```

```js
// 前端在请求头中设置
withCredentials:true
```

