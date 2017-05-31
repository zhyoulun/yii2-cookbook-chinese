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

```
<?php
namespace app\controllers;
use yii\rest\ActiveController;
class FilmController extends ActiveController
{
    public $modelClass = app\models\Film';
}
```

更新配置文件`@app/config/web.php`。添加`urlManager`组件：

```
'urlManager' => [
    'enablePrettyUrl' => true,
    'enableStrictParsing' => true,
    'showScriptName' => false,
    'rules' => [
        ['class' => 'yii\rest\UrlRule', 'controller' =>
            'films'],
    ],
],
```

2. 在`@app/config/web.php`中重新配置请求组件：

```
'request' => [
    'cookieValidationKey' => 'mySecretKey',
    'parsers' => [
        'application/json' => 'yii\web\JsonParser',
    ],
]
```

### 工作原理...

我们通过扩展`\yii\rest\ActiveController`来创建自己的控制器，然后设置这个控制器的`modelClass`属性。`\yii\rest\ActiveController`实现了常用动作的集合，以支持对ActiveRecord的RESTful访问。

上边通过很小的努力，就实现了对电影数据的RESTful API访问。

已经创建的API包括：

- `GET /films`：按页列出所有的电影
- `HEAD /films`：获取电影列表的概要信息
- `POST /films`：创建一个新的电影
- `GET /films/5`：获取电影5的详情
- `HEAD /films/5`：获取电影5的概要信息
- `PATCH /films/5 and PUT /films/5`：更新电影5
- `DELETE /films/5`：删除电影5
- `OPTIONS /films`：显示`/films`支持的操作
- `OPTIONS /films/5`：显示`/films/5`支持的操作

之所以会有这些功能是因为`\yii\rest\ActiveController`支持如下动作：

- `index`：列出模型
- `view`：返回模型的细节
- `create`：创建一个新的模型
- `update`：更新一个已有的模型
- `delete`：删除一个已有的模型
- `options`：返回允许的HTTP方法

`verbs()`方法为每一个动作定义了允许的请求方法。

为了检查我们的RESTful API是否工作正常，我们来发送几个请求。

首先使用`GET`请求。在控制台中运行如下命令：

```
curl -i -H "Accept:application/json" "http://yii-book.app/films"
```

你将会得到如下输出：

```
HTTP/1.1 200 OK
Date: Wed, 23 Sep 2015 17:46:35 GMT
Server: Apache
X-Powered-By: PHP/5.5.23
X-Pagination-Total-Count: 5
X-Pagination-Page-Count: 1
X-Pagination-Current-Page: 1
X-Pagination-Per-Page: 20
Link: <http://yii-book.app/films?page=1>; rel=self
Content-Length: 301
Content-Type: application/json; charset=UTF-8
[{"id":1,"title":"Interstellar","release_year":2014},{"id":2,"title":
"Harry Potter and the Philosopher's
Stone","release_year":2001},{"id":3,"title":"Back to the
Future","release_year":1985},{"id":4,"title":"Blade
Runner","release_year":1982},{"id":5,"title":"Dallas Buyers
Club","release_year":2013}]
```

发送一个`POST`请求。在控制台中运行如下命令：

```
curl -i -H "Accept:application/json" -X POST -d title="New film" -d release_year=2015 "http://yii-book.app/films"
```

你将会得到如下输出：

```
HTTP/1.1 201 Created
Date: Wed, 23 Sep 2015 17:48:06 GMT
Server: Apache
X-Powered-By: PHP/5.5.23
Location: http://yii-book.app/films/6
Content-Length: 49
Content-Type: application/json; charset=UTF-8
{"title":"New film","release_year":"2015","id":6}
```

获取创建好的电影。在控制台中运行如下命令：

```
curl -i -H "Accept:application/json" "http://yii-book.app/films/6"
```

你将会得到如下输出：

```
HTTP/1.1 200 OK
Date: Wed, 23 Sep 2015 17:48:36 GMT
Server: Apache
X-Powered-By: PHP/5.5.23
Content-Length: 47
Content-Type: application/json; charset=UTF-8
{"id":6,"title":"New film","release_year":2015}
```

发送一个`DELETE`请求。在控制台中运行如下命令：

```
curl -i -H "Accept:application/json" -X DELETE "http://yii-book.app/films/6"
```

你将会得到如下输出：

```
HTTP/1.1 204 No Content
Date: Wed, 23 Sep 2015 17:48:55 GMT
Server: Apache
X-Powered-By: PHP/5.5.23
Content-Length: 0
Content-Type: application/json; charset=UTF-8
```

### 更多...

接下来我们看下内容negotiation和自定义Rest URL规则：

#### Content negotiation

你也可以很容易使用内容negotiation行为格式化你的响应：

例如，你可以将这个代码放入到你的控制器中，这样所有的数据都会以XML格式返回：

你应该看下文档中所有的格式列表。

```
use yii\web\Response;
public function behaviors()
{
    $behaviors = parent::behaviors();
    $behaviors['contentNegotiator']['formats']['application/xml']=Response::FORMAT_XML;
    return $behaviors;
}
```

在控制台中运行：

```
curl -i -H "Accept:application/xml" "http://yii-book.app/films"
```

你将会得到如下输出：

