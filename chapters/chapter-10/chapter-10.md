# 第十章 部署

在本章中，我们将会讨论如下小节：

- 修改Yii目录的布局
- 修改应用的webroot
- 修改一个高级应用模板
- 移动配置文件到单独的文件中
- 使用多个配置来简化部署
- 实施和执行cron任务
- 维护模式
- 部署工具

## 介绍

在本章中，我们将会讨论多个建议，他们对应用部署特别有用；当以小组开发一个应用，或者你只是希望你的开发环境更舒适，这些建议就可以派上用场。

## 修改Yii目录布局

默认情况下，我们有基础和高级Yii2应用框架，他们有不同的目录结构。但是这些框架不是教条，如果有需要我们可以自定义他们。

例如，我们可以将runtime文件夹移除项目。

### 准备

按照官方指南[http://www.yiiframework.com/doc-2.0/guide-start-installation.html](http://www.yiiframework.com/doc-2.0/guide-start-installation.html)的描述，使用Composer包管理器创建一个新的`yii2-app-basic`应用。

### 如何做...

#### 修改runtime文件夹的位置

打开`config/web.php`和`config/console.php`，定义`runtimePath`参数：

```
$config = [
    'id' => 'basic',
    'basePath' => dirname(__DIR__),
    'bootstrap' => ['log'],
    'runtimePath' => '/tmp/runtime',
    'components' => [
    // ...
    ],
]
```

将runtime文件夹移到新的位置。

#### 修改vendor文件夹的位置

1. 打开`config/web.php`和`console.php`，定义`vendorPath`参数：

```
$config = [
    'id' => 'basic',
    'basePath' => dirname(__DIR__),
    'bootstrap' => ['log'],
    'vendorPath' => dirname(__DIR__), '/../vendor,
    'components' => [
        // ...
    ],
]
```

2. 将`vendor`文件夹以及`composer.json`和`composer.lock`文件移到新的位置。
3. 打开`web/index.php`和`yii`文件，找到这些行：

```
require(__DIR__ . '/../vendor/autoload.php');
require(__DIR__ . '/../vendor/yiisoft/yii2/Yii.php');
```

4. 修改包含的路径。

#### 修改控制器的位置

1. 重命名`commands`目录为`console`。
2. 修改命名空间`app\commands\HelloController`为`app\console\HelloController`。
3. 打开`config/console.php`，重新定义`controllerNamespace`参数：

```
$config = [
    'id' => 'basic-console',
    'basePath' => dirname(__DIR__),
    'bootstrap' => ['log'],
    'controllerNamespace' => 'app\console,
    'components' => [
        // ...
    ],
]
```

#### 修改视图文件夹的位置

1. 打开`config/web.php`，定义`viewPath`参数：

```
$config = [
    'id' => 'basic',
    'basePath' => dirname(__DIR__),
    'bootstrap' => ['log'],
    'viewPath' => '@app/myviews',
    'components' => [
        // ...
    ],
]
```

2. 重命名`views`目录。

### 工作原理...

在`yii\base\Application::preInit`方法中，我们的应用定义了`basePath`、`runtimePath`和`vendorPath`参数。

默认情况下，这些值分别指向了应用目录的根，`runtime`和`vendor`路径。

例如，你可以重定义`vendorPath`，如果你希望将vendor目录分享给同样项目的一些实例。但是注意包的版本的兼容性。

`yii\base\Application`类继承了`yii\base\Module`，它包含了`controllerNamespace`和`viewPath`参数。第一个参数允许你修改应用和模块的基命名空间。如果你希望在同一个模块目录中，提供前端和后端控制器，这会非常有帮助。只修改`controllers`目录到前端和后端，或者创建子目录并配置你的前端和后端应用：

```
return [
    'id' => 'app-frontend',
    'basePath' => dirname(__DIR__),
    'controllerNamespace' => 'frontend\controllers',
    'bootstrap' => ['log'],
    'modules' => [
        'user' => [
            'my\user\Module',
            'controllerNamespace' => 'my\user\controllers\frontend',
        ]
    ],
// ...
]
return [
    'id' => 'app-backend',
    'basePath' => dirname(__DIR__),
    'controllerNamespace' => 'backend\controllers',
    'bootstrap' => ['log'],
    'modules' => [
        'user' => [
            'my\user\Module',
            'controllerNamespace' => 'my\user\controllers\backend',
        ]
    ],
// ...
]
```

### 参考

为了了解更多关于应用结构的信息，参考[http://www.yiiframework.com/doc-2.0/guide-structure-applications.html](http://www.yiiframework.com/doc-2.0/guide-structure-applications.html)。

## 移动一个应用webroot

默认情况下，Yii2应用使用`web`文件夹存放你网站的入口脚本。但是共享的托管环境通常会对配置和目录结构有限制。你不能修改你网站的工作目录。大部分服务器只提供`public_html`目录存放你网站的入口脚本。

### 准备

按照官方指南[http://www.yiiframework.com/doc-2.0/guide-start-installation.html](http://www.yiiframework.com/doc-2.0/guide-start-installation.html)的描述，使用Composer包管理器创建一个新的`yii2-app-basic`应用。

### 如何做...

下边我们来讨论如何修改一个应用的webroot。

#### 在root中存放文件

1. 上传应用文件到你的托管服务器上
2. 重命名`web`目录为`public_html`
3. 检查网站是否工作正常

#### 将文件放在一个子文件夹中

一个托管服务器用户目录可能包含其它文件和文件夹。遵照下面的步骤，你可以将文件移动到子文件夹中：

1. 创建`application`和`public_html`文件夹
2. 将应用文件移动到`application`文件夹中
3. 将`application/web`文件夹中的内容移动到`public_html`文件夹中
4. 打开`public_html/index.php`文件，并修改包含的路径：

```
require(__DIR__ . '/../application/vendor/autoload.php');
require(__DIR__ . '/../application/vendor/yiisoft/yii2/Yii.php');
```

### 工作原理...

Yii2应用基于入口脚本的位置自动设置`@web`和`@webroot` alias路径。因此我们可以很容易移动或者重命名一个`web`目录，并且不需要修改应用的配置。

对于`yii2-app-advanced`，你可以移动`backend`中`web`目录中的内容到一个子文件夹中，例如`admin`：

```
public_html
    index.php
        ...
    admin
        index.php
        ...
backend
common
console
frontend
...
```

### 参考

欲了解更多关于在一个共享托管环境上安装Yii，参考[http://www.yiiframework.com/doc-2.0/guide-tutorial-shared-hosting.html](http://www.yiiframework.com/doc-2.0/guide-tutorial-shared-hosting.html)。

## 修改一个高级应用模板

默认情况下，Yii2的高级模板有`console`，`frontend`和`backend`应用。但是，在你的特殊情况下，你可以重命名已有的一个，创建你自己的应用。例如，如果你在为你的网站开发API，你可以添加`api`应用。

### 准备

按照官方指南[http://www.yiiframework.com/doc-2.0/guide-start-installation.html](http://www.yiiframework.com/doc-2.0/guide-start-installation.html)的描述，使用Composer包管理器创建一个新的`yii2-app-advanced`应用。

### 如何做...

1. 在应用的根目录下，复制`backend`文件夹中的内容到一个新的`api`文件夹中。
2. 打开`api/config/main.php`文件，修改`controllerNamespace`的值：

```
return [
    'id' => 'app-manager',
    'basePath' => dirname(__DIR__),
    'controllerNamespace' => 'api\controllers',
    // ....
]
```

3. 打开`api/assets/AppAsset.php`和`api/controllers/SiteController.php`，将命名空间从`backend`修改为`api`：

```
namespaces api\assets;
namespaces api\controllers;
```

4. 打开`api/views/layouts/main.php`文件，找到如下行：

```
use backend\assets\AppAsset;
```

修改为：

```
use api\assets\AppAsset;
```

5. 打开`common/config/bootstrap.php`，为新的应用添加`@api` alias：

```
<?php
Yii::setAlias('@common', dirname(__DIR__));
Yii::setAlias('@frontend', dirname(dirname(__DIR__)) .'/frontend');
Yii::setAlias('@backend', dirname(dirname(__DIR__)) .'/backend');
Yii::setAlias('@console', dirname(dirname(__DIR__)) .'/console');
Yii::setAlias('@api', dirname(dirname(__DIR__)) . '/api);
```

6. 打开`environments`目录，在`dev`和`prod`子文件夹中，拷贝`backend`文件夹为`api`。
7. 打开`environments/index.php`文件，为`api`应用添加如下行：

```
return [
    'Development' => [
        'path' => 'dev',
        'setWritable' => [
            'backend/runtime',
            'backend/web/assets',
            'frontend/runtime',
            'frontend/web/assets',
            'api/runtime',
            'api/web/assets',
        ],
        'setExecutable' => [
            'yii',
            'tests/codeception/bin/yii',
        ],
        'setCookieValidationKey' => [
            'backend/config/main-local.php',
            'frontend/config/main-local.php',
            'api/config/main-local.php',
        ],
    ],
    'Production' => [
        'path' => 'prod',
        'setWritable' => [
            'backend/runtime',
            'backend/web/assets',
            'frontend/runtime',
            'frontend/web/assets',
            'api/runtime',
            'api/web/assets',
        ],
        'setExecutable' => [
            'yii',
        ],
        'setCookieValidationKey' => [
            'backend/config/main-local.php',
            'frontend/config/main-local.php',
            'api/config/main-local.php',
        ],
    ],
];
```

现在，你就有了`console`、`frontend`、`backend`和`api`应用。

### 工作原理...

高级应用模板，是一组带有自定义aliases的应用集合，例如`@frontend`，`@backend`，`@common`，`@console`，以及相对应的命名空间。而对于`Basic`模板只有一个简单的`@app` alias。

如果有需要，你可以很容易的添加、删除或者重命名这些应用（以及他们的aliases和命名空间）。

### 参考

欲了解更多关于应用目录结构使用的信息，参考[https://github.com/yiisoft/yii2-app-advanced/tree/master/docs/guide](https://github.com/yiisoft/yii2-app-advanced/tree/master/docs/guide)

## 将配置部分移到单独的文件中

在基础应用模板中，我们分离了web和console配置文件。通常情况下，我们可以在两个配置文件中设置一些应用组件。

而且，当我们开发一个大型应用时，我们可能面临一些不方便的问题。例如，如果我们需要调整一些设置，我们很可能需要在web应用配置和console应用配置中做多次重复修改。

### 准备

按照官方指南[http://www.yiiframework.com/doc-2.0/guide-start-installation.html](http://www.yiiframework.com/doc-2.0/guide-start-installation.html)的描述，使用Composer包管理器创建一个新的`yii2-app-basic`应用。

### 如何做...

1. 打开`config/web.php`文件，添加`urlManager`部分到组件配置中：

```
'components' => [
    // ...
    'db' => require(__DIR__ . '/db.php'),
    'urlManager' => [
        'class' => 'yii\web\UrlManager',
        'enablePrettyUrl' => true,
        'showScriptName' => false,
        'rules' => [
            '' => 'site/index',
            '<_c:[\w\-]+>/<id:\d+>' => '<_c>/view',
            '<_c:[\w\-]+/<_a:[\w\-]+>>/<id:\d+>' => '<_c>/<_a>',
            '<_c:[\w\-]+>' => '<_c>/index',
        ],
    ],
],
```

2. 创建`config/urlRules.php`文件，并将规则数组移到这里边：

```
<?php
return [
    '' => 'site/index',
    '<_c:[\w\-]+>/<id:\d+>' => '<_c>/view',
    '<_c:[\w\-]+/<_a:[\w\-]+>>/<id:\d+>' => '<_c>/<_a>',
    '<_c:[\w\-]+>' => '<_c>/index',
];
```

3. 使用这个文件替换这个规则数组：

```
'urlManager' => [
    'class' => 'yii\web\UrlManager',
    'enablePrettyUrl' => true,
    'showScriptName' => false,
    'rules' => require(__DIR__ . '/urlRules.php'),
],
```

### 工作原理...

先前的技术依赖于这样的事实，Yii配置文件是原生的PHP文件，里边有一个数组：

```
<?php return [...];
```

我们看`require`结构：

```
'rules' => require(__DIR__ . '/urlRules.php'),
```

当我们使用这个的时候，它会读取指定的文件，并且，如果文件中有一个`return`语句，它会返回一个值。

因此，将主配置文件中的一部分移到一个单独的文件中，需要创建一个新的文件，将配置的一部分移到`return`语句的后边，并在主配置文件中使用`require`。

它分离了应用需要的一些常用配置部分（在我们的例子中，有web应用和console应用），我们可以使用`require`将他们移到一个单独的文件中。

### 参考

欲了解更多关于`require`和`include`语句，参考如下URL：

- [http://php.net/manual/en/function.require.php](http://php.net/manual/en/function.require.php)
- [http://php.net/manual/en/function.include.php](http://php.net/manual/en/function.include.php)

## 使用多个配置来简化部署

高级应用模板为它的每一个应用使用不同的配置文件。

```
common
    config
        main.php
        main-local.php
        params.php
        params-local.php
console
    config
        main.php
        main-local.php
        params.php
        params-local.php
backend
    config
        main.php
        main-local.php
        params.php
        params-local.php
frontend
    config
        main.php
        main-local.php
        params.php
        params-local.php
```

每一个入口`web/index.php`脚本合并自己配置文件的集合：

```
$config = yii\helpers\ArrayHelper::merge(
    require(__DIR__ . '/../../common/config/main.php'),
    require(__DIR__ . '/../../common/config/main-local.php'),
    require(__DIR__ . '/../config/main.php'),
    require(__DIR__ . '/../config/main-local.php')
);
$application = new yii\web\Application($config);
$application->run();
```

每一个`config/main.php`文件和并参数：

```
<?php
$params = array_merge(
    require(__DIR__ . '/../../common/config/params.php'),
    require(__DIR__ . '/../../common/config/params-local.php'),
    require(__DIR__ . '/params.php'),
    require(__DIR__ . '/params-local.php')
);
return [
    // ...
    'params' => $params,
];
```

这个系统允许你配置应用中常用和特殊应用属性和组件。并且我们可以基于版本控制系统，存储缺省配置文件，并忽略所有的`*-local.php`文件。

所有的本地文件模板在`environments`文件夹中准备，当你在控制台中运行`php init`，并选择一个needle环境，这个初始脚本复制相应的文件，并将它们放在目标文件夹中。

但是基础应用模板不包含敏捷配置系统，只提供如下文件：

```
config
    console.php
    web.php
    db.php
    params.php
```

尝试添加一个高级配置系统到`yii2-app-basic`应用模板中。

### 准备

按照官方指南[http://www.yiiframework.com/doc-2.0/guide-start-installation.html](http://www.yiiframework.com/doc-2.0/guide-start-installation.html)的描述，使用Composer包管理器创建一个新的`yii2-app-basic`应用。

### 如何做...

1. 创建`config/common.php`文件：

```
<?php
$params = array_merge(
    require(__DIR__ . '/params.php'),
    require(__DIR__ . '/params-local.php')
);
return [
    'basePath' => dirname(__DIR__),
    'components' => [
        'cache' => [
            'class' => 'yii\caching\FileCache',
        ],
        'mailer' => [
            'class' => 'yii\swiftmailer\Mailer',
        ],
        'db' => [],
    ],
    'params' => $params,
];
```

2. 创建`config/common-local`文件：

```
<?php
return [
    'components' => [
        'db' => [
            'class' => 'yii\db\Connection',
            'dsn' => 'mysql:host=localhost;dbname=yii2basic',
            'username' => 'root',
            'password' => '',
            'charset' => 'utf8',
        ],
        'mailer' => [
            'useFileTransport' => true,
        ],
    ],
];
```

3. 移除`config/db.php`文件：
4. 从`config/console.php`移除重复的代码：

```
<?php
Yii::setAlias('@tests', dirname(__DIR__) . '/tests');
return [
    'id' => 'basic-console',
    'bootstrap' => ['log', 'gii'],
    'controllerNamespace' => 'app\commands',
    'modules' => [
        'gii' => 'yii\gii\Module',
    ],
    'components' => [
        'log' => [
            'targets' => [
                [
                    'class' => 'yii\log\FileTarget',
                    'levels' => ['error', 'warning'],
                ],
            ],
        ],
    ],
];
```

5. 创建一个带有空数组的文件`config/console-local.php`：

```
<?php return [];
```

6. 修改`config/web.php`文件：

```
$config = [
    'id' => 'basic',
    'bootstrap' => ['log'],
    'components' => [
        'user' => [
            'identityClass' => 'app\models\User',
            'enableAutoLogin' => true,
        ],
        'errorHandler' => [
            'errorAction' => 'site/error',
        ],
        'log' => [
            'traceLevel' => YII_DEBUG ? 3 : 0,
            'targets' => [
                [
                    'class' => 'yii\log\FileTarget',
                    'levels' => ['error', 'warning'],
                ],
            ],
        ],
    ],
];
if (YII_ENV_DEV) {
// configuration adjustments for 'dev' environment
    $config['bootstrap'][] = 'debug';
    $config['modules']['debug'] = 'yii\debug\Module';
    $config['bootstrap'][] = 'gii';
    $config['modules']['gii'] = 'yii\gii\Module';
}
return $config;
```

7. 将`request`配置移动到`config/web-local.php`文件中：

```
<?php
return [
    'components' => [
        'request' => [
            'cookieValidationKey' => 'TRk9G1La5kvLFwqMEQTp6PmC1NHdjtkq',
        ],
    ],
];
```

8. 从`config/params.php`文件中移除电子邮件ID：

```
<?php
return [
    'adminEmail' => '',
];
```

9. 粘贴ID到`config/params-local.php`文件：

```
<?php
return [
    'adminEmail' => 'admin@example.com',
];
```

10. 从`tests/codeception/config/config.php`移除`dsn`字符串：

```
<?php
/**
 * Application configuration shared by all test types
 */
return [
    'controllerMap' => [
// ...
    ],
    'components' => [
        'db' => [
            'dsn' => '',
        ],
        'mailer' => [
            'useFileTransport' => true,
        ],
        'urlManager' => [
            'showScriptName' => true,
        ],
    ],
];
```

11. 将字符串放到一个新文件中`tests/codeception/config/config-local.php`：

```
<?php
return [
    'components' => [
        'db' => [
            'dsn' => 'mysql:host=localhost;dbname=yii2_basic_tests',
        ],
    ],
];
```

12. 在文件`web/index.php`中添加配置合并：

```
$config = yii\helpers\ArrayHelper::merge(
    require(__DIR__ . '/../config/common.php'),
    require(__DIR__ . '/../config/common-local.php'),
    require(__DIR__ . '/../config/web.php'),
    require(__DIR__ . '/../config/web-local.php')
);
```

13. 添加配置合并到终端入口脚本，`yii`：

```
$config = yii\helpers\ArrayHelper::merge(
    require(__DIR__ . '/config/common.php'),
    require(__DIR__ . '/config/common-local.php'),
    require(__DIR__ . '/config/console.php'),
    require(__DIR__ . '/config/console-local.php')
);
```

14. 添加配置合并到`tests/codeception/config`中含有单元、功能、验收测试的测试配置中：

```
return yii\helpers\ArrayHelper::merge(
    require(__DIR__ . '/../../../config/common.php'),
    require(__DIR__ . '/../../../config/common-local.php'),
    require(__DIR__ . '/../../../config/web.php'),
    require(__DIR__ . '/../../../config/web-local.php'),
    require(__DIR__ . '/config.php'),
    require(__DIR__ . '/config-local.php'),
    [
        // ...
    ]
);
```

15. 添加配置合并到测试环境控制台的入口脚本，`tests/codeception/bin/yii`：

```
$config = yii\helpers\ArrayHelper::merge(
    require(YII_APP_BASE_PATH . '/config/common.php'),
    require(YII_APP_BASE_PATH . '/config/common-local.php'),
    require(YII_APP_BASE_PATH . '/config/console.php'),
    require(YII_APP_BASE_PATH . '/config/console-local.php'),
    require(__DIR__ . '/../config/config.php'),
    require(__DIR__ . '/../config/config-local.php')
);
```

16. 结果是，你可以在你的配置文件夹下，获得如下内容：

```
config
    common.php
    common-local.php
    console.php
    console-local.php
    web.php
    web-local.php
    params.php
    params-local.php
```

17. 最终，你可以添加一个新的`.gitignore`文件到你的`config`和`tests/codeception/config`文件夹下，所以你可以通过版本控制系统忽略本地配置文件：

```
/*-local.php
```

### 工作原理...

你可以在`config/common.php`文件中存储常见的应用组件配置，同时也可以为web和控制台应用设置指定的配置。你可以将你的临时和安全配置数据放在`*-local.php`文件中。

此外，你可以从`yii2-app-advanced`中复制初始化shell脚本。

1. 创建一个新的`environments`目录，并复制你的模板到里边：

```
environments
    dev
        config
            common-local.php
            console-local.php
            web-local.php
            params-local.php
        web
            index.php
            index-test.php
        tests
            codeception
                config
                    config.php
                    config-local.php
        yii
    prod
        config
            common-local.php
            console-local.php
            web-local.php
            params-local.php
        web
            index.php
        yii
```

2. 创建`environments/index.php`文件：

```
<?php
return [
    'Development' => [
        'path' => 'dev',
        'setWritable' => [
            'runtime',
            'web/assets',
        ],
        'setExecutable' => [
            'yii',
            'tests/codeception/bin/yii',
        ],
        'setCookieValidationKey' => [
            'config/web-local.php',
        ],
    ],
    'Production' => [
        'path' => 'prod',
        'setWritable' => [
            'runtime',
            'web/assets',
        ],
        'setExecutable' => [
            'yii',
        ],
        'setCookieValidationKey' => [
            'config/web-local.php',
        ],
    ],
];
```

3. 从你的`composer.json`中移除默认的`Installer::postCreateProject`配置：

```
"extra": {
    "asset-installer-paths": {
        "npm-asset-library": "vendor/npm",
        "bower-asset-library": "vendor/bower"
    }
}
```

4. 从高级模板[https://github.com/yiisoft/yii2-app-advanced](https://github.com/yiisoft/yii2-app-advanced)拷贝`init`和`init.bat`脚本，你可以使用`php init`运行初始化过程，从repository中克隆项目。

### 参考

欲了解更多关于应用配置的信息，参考[http://www.yiiframework.com/doc-2.0/guide-concept-configurations.html](http://www.yiiframework.com/doc-2.0/guide-concept-configurations.html)。

## 实施和执行cron任务

有时，一个应用需要一些后台任务，例如重新生成一个站点地图，或者刷新统计数据。一种常见的方式是使用cron任务。当使用Yii时，有一种方法可以使用一个命令做为任务来运行。

在这个小节中，我们将会看到如何同时实现。在我们的小节中，我们将会实现写当前时间戳到受保护文件夹中的`timestamp.txt`文件中。

### 准备

按照官方指南[http://www.yiiframework.com/doc-2.0/guide-start-installation.html](http://www.yiiframework.com/doc-2.0/guide-start-installation.html)的描述，使用Composer包管理器创建一个新的`yii2-app-basic`应用。

### 如何做...

#### 运行Hello命令

尝试作为shell命令运行`app\commands\HelloController::actionIndex`：

```
<?php
namespace app\commands;
use yii\console\Controller;
/**
* This command echoes the first argument that you have entered.
*/
class HelloController extends Controller
{
    /**
    * This command echoes what you have entered as the message.
    * @param string $message the message to be echoed.
    */
    public function actionIndex($message = 'hello world')
    {
        echo $message . "\n";
    }
}
```

1. 在你的应用目录中，打开shell，并执行如下命令：

```
php yii
```

此外，你也可以调用如下，确保shell可以工作：

```
./yii
```

2. 输入如下命令，展示`hello`：

```
./yii help hello
```

2. 这个框架可以展示一些信息：

```
DESCRIPTION
This command echoes what you have entered as the message.
USAGE
yii hello [message] [...options...]
- message: string (defaults to 'hello world')
the message to be echoed.
```

4. 运行缺省命令动作：

```
./yii hello
```

或者，运行指定的`index`动作：

```
./yii hello/index
```

5. 你可以看到默认提示：

```
Hello world
```

6. 运行带有任何参数的命令，将会看到响应：

```
./yii hello 'Bond, James Bond'
```

#### 创建你自己的命令

你也可以创建你自己的控制台控制器。例如，创建一个`commands/CronController.php`文件：

```
<?php
namespace app\commands;
use yii\console\Controller;
use yii\helpers\Console;
use Yii;
/**
 * Console crontab actions
 */
class CronController extends Controller
{
    /**
     * Regenerates timestamp
     */
    public function actionTimestamp()
    {
        file_put_contents(Yii::getAlias('@app/timestamp.txt'),
            time());
        $this->stdout('Done!', Console::FG_GREEN, Console::BOLD);
        $this->stdout(PHP_EOL);
    }
}
```

这些完成以后，在控制台中运行命令：

```
./yii cron/timestamp
```

然后，检查响应文本，以及生成的新文件`timestamp.txt`。

#### 设置cron任务

在你的Linux服务器上创建`/etc/cron.d/myapp`，并添加如下内容，让我们的脚本在每天的半夜12点整运行一次：

```
0 0 * * * www-data /path/to/yii cron/timestamp >/dev/null
```

### 工作原理...

一个控制台命令被定义成了一个控制器类，这个类继承了`yii\console\Controller`。在控制器类中，你可以定义一个或多个动作，分别对应这个控制器的多个子命令。在每一个动作中，你可以为每一个指定的子命令实现恰当的任务。

在运行一个命令时，你需要指定控制器动作的路由。例如，`migrate/create`调用的子命令对应于`MigrateController::actionCreate()`动作函数。如果在执行时，提供的路由不包含一个动作ID，默认的动作将会被执行（作为一个web控制器）。

注意你的控制台控制器被放置在指定的文件夹中，位置由`web/console.php`中的`controllerNamespace`选项定义。

### 参考

- 欲了解更多关于Yii2控制台命令的信息，参考[http://www.yiiframework.com/doc-2.0/guide-tutorial-console.html](http://www.yiiframework.com/doc-2.0/guide-tutorial-console.html)
- 为了了解更多关于Cron daemon的信息，参考[https://en.wikipedia.org/wiki/Cron](https://en.wikipedia.org/wiki/Cron)
- *修改你的Yii目录layout*小节中的`controllerNamespace`

## 维护模式

有时，需要微调一些应用的设置，或者从一个备份中恢复数据库。当处理这些任务时，你不希望允许每一个人使用应用，因为它可能会导致丢失最新的用户消息，或者展示应用实现的细节。

在这个小节中，我们将会看到如何向除了开发者以外的每一个人展示一条维护消息。

### 准备

按照官方指南[http://www.yiiframework.com/doc-2.0/guide-start-installation.html](http://www.yiiframework.com/doc-2.0/guide-start-installation.html)的描述，使用Composer包管理器创建一个新的`yii2-app-basic`应用。

### 如何做...

执行如下步骤：

1. 首先，我们需要创建`protected/controllers/MaintenanceController.php`。我们按如下方式做：

```
class MaintenanceController extends Controller
{
    public function actionIndex()
    {
        $this->renderPartial("index");
    }
}
```

2. 然后，我们创建一个名为`views/maintenance/index.php`的视图，如下所示：

```
<?php
use yii\helpers\Html;
?>
<!doctype html>
<head>
    <meta charset="utf-8" />
    <title><?php echo
        Html::encode(Yii::$app->name)?>is under maintenance</title>
</head>
<body>
<h1><?php echo CHtml::encode(Yii::$app->name)?>is under
    maintenance</h1>
<p>We'll be back soon. If we aren't back for too
    long,please drop a message to <?php echo
    Yii::$app->params['adminEmail']?>.</p>
<p>Meanwhile, it's a good time to get a cup of coffee,to read a book or to check email.</p>
</body>
```

3. 现在，我们需要添加一行代码到`config/web.php`，如下所示：

```
$config = [
    'catchAll' => file_exists(dirname(__DIR__).'/.maintenance') && !(isset($_COOKIE['secret']) && $_COOKIE['secret']=="password") ? ['maintenance/index'] : null,
    // …
]
```

4. 现在为了前往维护模式，你需要在你的网站目录下创建一个名为`.maintenance`的文件。做完这一步后，你将会看到这个页面。

为了回到正常模式，你只需要删除这个文件。为了查看网站的维护模式，你可以创建一个名为`secret`的cookie，它的值是等于`password`。

### 工作原理...

一个Yii web应用提供了一种方式，可以拦截所有可能的请求，并这它们重定向到一个指定的控制器动作上。你可以通过设置`yii\web\Application::catchAll`为一个包含应用路由的数组来做到它：

```
'catchAll' => ['maintenance/index'],
```

这个维护控制器本身并不特别；它只是渲染一个带有文字的视图。

我们需要一种简单的方式，来打开或者关闭维护模式。因为应用配置文件是一个特殊的PHP文件，我们可以使用一个简单的检查，查看制定文件是否存在，来做到这些。如下所示：

```
file_exists(dirname(__DIR__) . '/.maintenance')
```

此外，我们检查cookie值来能覆盖维护模式。如下所示：

```
!(isset($_COOKIE['secret']) && $_COOKIE['secret']=="password")
```

### 参考

为了了解更多关于在Yii应用中如何捕获所有请求，以及为production ready solution用于维护，参考[http://www.yiiframework.com/doc-2.0/yii-webapplication.html#$catchAll-detail](http://www.yiiframework.com/doc-2.0/yii-webapplication.html#$catchAll-detail)。

## 部署工具

如果你为你的项目代码在使用一个版本控制系统，例如Git，将发布包推到远程库，你可以使用Git中的`git pull`命令来部署代码到你的生产服务器上，而不用手动上传文件。此外，你可以给自己写一个shell脚本来拉取新的库提交，更新vendors，应用migration等等。

但是，有很多工具可以用来做自动化部署。在本小节中，我们来看一些名叫Deployer的工具。

### 准备

按照官方指南[http://www.yiiframework.com/doc-2.0/guide-start-installation.html](http://www.yiiframework.com/doc-2.0/guide-start-installation.html)的描述，使用Composer包管理器创建一个新的`yii2-app-basic`应用。

### 如何做...

如果你有一个共享的远程库，你可以使用它用来部署源。

#### 第一步：准备远程host

1. 到你的远程host，安装Composer以及`asset-plugin`：

```
global require 'fxp/composer-asset-plugin:~1.1.1'
```


2. 使用`ssh-kengen`生成SSH秘钥。
3. 添加`~/.ssh/id_rsa.pub`文件内容到你的库设置页面中（部署SSH秘钥页面），例如Github、Bitbucket或者其它库存储。
4. 尝试游动克隆的库：

```
git clone git@github.com:user/repo.git
```

5. 添加Github地址，以及已知的host列表（如果你的系统问你要的话）。

#### 第二步：准备localhost

1. 在本地全局安装`deploy.phar`：

```
sudo wget http://deployer.org/deployer.phar
sudo mv deployer.phar /usr/local/bin/dep
sudo chmod +x /usr/local/bin/dep
```

2. 使用部署配置添加`deploy.php`文件：

```
<?php
require 'recipe/yii2-app-basic.php';
set('shared_files', [
    'config/db.php',
    'config/params.php',
    'web/index.php',
    'yii',
]);
server('prod', 'site.com', 22) // SSH access to remote server
->user('user')
// ->password(password) // uncomment for authentication by
password
// ->identityFile() // uncomment for authentication by SSH key
->stage('production')
    ->env('deploy_path', '/var/www/project');
set('repository', 'git@github.com:user/repo.git');
```

3. 尝试准备远程项目目录结构：

```
dep deploy:prepare prod
```

#### 第三步：添加远程配置

1. 打开服务器的`/var/www/project`目录。初始化后它有两个子目录：

```
project
├── releases
└── shared
```

2. 在`shared`文件中创建带有私有配置的原始文件：

```
project
    ├── releases
    └── shared
        ├── config
        │   ├── db.php
        │   └── params.php
        ├── web
        │   └── index.php
        └── yii
```

Deployer工具将会在每一个发布的子目录中通过软连接的方式包含这些文件：

在`share/config/db.php`文件中指定你的私有配置：

```
<?php
return [
    'class' => 'yii\db\Connection',
    dsn' => 'mysql:host=localhost;dbname=catalog',
    'username' => 'root',
    'password' => 'root',
    'charset' => 'utf8',
];
```

此外，在`share/config/params.php`中指定它：

```
<?php
return [
    'adminEmail' => 'admin@example.com',
];
```

设置文件`share/web/index.php`的内容：

```
<?php
defined('YII_DEBUG') or define('YII_DEBUG', false);
defined('YII_ENV') or define('YII_ENV', 'prod');
$dir = dirname($_SERVER['SCRIPT_FILENAME']);
require($dir . '/../vendor/autoload.php');
require($dir . '/../vendor/yiisoft/yii2/Yii.php');
$config = require($dir . '/../config/web.php');
(new yii\web\Application($config))->run();
```

此外，设置`share/yii`文件的内容：

```
#!/usr/bin/env php
<?php
defined('YII_DEBUG') or define('YII_DEBUG', false);
defined('YII_ENV') or define('YII_ENV', 'prod');
$dir = dirname($_SERVER['SCRIPT_FILENAME']);
require($dir . '/vendor/autoload.php');
require($dir . '/vendor/yiisoft/yii2/Yii.php');
$config = require($dir. '/config/console.php');
$application = new yii\console\Application($config);
$exitCode = $application->run();
exit($exitCode);
```

**注意**：我们故意使用`dirname($_SERVER['SCRIPT_FILENAME'])`，而不是原始的`__DIR__`常量，因为如果这个文件时软连接的话，`__DIR__`将会返回不正确的值。

注意：如果你使用`yii2-app-advanced`模板，你可以只重定义`config/main-local.php`和`config/params-local.php`文件（backend、frontend、console和common），因为`web/index.php`和`yii`将会自动通过`init`命令生成。

#### 第四步：尝试部署

1. 回到本地，使用`deploy.php`文件，并运行部署命令：

```
dep deploy prod
```

![](../images/a1001.png)

2. 如果成功，你将会看到部署报告：
3. Deployer在你的远程服务器上，创建一个新的发布子目录，并从你的项目到共享的items，以及从`current`目录到当前发布添加软连接：

```
project
├── current -> releases/20160412140556
├── releases
│   └── 20160412140556
│       ├── ...
│       ├── runtime -> /../../shared/runtime
│       ├── web
│       ├── vendor
│       ├── ...
│       └── yii -> /../../shared/yii
└── shared
    ├── config
    │   ├── db.php
    │   └── params.php
    ├── runtime
    ├── web
    │   └── index.php
    └── yii
```

4. 所有这些完成以后，你必须在`project/current/web`目录中设置你的服务器`DocumentRoot`。
5. 如果在部署过程中，发生了一些错误，你可以回滚到先前的发布上：

```
dep rollback prod
```

`current`目录将会定向到你先前的发布文件上。

### 工作原理...

大部分的部署工具都做了同样的任务：

- 创建一个新的发布子目录
- 克隆库文件
- 从项目中制作软连接到共享的目录上，以及到本地配置文件上
- 安装Composer包
- 应用项目migration
- 从服务器的`DocumentRoot`路径上切换软链接到当前发布目录上

Deployer工具为流行的框架都做了预定义。你可以扩展任何已有的例子，或者为你的特殊的例子制作新的。

### 参考

- 欲了解更多关于Deployer的信息，参考[http://deployer.org/docs](http://deployer.org/docs)
- 关于创建SHH秘钥的信息，参考[https://git-scm.com/book/en/v2/Git-on-the-Server-Generating-Your-SSH-Public-Key](https://git-scm.com/book/en/v2/Git-on-the-Server-Generating-Your-SSH-Public-Key)
