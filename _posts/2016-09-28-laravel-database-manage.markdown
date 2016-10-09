---
layout: post
title: "Laravel框架下的数据库管理"
date: 2016-09-19
author: Vincent, Dong
category: PHP
tags: Laravel PHP
finished: false
---

## 创建一个数据模型

在项目路径下执行：

`$ php artisan make:model -m ModelName`

-m 选参表示为该数据模型创建 migration 数据库迁移文件

然后就可以在/app路径下面看到新建好的model文件，打开文件。  
为模型设置指定的配置文件中的数据库链接名称（若使用默认数据库链接可以不设置）：

`protected $connection = 'db_config_name';`

为模型指定对应的数据表名称（缺省为模型名称的小写复数形式，Person的复数形式为people，类似同理）：

`protected $table = 'table_name';`

为模型指定对应数据表的主键字段名称（缺省为id）：

`protected $primaryKey = 'primary_key_name';`

在默认情况下，在数据库中有 updated_at 和 created_at 两个字段，如果不想设定或自动更新这两个字段，则做如下设置：

`protected $timestamps = false;`

详细的关于 laravel 数据模型的使用可以参照：

https://laravel-china.org/docs/5.0/eloquent

## 创建一个数据表迁移

这里创建一个数据表的迁移文件， 若前面创建model的时候已经使用-m选项自动创建了迁移文件，则此处可以不必重复创建。

在项目路径下执行：

`$ sudo php artisan make:migration create_tablename_table --create=tablename`

这里使用--create=tablename选项的好处是，artisan程序会在生成的迁移文件up方法中自动加入:  
{% highlight md %}
Schema::create('tablename', function (Blueprint $table) {
    $table->increments('id');
    $table->timestamps();
});
{% endhighlight %}

以及在down方法中加入:  
{% highlight md %}
Schema::create('tablename', function (Blueprint $table) {
    Schema::drop('tablename');
});
{% endhighlight %}

如果不使用--create选项，则up和down方法中都为空，其中内容需要你手动填写。

除了--create选项之外，我们在生成非创建表的迁移中，例如生成更改表的某个字段的迁移文件，可以使用--table=tablename属性来自动生成代码。

## 执行迁移文件

创建完数据表迁移文件，并添加了其中的字段内容之后，执行下面命令执行迁移文件：

`$ sudo php artisan migrate --database=db_config_name`

这其中的db_config_name为你在配置文件中配置的数据库链接的名称。

## 回滚迁移操作

如果你发现之前执行的迁移文件中有错误，那么此时在还未向数据表中插入数据的时候，我们可以对之前的迁移命令进行回滚，这样便可以撤销之前的错误。

`$ sudo php artisan migrate:rollback --database=db_config_name`

如果此时提示你出现找不到类文件的错误，那么需要更新自动载入路径，请执行下面命令：

`$ composer dump-autoload`

## 生成控制器文件

在数据模型和底层数据架构完毕后，我们需要建立对应的控制器来处理用户的请求逻辑。

在项目路径下执行：

`$ php artisan make:controller PhotoController`

接着，我们需要注册一个指向此控制器的资源路由：

Route::resource('photo', 'PhotoController');

### 由资源控制器处理的行为

动词 | 路径 | 行为 | 路由名称
GET | /photo | 索引 | photo.index
GET | /photo/create | 创建 | photo.create
POST | /photo | 保存 | photo.store
GET | /photo/{photo_id} | 显示 | photo.show
GET | /photo/{photo_id}/edit | 编辑 | photo.edit
PUT/PATCH | /photo/{photo_id} | 更新 | photo.update
DELETE | /photo/{photo_id} | 删除 | photo.destroy

## 总结

使用migration文件管理数据库的好处是

