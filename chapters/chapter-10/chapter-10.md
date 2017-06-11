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

默认情况下，Yii2应用使用`web`文件夹存放你网站的入口脚本。但是

```
require(__DIR__ . '/../application/vendor/autoload.php');
require(__DIR__ . '/../application/vendor/yiisoft/yii2/Yii.php');
```


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



```
return [
    'id' => 'app-manager',
    'basePath' => dirname(__DIR__),
    'controllerNamespace' =>
    'api\controllers',
    // ....
]
```


```
namespaces api\assets;
namespaces api\controllers;
```



```
use backend\assets\AppAsset;
```


```
use api\assets\AppAsset;
```


```
<?php
Yii::setAlias('@common', dirname(__DIR__));
Yii::setAlias('@frontend', dirname(dirname(__DIR__)) .'/frontend');
Yii::setAlias('@backend', dirname(dirname(__DIR__)) .'/backend');
Yii::setAlias('@console', dirname(dirname(__DIR__)) .'/console');
Yii::setAlias('@api', dirname(dirname(__DIR__)) . '/api);
```

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


```
<?php
return [
    '' => 'site/index',
    '<_c:[\w\-]+>/<id:\d+>' => '<_c>/view',
    '<_c:[\w\-]+/<_a:[\w\-]+>>/<id:\d+>' => '<_c>/<_a>',
    '<_c:[\w\-]+>' => '<_c>/index',
];
```


```
'urlManager' => [
    'class' => 'yii\web\UrlManager',
    'enablePrettyUrl' => true,
    'showScriptName' => false,
    'rules' => require(__DIR__ . '/urlRules.php'),
],
```

```
<?php return [...];
```

```
'rules' => require(__DIR__ . '/urlRules.php'),
```



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


```
config
    console.php
    web.php
    db.php
    params.php
```


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



```
<?php return [];
```




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






```
<?php
return [
    'adminEmail' => '',
];
```






```
<?php
return [
    'adminEmail' => 'admin@example.com',
];
```






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





```
$config = yii\helpers\ArrayHelper::merge(
    require(__DIR__ . '/../config/common.php'),
    require(__DIR__ . '/../config/common-local.php'),
    require(__DIR__ . '/../config/web.php'),
    require(__DIR__ . '/../config/web-local.php')
);
```


```
$config = yii\helpers\ArrayHelper::merge(
    require(__DIR__ . '/config/common.php'),
    require(__DIR__ . '/config/common-local.php'),
    require(__DIR__ . '/config/console.php'),
    require(__DIR__ . '/config/console-local.php')
);
```



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



```
/*-local.php
```




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


```
"extra": {
    "asset-installer-paths": {
        "npm-asset-library": "vendor/npm",
        "bower-asset-library": "vendor/bower"
    }
}
```


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


```
php yii
```


```
./yii
```


```
./yii help hello
```


```
DESCRIPTION
This command echoes what you have entered as the message.
USAGE
yii hello [message] [...options...]
- message: string (defaults to 'hello world')
the message to be echoed.
```


```
./yii hello
```


```
./yii hello/index
```


```
Hello world
```


```
./yii hello 'Bond, James Bond'
```


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


```
./yii cron/timestamp
```


```
0 0
* * * www-data /path/to/yii cron/timestamp >/dev/null
```


```
class MaintenanceController extends Controller
{
    public function actionIndex()
    {
        $this->renderPartial("index");
    }
}
```

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


```
$config = [
    'catchAll' => file_exists(dirname(__DIR__).'/.maintenance') && !(isset($_COOKIE['secret']) && $_COOKIE['secret']=="password") ? ['maintenance/index'] : null,
    // …
]
```

```
'catchAll' => ['maintenance/index'],
```



```
file_exists(dirname(__DIR__) . '/.maintenance')
```



```
!(isset($_COOKIE['secret']) && $_COOKIE['secret']=="password")
```



```
global require 'fxp/composer-asset-plugin:~1.1.1'
```


```
git clone git@github.com:user/repo.git
```


```
sudo wget http://deployer.org/deployer.phar
sudo mv deployer.phar /usr/local/bin/dep
sudo chmod +x /usr/local/bin/dep
```


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


```
dep deploy:prepare prod
```


```
project
├── releases
└── shared
```


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


```
<?php
return [
    'adminEmail' => 'admin@example.com',
];
```


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

```
dep deploy prod
```

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
