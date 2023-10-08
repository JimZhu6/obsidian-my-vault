# php-codeIgniter

## 安装

[前往官网](http://codeigniter.org.cn/)下载最新版本并解压至服务器文件夹。

## 主题

### 控制器

#### url分段

`example.com/class/function/ID`

1. 第一段表示要调用的控制器**类**；（文件夹：`application/controllers`）
2. 第二段表示要调用类中的指定**函数**或**方法**；
3. 第三段及其后面所有代表传给控制器的参数。

#### 隐藏url中的index.php

默认情况下，url会包含index.php，如果你的服务器是使用Apache，且启用了mod\_rewrite，则可以在项目的根目录添加一个`.htaccess`文件，里面内容：

```
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond $1 !^(index\.php|images|css|js|robots\.txt) 	# 这里写要排除的资源
RewriteRule ^(.*)$ index.php/$1 [L]
```

#### 启用查询字符串

在 `application/config.php` 配置文件中启用它。 打开你的配置文件，查找下面这几项：

```php
$config['enable_query_strings'] = FALSE;
$config['controller_trigger'] = 'c';
$config['function_trigger'] = 'm';
```

将`enable_query_strings`设置为`true`，设置后，url格式变成：`index.php?c=controller&m=method`。其中`controller`是控制器的类名，`method`是要执行的方法名。

#### 处理输出

**控制器**里创建一个`_output()`方法，它会将数据的最终状态发送到浏览器。

当数据传到 `_output()` 方法时，数据已经是最终状态。这时基准测试和计算内存占用都已经完成， 缓存文件也已经写到文件（如果你开启缓存的话），HTTP 头也已经发送。 为了使你的控制器能正确处理缓存，`_output()` 可以这样写:

```php
if ($this->output->cache_expiration > 0)
{
    $this->output->_write_cache($output);
}
```

#### 私有方法

将方法声明为 private 或 protected，在方法名前加上一个下划线前缀也可以让该方法无法访问。

```php
private function _utility()
{
    // some code
}
```

#### 构造函数

当有需要在**控制器**中使用构造函数时，需要用代码覆盖父类的构造函数，需要手工调用它

```php
<?php
class Blog extends CI_Controller {
    public function __construct()
    {
        // 需要添加下面这一行
        parent::__construct();
        // Your own constructor code
    }
}
```

### 视图

#### 创建与加载视图

在`application/views/`创建一个php文件，里面写`html`内容，在**控制器**里使用下面的方法来加载指定视图

```php
$this->load->view('name');
```

name参数是视图的文件名。

#### 向视图添加动态数据

在上面加载视图的方法的第二个参数，可以像视图动态传入数据，这个参数可以使**数组**或者**对象**。

```php
// 数组
$data = array(
    'title' => 'My Title',
    'heading' => 'My Heading',
    'message' => 'My Message'
);
// 对象
$data = new Someclass();

$this->load->view('blogview', $data);
```

在视图文件写入数组相对于的变量

```php+HTML
<html>
<head>
    <title><?php echo $title;?></title>
</head>
<body>
    <h1><?php echo $heading;?></h1>
</body>
</html>
```

#### 循环

用于在视图页面循环一个多维数组

```php
<?php
class Blog extends CI_Controller {

    public function index()
    {
        $data['todo_list'] = array('Clean House', 'Call Mom', 'Run Errands');

        $data['title'] = "My Real Title";
        $data['heading'] = "My Real Heading";

        $this->load->view('blogview', $data);
    }
}
```

在视图页面展开循环

```php+HTML
<html>
<head>
    <title><?php echo $title;?></title>
</head>
<body>
    <h1><?php echo $heading;?></h1>
    <h3>My Todo List</h3>
    <ul>
    <?php foreach ($todo_list as $item):?>
        <li><?php echo $item;?></li>
    <?php endforeach;?>
    </ul>
</body>
</html>
```

### 模型

#### 创建模型

在`application/models/`目录下创建模型php文件

```php
class Model_name extends CI_Model {
    public function __construct()
    {
        parent::__construct();
        // Your own constructor code
    }
}
```

文件名和类名应该一致，如上方的模型，其文件名应该为`Model_name.php`；

#### 加载模型

在**控制器**的方法中加载和调用

```php
 $this->load->model('model_name');
// 如果是在子目录里，需要带上文件夹名称
$this->load->model('blog/queries');

// 加载之后，就可以通过类名的方式调用模型里面的方法
$this->load->model('model_name');
$this->model_name->method();

// 如果你想将你的模型对象赋值给一个不同名字的对象，你可以使用 $this->load->model() 方法的第二个参数:
$this->load->model('model_name', 'foobar');
$this->foobar->method();


```

如果你有一个模型需要在整个应用程序中使用，需要在CodeIgniter初始化的时候就自动加载它，需要在`application/config/autoload.php`文件将该模型添加到`autoload['model']`数组中。

### 辅助函数

[辅助函数参考](http://codeigniter.org.cn/user\_guide/helpers/index.html)

#### 加载辅助函数

在**控制器**使用下面的方法加载辅助函数

```php
$this->load->helper('name');

// 假如要加载URL辅助函数
$this->load->helper('url');

// 加载多个辅助函数
$this->load->helper(array('helper1', 'helper2', 'helper3'));
```

如果有辅助函数需要在整个应用程序中使用，在`application/config/autoload.php`中的`$autoload['helper']`添加它们。

#### 扩展辅助函数

在`application/helpers/`目录下新建一个文件，文件名以**MY\_**开头，以**\_helper**结尾。

> 开头的**MY\_**可以通过配置项`$config['subclass_prefix']`进行修改。

例如，写一个数组判断的辅助函数，创建文件`application/helpers/MY_array_helper.php`

```php
function any_in_array($needle, $haystack)
{
    $needle = is_array($needle) ? $needle : array($needle);
    foreach ($needle as $item)
    {
        if (in_array($item, $haystack))
        {
            return TRUE;
        }
        }
    return FALSE;
}
```

### CodeIgniter类库

[类库参考](http://codeigniter.org.cn/user\_guide/libraries/index.html)

#### 加载类库

使用**控制器**初始化：

```php
// 调用表单验证类库
$this->load->library('form_validation');

// 多类库同时加载
$this->load->library(array('email', 'table'));
```

#### 创建类库

> 除了**数据库类**不能被扩展和替换以外，其他的类都可以。

新建的类库文件需要存储在`application/libraries`目录下。

命名约定

* 文件名首字母需要大写；
* 类名首字母需要大写；
* 类名和文件名必须一致。

```php
<?php
defined('BASEPATH') OR exit('No direct script access allowed');
class Someclass {
    public function some_method()
    {
    }
}
```

在**控制器**加载上面的那个自己创建的类

```php
$this->load->library('Someclass')
```

加载后，即可使用小写字母命名的方式来访问你的类

```php
$this->someclass->some_method();
```

如果需要在**类库初始化传入参数**，可以通过第二个参数动态传递一个数组。

```php
$params = array('type' => 'large', 'color' => 'red');
$this->load->library('someclass', $params);
```

如果你使用了该功能，你必须在定义类的构造函数时加上参数:

```php
<?php defined('BASEPATH') OR exit('No direct script access allowed');
class Someclass {
    public function __construct($params)
    {
        // Do something with $params
    }
}
```

#### 在创建的类库中使用CodeIgniter资源

在一般情况下，在**控制器**里使用CodeIgniter原生资源会使用`$this`来调用，但是如果要在自己创建的类中使用Codeigniter资源，则需要通过`get_instance()`函数来访问。将这个函数赋值给一个变量后，就可以用这个变量代替`$this`。

```php
$CI =& get_instance();

$CI->load->helper('url');
$CI->load->library('session');
$CI->config->item('base_url');
```

> 你会看到上面的 `get_instance()` 函数通过引用来传递:
>
> ```php
> $CI =& get_instance();
> ```
>
> 这是非常重要的，引用赋值允许你使用原始的 CodeIgniter 对象，而不是创建一个副本。

#### 使用新建的类库替换原生类库

简单的将你的类文件名改为和原生的类库文件一致，CodeIgniter 就会使用它替换掉原生的类库。 要使用该功能，你必须将你的类库文件和类定义改成和原生的类库完全一样，例如， 要替换掉原生的 Email 类的话，你要新建一个 `application/libraries/Email.php` 文件， 然后定义定义你的类:

```php
class CI_Email {}
```

注意大多数原生类都以 CI\_ 开头。

要加载你的类库，和标准的方法一样:

```php
$this->load->library('email');
```

**注意数据库类不能被你自己的类替换掉。**

#### 扩展原生类库

在扩展类的时候需要注意：

* 类的定义时需要继承它的父类
* 新的类名和文件名需要以**MY\_**为前缀（可通过配置项`$config['subclass_prefix']`进行修改）

例如，要扩展原生的 Email 类你需要新建一个文件命名为 _application/libraries/MY\_Email.php_ ， 然后定义你的类:

```php
class MY_Email extends CI_Email {}
```

如果你需要在你的类中使用构造函数，确保你调用了父类的构造函数:

```php
class MY_Email extends CI_Email {
    public function __construct($config = array())
    {
        parent::__construct($config);
    }
}
```

**并不是所有的类库构造函数的参数都是一样的，在对类库扩展之前 先看看它是怎么实现的。**

#### 加载扩展的类

要加载你的扩展类，还是使用和通常一样的语法。不用包含前缀。例如， 要加载上例中你扩展的 Email 类，你可以使用:

```php
$this->load->library('email');
```

### 资源自动加载

* libraries/ 目录下的核心类
* helpers/ 目录下的辅助函数
* config/ 目录下的用户自定义配置文件
* system/language/ 目录下的语言文件
* models/ 目录下的模型类

修改配置文件`application/config/autoload.php`，将其数组添加你要自动加载的资源。

另外，如果你想让 CodeIgniter 使用 [Composer](https://getcomposer.org/) 的自动加载， 只需将 `application/config/config.php` 配置文件中的 `$config['composer_autoload']` 设置为 `TRUE` 或者设置为你自定义的路径。

## 小技巧

### 引用静态资源

将需要引用的静态资源放在根目录，设置CI框架的`base_url`，设置自动加载`url`辅助函数。

然后就可以在视图文件里进行引入：

```php+HTML
<link rel="stylesheet" href="<?=base_url()?>/assets/css/index.css">
<script src="<?=base_url()?>/assets/js/index.js"></script>
```

### 使用Redis

配置文件`application/config/config.php`中的`$config['sess_driver']`修改为`redis`。

在`application/config/`目录下创建一个名为`redis.php`配置文件，可用参数如下：

```php
$config['socket_type'] = 'tcp'; //`tcp` or `unix`
$config['socket'] = '/var/run/redis.sock'; // in case of `unix` socket type
$config['host'] = '127.0.0.1';
$config['password'] = NULL;
$config['port'] = 6379;
$config['timeout'] = 0;
```

使用方法例子

```php
$this->load->driver('cache');
$this->cache->redis->save('foo', 'bar', 10);
```

### 引用composer安装的库

在项目跟目录下载号需要安装的包后，需要修改`application/config/config.php`里面的`$config['composer_autoload']`，指向vendor文件夹的`autoload.php`文件。

然后在你需要用到这个库的**控制器**使用就可以了。

### 部署CI框架的项目到服务器后遇到的问题

#### 提示session路径错误

前往`config.php`文件内将`$config['sess_save_path'] = NULL;`修改为`$config['sess_save_path'] = sys_get_temp_dir();`

#### 访问CI接口404

设置伪静态：

```
# CI接口在api文件夹内
if (!-f $request_filename){
  set $rule_0 1$rule_0;
}
if (!-d $request_filename){
  set $rule_0 2$rule_0;
}
if ($rule_0 = "21"){
  rewrite ^/api/(.*)$ /api/index.php/$1 last;
}
```

或者

```
# CI直接在根目录部署
if ($request_uri ~* ^/system)
{
 rewrite ^/(.*)$ /index.php?/$1 last;
 break;
}

if (!-e $request_filename)
{
 rewrite ^/(.*)$ /index.php?/$1 last;
 break;
}
```

### 通过axios使用post方式提交的错误

当前端项目是使用`axios`时，通过post提交数据到CI框架的后台时，后台通过`$_POST`将接收不到这些数据，原因是：

当我们使用`axios`时，`axios`将会帮我们将**请求数据**和**响应数据**将会自动转换为**JSON**数据。这时，我们的请求中的`Content-Type`会变成`application/json;charset=utf-8`。因为我们的参数是 JSON 对象，axios 帮我们做了一个 stringify 的处理。

然而，我们服务端的要求是`Content-Type': 'application/x-www-form-urlencoded`，所以是接收不到前端发送的数据的。

#### 解决方式1：使用URLSearchParams 传参

使用[URLSearchParams](https://developer.mozilla.org/zh-CN/docs/Web/API/URLSearchParams)对URL进行字符串处理，需要注意的是，**IE浏览器不支持这个api**。

```javascript
let param = new URLSearchParams()
param.append('username', 'admin')
param.append('pwd', 'admin')
axios({
	method: 'post',
	url: '/api/lockServer/search',
	data: param
})
```

#### 解决方式2：使用transformRequest转换数据

```js
import Qs from 'qs'
axios({
	url: '/api/lockServer/search',
	method: 'post',
	transformRequest: [function (data) {
	    // 对 data 进行任意转换处理
	    return Qs.stringify(data)
    }],
	data: {
	    username: 'admin',
		pwd: 'admin'
	}
})
```

#### 解决方式3：通过字符串拼接的方式传参

```js
axios.post('/api/lockServer/search',"userName='admin'&pwd='admin'");
```

### 修改开发/线上环境

根据当前环境是“开发环境”或是“线上环境”以处理程序异常时是否抛出错误信息。

修改`.htaccess`文件

```php
# 添加这一行代表处于开发环境
SetEnv CI_ENV development

# 添加这一行代表处于线上环境，不显示错误信息
SetEnv CI_ENV production
```

具体错误信息处理，可在`index.php`文件头部进行处理

### 部署线上环境后提示system文件指向错误

在IIS服务器上遇到过这个问题，部署完成后提示：Your system folder path does not appear to be set correctly. Please open the following file and correct this: index.php

可将CI根目录的index.php文件的两个路径指向更改为：

```php
$system_path = dirname(__FILE__) . DIRECTORY_SEPARATOR . 'system';
$application_folder = dirname(__FILE__) . DIRECTORY_SEPARATOR . 'application';
```