```
HTTP/1.1 200 OK
Date: Wed, 23 Sep 2015 18:02:47 GMT
Server: Apache
X-Powered-By: PHP/5.5.23
X-Pagination-Total-Count: 5
X-Pagination-Page-Count: 1
X-Pagination-Current-Page: 1
X-Pagination-Per-Page: 20
Link: <http://yii-book.app/films?page=1>; rel=self
Content-Length: 516
Content-Type: application/xml; charset=UTF-8
<?xml version="1.0" encoding="UTF-8"?>
<response>
    <item>
        <id>1</id>
        <title>Interstellar</title>
        <release_year>2014
        </release_year>
    </item>
    <item>
        <id>2</id>
        <title>Harry Potter and the Philosopher's Stone</title>
        <release_year>2001
        </release_year>
    </item>
    <item>
        <id>3</id>
        <title>Back to the Future</title>
        <release_year>1985
        </release_year>
    </item>
    <item>
        <id>4</id>
        <title>Blade Runner</title>
        <release_year>1982
        </release_year>
    </item>
    <item>
        <id>5</id>
        <title>Dallas Buyers Club</title>
        <release_year>2013
        </release_year>
    </item>
</response>
```

#### 自定义Rest URL规则

默认情况下，你必须记住一个以复杂格式定义的控制器ID。这是因为`yii\rest\UrlRule`自动复杂化控制器ID。你可以很容易的通过设置`yii\rest\UrlRule::$pluralize`的值为false，来关掉它：

```
'urlManager' => [
    //..
    'rules' => [
        [
            'class' => 'yii\rest\UrlRule',
            'controller' => 'film'
            'pluralize' => false
        ],
    ],
    //..
]
```

如果你希望控制器按照指定的格式出现，你可以添加一个自定义名称到键值对，其中键是控制器ID，值是真正的控制器ID，例如：

```
'urlManager' => [
    //..
    'rules' => [
        [
            'class' => 'yii\rest\UrlRule',
            'controller' => ['super-films' => 'film']
        ],
    ],
    //..
]
```

### 参考

欲了解更多信息，参考如下地址：

- [http://www.yiiframework.com/doc-2.0/guide-rest-quick-start.html](http://www.yiiframework.com/doc-2.0/guide-rest-quick-start.html)
- [http://www.yiiframework.com/doc-2.0/yii-rest-urlrule.html](http://www.yiiframework.com/doc-2.0/yii-rest-urlrule.html)
- [http://www.yiiframework.com/doc-2.0/guide-rest-response-formatting.html](http://www.yiiframework.com/doc-2.0/guide-rest-response-formatting.html)
- [http://budiirawan.com/setup-restful-api-yii2/](http://budiirawan.com/setup-restful-api-yii2/)

## 身份校验

在本小节中我们将会设置身份校验模型。

### 准备

重复*创建一个REST服务器*小节中*准备*和*如何做*的所有步骤。

### 如何做...

1. 修改`@app/controllers/FilmController`：

```
<?php
namespace app\controllers;
use app\models\User;
use Yii;
use yii\helpers\ArrayHelper;
use yii\rest\ActiveController;
use yii\filters\auth\HttpBasicAuth;
class FilmController extends ActiveController
{
    public $modelClass = 'app\models\Film';
    public function behaviors()
    {
        return ArrayHelper::merge(parent::behaviors(),[
            'authenticator' => [
                'authMethods' => [
                    'basicAuth' => [
                        'class' =>HttpBasicAuth::className(),
                        'auth' => function ($username,$password) {
                            $user =User::findByUsername($username);
                            if ($user !== null && $user->validatePassword($password)){
                                return $user;
                            }
                            return null;
                        },
                    ]
                ]
            ]
        ]);
    }
}
```

在浏览器中打开`http://yii-book.app/films`，确保我们配置了HTTP基本身份验证：

![](../images/601.png)

尝试身份验证。在控制台中运行如下命令：

```
curl -i -H "Accept:application/json" "http://yii-book.app/films"
```

你将会得到如下结果：

```
HTTP/1.1 401 Unauthorized
Date: Thu, 24 Sep 2015 01:01:24 GMT
Server: Apache
X-Powered-By: PHP/5.5.23
Www-Authenticate: Basic realm="api"
Content-Length: 149
Content-Type: application/json; charset=UTF-8
{"name":"Unauthorized","message":"You are requesting with an invalid credential.","code":0,"status":401,"type":"yii\\web\\UnauthorizedHttp
Exception"}
```

1. 现在尝试使用`cURL`进行`auth`：

```
curl -i -H "Accept:application/json" -u admin:admin "http://yii-book.app/films"
```

2. 你将会得到类似如下结果：

```
HTTP/1.1 200 OK
Date: Thu, 24 Sep 2015 01:01:40 GMT
Server: Apache
X-Powered-By: PHP/5.5.23
Set-Cookie: PHPSESSID=8b3726040bf8850ebd07209090333103; path=/;
HttpOnly
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate,
post-check=0, pre-check=0
Pragma: no-cache
X-Pagination-Total-Count: 5
X-Pagination-Page-Count: 1
X-Pagination-Current-Page: 1
X-Pagination-Per-Page: 20
Link: <http://yii-book.app/films?page=1>; rel=self
Content-Length: 301
Content-Type: application/json; charset=UTF-8
[{"id":1,"title":"Interstellar","release_year":2014},{"id":2,"ti
tle":"Harry Potter and the Philosopher's
Stone","release_year":2001},{"id":3,"title":"Back to the
Future","release_year":1985},{"id":4,"title":"Blade
Runner","release_year":1982},{"id":5,"title":"Dallas Buyers
Club","release_year":2013}]
```

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





