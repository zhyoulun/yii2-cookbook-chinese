# 第六章 RESTful web服务

在本章中，我们将会讨论如下话题：

- 创建一个REST服务器
- 身份校验
- 频率限制
- 版本控制
- 错误处理

## 介绍

本章将会帮助你了解一些便利的东西，包括Yii URL路由，控制器和视图。你将能够更加灵活的创建自己的控制器和视图。

## 创建一个REST服务器

在接下来的章节中，我们使用了一个例子，给你演示如何用最少的代码，简历和设置RESTful API。这小节会在本章的其它小节中复用。

### 准备

1. 按照官方指南[http://www.yiiframework.com/doc-2.0/guide-start-installation.html](http://www.yiiframework.com/doc-2.0/guide-start-installation.html)的描述，使用Composer包管理器创建一个新的应用。
2. 使用如下命令创建一个migration，用于创建一个文章表：

```
./yii migrate/create create_film_table
```

3. 然后，使用如下代码更新刚刚创建的migration方法，`up`：

```
public function up()
{
    $tableOptions = null;
    if ($this->db->driverName === 'mysql') {
        $tableOptions = 'CHARACTER SET utf8 COLLATE utf8_general_ci ENGINE=InnoDB';
    }
    $this->createTable('{{%film}}', [
        'id' => $this->primaryKey(),
        'title' => $this->string(64)->notNull(),
        'release_year' => $this->integer(4)->notNull(),
    ], $tableOptions);
    $this->batchInsert('{{%film}}',
        ['id','title','release_year'], [
            [1, 'Interstellar', 2014],
            [2, "Harry Potter and the Philosopher's Stone",2001],
            [3, 'Back to the Future', 1985],
            [4, 'Blade Runner', 1982],
            [5, 'Dallas Buyers Club', 2013],
        ]);
}
```

使用如下代码更新`down`方法：

```
public function down()
{
    $this->dropTable('film');
}
```

4. 运行创建好的migration `create_film_table`。
5. 使用Gii生成`Film`模型。
6. 使用整洁的URL配置你的应用服务器。如果你使用带有`mod_rewrite`的Apache服务器，并将`AllowOverride`打开，那么你就可以添加如下代码到`@web`目录下的`.htaccess`文件中：

```
Options +FollowSymLinks
IndexIgnore */*
RewriteEngine on
# if a directory or a file exists, use it directly
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
# otherwise forward it to index.php
RewriteRule . index.php
```

### 如何做...

1. 使用如下代码创建控制器`@app/controller/FilmController.php`：

### 工作原理...

### 更多...

#### Content negotiation

#### 自定义REST URL规则

### 参考

## 身份校验

### 准备

### 如何做...

![](../images/601.png)

### 工作原理...

### 更多...

### 参考

## 频率限制

### 准备

### 如何做...

### 工作原理...

### 参考

## 版本控制

### 准备

### 如何做...

### 工作原理...

### 更多...

## 错误控制

### 准备

### 如何做...

### 工作原理...

### 参考





