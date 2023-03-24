# Laravel笔记

使用Laravel 6版本。[文档地址](https://learnku.com/docs/laravel/6.x/installation/5124)



## 数据库初始化

### 配置

在`/config/database.php`里修改正确的数据库连接配置



### 创建数据库表文件

在Laravel项目内创建数据库文件：`/database/migrations/`，创建表的逻辑需要写在`up`方法内，需要注意每创建了一个表，需要在`down`方法内写一个删除表的方法（`dropIfExists({name})`），用于回滚。[文档地址](https://learnku.com/docs/laravel/6.x/migrations/5173#creating-tables)



### 创建数据库表模型

在数据库里取数据的时候，有一些敏感字段需要进行隐藏，这时可以在`/app/Models/`里创建对应表名的php文件（首字母大写），并在里面添加`$hidden`属性；在对数据库进行编辑/新增的时候，你还可以设置哪些字段允许更改，使用`$fillable`属性来实现。

```php
<?php

namespace App\Models;

use Eloquent;

class Admin extends Eloquent
{
    protected $table = 'admin_list';
    /*
     * The attributes that are mass assignable.
     *
     * @var array
     */
    // 插入数据时可更改的字段
    protected $fillable = [
        'username', 'password'
    ];

    /*
     * The attributes that should be hidden for arrays.
     *
     * @var array
     */
    // 取出数据时隐藏的字段
    protected $hidden = [
        'password'
    ];
}
```



准备工作完成后即可执行命令进行数据库迁移（创建数据库）：`php artisan migrate`



## 使用前端脚手架开发后端视图页面

在安装完Laravel后，也许你想使用vue或者react开发视图页面， Laravel 提供的引导和 vue 脚手架位于 `laravel/ui` composer 包中，可以使用 composer 进行安装： 

```php
composer require laravel/ui --dev
```

安装完成后就可以生成对应的脚手架了

```php
// 生成基本脚手架
php artisan ui vue
php artisan ui react

npm install
```

安装热更新调试模块

```shell
 npm install browser-sync browser-sync-webpack-plugin@2.0.1 --save-dev --production=false
```

在`/webpack.mix.js`中配置热更新

```js
// 在 webpack.mix.js 中添加配置
mix.browserSync({
    proxy: 'localhost:8000'
});
```

执行前端视图热更新调试

```shell
npm run watch
```

启动Laravel

```she
php artisan serve --host=localhost --port=3000
```

这时，`/resources/views/welcome.blade.php` 相当于vue脚手架里的`app.vue`，`/resources/js/app.js`相当于`main.js`

