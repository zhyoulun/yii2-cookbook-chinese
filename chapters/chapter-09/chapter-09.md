# 性能调优



## 使用最佳实践

### 准备

### 如何做...

```
defined('YII_DEBUG') or define('YII_DEBUG', false);
defined('YII_ENV') or define('YII_ENV', 'prod');
```

```
'components' => [
    'cache' => [
        'class' => 'yii\caching\FileCache',
    ],
],
```

```
'components' => [
    'cache' => [
        'class' => 'yii\caching\MemCache',
        'useMemcached' => true,
    ],
    'cache2' => [
        'class' => 'yii\caching\FileCache',
    ],
],
```


```
return [
    // ...
    'components' => [
        // ...
        'cache' => [
            'class' => 'yii\caching\FileCache',
        ],
        'db' => [
            'class' => 'yii\db\Connection',
            'dsn' => 'mysql:host=localhost;dbname=mydatabase',
            'username' => 'root',
            'password' => '',
            'enableSchemaCache' => true,
            // Optional. Default value is 3600 seconds
            'schemaCacheDuration' => 3600,
            // Optional. Default value is 'cache'
            'schemaCache' => 'cache',
        ],
    ],
];
```



```
$categoriesArray = Categories::find()->asArray()->all();
```


```
foreach (Post::find()->each() as $post) {
    // ...
}
```

```
composer dump-autoload -o
```

### 工作原理...

```
'cache' => [
    'class' => 'yii\cache\FileCache',
],
```

### 参考

## 加速session处理

### 准备

### 如何做...

```
ab -n 1000 -c 5 http://yii-book.app/index.php?r=site/contact
```

```
This is ApacheBench, Version 2.3 <$Revision: 1528965 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd,
http://www.zeustech.net/
Licensed to The Apache Software Foundation,
http://www.apache.org/
...
Server Software: nginx
Server Hostname: yii-book.app
Server Port: 80
Document Path: /index.php?r=site/contact
Document Length: 14866 bytes
Concurrency Level: 5
Time taken for tests: 10.961 seconds
Complete requests: 1000
Failed requests: 0
Total transferred: 15442000 bytes
HTML transferred: 14866000 bytes
Requests per second: 91.24 [#/sec] (mean)
Time per request: 54.803 [ms] (mean)
Time per request: 10.961 [ms] (mean, across all
concurrent requests)
Transfer rate: 1375.84 [Kbytes/sec] received
Connection Times (ms)
min mean[+/-sd] median max
Connect: 0 0 0.0 0 0
Processing: 18 55 324.9 29 4702
Waiting: 15 41 255.1 24 4695
Total: 18 55 324.9 29 4702
```

```
'session' => array(
    'class' => 'yii\web\CacheSession',
    'cache' => 'sessionCache',
),
'sessionCache' => array(
    'class' => 'yii\caching\MemCache',
),
```

### 工作原理...


```
'session' => array(
    'class' => 'yii\web\CacheSession',
    'cache' => 'sessionCache',
),
'sessionCache' => array(
    'class' => 'yii\caching\MemCache',
),
```


```
return [
    // ...
    'components' => [
        'session' => [
            'class' => 'yii\web\DbSession',
        ],
    ],
];
```


```
CREATE TABLE session (
    id CHAR(40) NOT NULL PRIMARY KEY,
    expire INTEGER,
    data BLOB
)
```

### 更多...

```
Yii:$app->session->close();
```

### 参考

## 使用缓存依赖和chains

### 注意

### 准备

```
return [
    // ...
    'components' => [
        'cache' => ['class' => 'yii\caching\FileCache',

        ],
    ],
];
```

```
<?php
use yii\db\Schema;
use yii\db\Migration;
class m160308_093233_create_example_tables extends Migration
{
    public function up()
    {
        $tableOptions = null;
        if ($this->db->driverName === 'mysql') {
            $tableOptions = 'CHARACTER SET utf8 COLLATE utf8_general_ci ENGINE=InnoDB';
        }
        $this->createTable('{{%account}}', [
            'id' => Schema::TYPE_PK,
            'amount' => Schema::TYPE_DECIMAL . '(10,2) NOT NULL',
        ], $tableOptions);
        $this->createTable('{{%article}}', [
            'id' => Schema::TYPE_PK,
            'title' => Schema::TYPE_STRING . ' NOT NULL',
            'text' => Schema::TYPE_TEXT . ' NOT NULL',
        ], $tableOptions);
    }
    public function down()
    {
        $this->dropTable('{{%article}}');
        $this->dropTable('{{%account}}');
    }
}
```


```
<?php
namespace app\controllers;
use app\models\Account;
use app\models\Article;
use yii\web\Controller;
class DashboardController extends Controller
{
    public function actionIndex()
    {
        $total = Account::find()->sum('amount');
        $articles = Article::find()->orderBy('id DESC')->limit(5)->all();
        return $this->render('index', array(
            'total' => $total,
            'articles' => $articles,
        ));
    }
    public function actionRandomOperation()
    {
        $rec = new Account();
        $rec->amount = rand(-1000, 1000);
        $rec->save();
        echo 'OK';
    }
    public function actionRandomArticle()
    {
        $n = rand(0, 1000);
        $article = new Article();
        $article->title = "Title #".$n;
        $article->text = "Text #".$n;
        $article->save();
        echo 'OK';
    }
}
```

```
<?php
use yii\helpers\Html;
/* @var $this yii\web\View */
/* @var $total int */
/* @var $articles app\models\Article[] */
?>
    <h1>Total: <?= $total ?></h1>
    <h2>5 latest articles:</h2>
<?php foreach($articles as $article): ?>
    <h3><?= Html::encode($article->title) ?></h3>
    <div><?= Html::encode($article->text) ?></div>
<?php endforeach ?>
```

![](../images/901.png)

![](../images/902.png)

![](../images/903.png)


### 如何做...

```
<?php
namespace app\controllers;
use app\models\Account;
use app\models\Article;
use yii\caching\DbDependency;
use yii\caching\TagDependency;
use yii\web\Controller;
class DashboardController extends Controller
{
    public function behaviors()
    {
        return [
            'pageCache' => [
                'class' => 'yii\filters\PageCache',
                'only' => ['index'],
                'duration' => 24 * 3600 * 365, // 1 year
                'dependency' => [
                    'class' => 'yii\caching\ChainedDependency',
                    'dependencies' => [
                        new TagDependency(['tags' =>
                            ['articles']]),
                        new DbDependency(['sql' => 'SELECT MAX(id) FROM ' . Account::tableName()])
                    ]
                ],
            ],
        ];
    }

    public function actionIndex()
    {
        $total = Account::find()->sum('amount');
        $articles = Article::find()->orderBy('id DESC')->limit(5)->all();
        return $this->render('index', array(
            'total' => $total,
            'articles' => $articles,
        ));
    }
    public function actionRandomOperation()
    {
        $rec = new Account();
        $rec->amount = rand(-1000, 1000);
        $rec->save();
        echo 'OK';
    }

    public function actionRandomArticle()
    {
        $n = rand(0, 1000);
        $article = new Article();
        $article->title = "Title #".$n;
        $article->text = "Text #".$n;
        $article->save();
        TagDependency::invalidate(\Yii::$app->cache,
            'articles');
        echo 'OK';
    }
}
```

![](../images/904.png)

![](../images/905.png)

### 工作原理...

```
public function behaviors()
{
    return [
        'pageCache' => [
            'class' => 'yii\filters\PageCache',
            'only' => ['index'],
            'duration' => 24 * 3600 * 365, // 1 year
            'dependency' => [
                'class' => 'yii\caching\ChainedDependency',
                'dependencies' => [
                    new TagDependency(['tags' => ['articles']]),
                    new DbDependency(['sql' => 'SELECT MAX(id) FROM account'])
                ]
            ],
        ],
    ];
}
```

### 参考

## 使用Yii profiling一个应用

### 准备

```
<?php
use yii\db\Migration;
class m160308_093233_create_example_tables extends Migration
{
    public function up()
    {
        $tableOptions = null;
        if ($this->db->driverName === 'mysql') {
            $tableOptions = 'CHARACTER SET utf8 COLLATE utf8_general_ci ENGINE=InnoDB';
        }
        $this->createTable('{{%category}}', [
            'id' => $this->primaryKey(),
            'name' => $this->string()->notNull(),
        ], $tableOptions);
        $this->createTable('{{%article}}', [
            'id' => $this->primaryKey(),
            'category_id' => $this->integer()->notNull(),
            'title' => $this->string()->notNull(),
            'text' => $this->text()->notNull(),
        ], $tableOptions);
        $this->createIndex('idx-article-category_id',
            '{{%article}}', 'category_id');
        $this->addForeignKey('fk-article-category_id',
            '{{%article}}', 'category_id', '{{%category}}', 'id');
    }
    public function down()
    {
        $this->dropTable('{{%article}}');
        $this->dropTable('{{%category}}');
    }
}
```

```
<?php
namespace app\commands;
use app\models\Article;
use app\models\Category;
use Faker\Factory;
use yii\console\Controller;
class DataController extends Controller
{
    public function actionInit()
    {
        $db = \Yii::$app->db;
        $faker = Factory::create();
        $transaction = $db->beginTransaction();
        try {
            $categories = [];
            for ($id = 1; $id <= 100; $id++) {
                $categories[] = [
                    'id' => $id,
                    'name' => $faker->name,
                ];
            }
            $db->createCommand()->batchInsert(Category::tableName(), ['id', 'name'], $categories)->execute();
            $articles = [];
            for ($id = 1; $id <= 100; $id++) {
                $articles[] = [
                    'id' => $id,
                    'category_id' => $faker->numberBetween(1, 100),
                    'title' => $faker->text($maxNbChars = 100),
                    'text' => $faker->text($maxNbChars = 200),
                ];
            }
            $db->createCommand()
                ->batchInsert(Article::tableName(), ['id', 'category_id', 'title', 'text'], $articles)->execute();
            $transaction->commit();
        } catch (\Exception $e) {
            $transaction->rollBack();
            throw $e;
        }
    }
}
```

```
./yii data/init
```

```
<?php
namespace app\controllers;
use Yii;
use app\models\Article;
use yii\data\ActiveDataProvider;
use yii\web\Controller;
class ArticleController extends Controller
{
    public function actionIndex()
    {
        $query = Article::find();
        $dataProvider = new ActiveDataProvider([
            'query' => $query,
        ]);
        return $this->render('index', [
            'dataProvider' => $dataProvider,
        ]);
    }
}
```


```
<?php
use yii\helpers\Html;
use yii\widgets\ListView;
/* @var $this yii\web\View */
/* @var $dataProvider yii\data\ActiveDataProvider */
$this->title = 'Articles';
$this->params['breadcrumbs'][] = $this->title;
?>
<div class="article-index">
    <h1><?= Html::encode($this->title) ?></h1>
    <?= ListView::widget([
        'dataProvider' => $dataProvider,
        'itemOptions' => ['class' => 'item'],
        'itemView' => '_item',
    ]) ?>
</div>
```

```
<?php
use yii\helpers\Html;
/* @var $this yii\web\View */
/* @var $model app\models\Article */
?>
<div class="panel panel-default">
    <div class="panel-heading"><?= Html::encode($model->title);
        ?></div>
    <div class="panel-body">
        Category: <?= Html::encode($model->category->name) ?>
    </div>
</div>
```

### 如何做...


![](../images/906.png)

```
<div class="article-index">
    <h1><?= Html::encode($this->title) ?></h1>
    <?php Yii::beginProfile('articles') ?>
    <?= ListView::widget([
        'dataProvider' => $dataProvider,
        'itemOptions' => ['class' => 'item'],
        'itemView' => '_item',
    ]) ?>
    <?php Yii::endProfile('articles') ?>
</div>
```

![](../images/907.png)

![](../images/908.png)

```
class ArticleController extends Controller
{
    public function actionIndex()
    {
        $query = Article::find()->with('category');
        $dataProvider = new ActiveDataProvider([
            'query' => $query,
        ]);
        return $this->render('index', [
            'dataProvider' => $dataProvider,
        ]);
    }
}
```

![](../images/909.png)

### 工作原理...

```
Yii::beginProfile('articles');
// ...
Yii::endProfile('articles');
```

```
Yii::beginProfile('outer');
    Yii::beginProfile('inner');
        // ...
    Yii::endProfile('inner');
Yii::endProfile('outer');
```


### 参考

## Leveraging HTTP缓存

### 准备

```
<?php
use yii\db\Migration;
class m160308_093233_create_example_tables extends Migration
{
    public function up()
    {
        $this->createTable('{{%article}}', [
            'id' => $this->primaryKey(),
            'created_at' => $this->integer()->unsigned()->notNull(),
            'updated_at' =>
                $this->integer()->unsigned()->notNull(),
            'title' => $this->string()->notNull(),
            'text' => $this->text()->notNull(),
        ]);
    }
    public function down()
    {
        $this->dropTable('{{%article}}');
    }
}
```


```
<?php
namespace app\models;
use Yii;
use yii\behaviors\TimestampBehavior;
use yii\db\ActiveRecord;
class Article extends ActiveRecord
{
    public static function tableName()
    {
        return '{{%article}}';
    }
    public function behaviors()
    {
        return [
            TimestampBehavior::className(),
        ];
    }
}
```


```
<?php
namespace app\controllers;
use app\models\Article;
use yii\web\Controller;
use yii\web\NotFoundHttpException;
class BlogController extends Controller
{
    public function actionIndex()
    {
        $articles = Article::find()->orderBy(['id' =>
            SORT_DESC])->all();
        return $this->render('index', array(
            'articles' => $articles,
        ));
    }
    public function actionView($id)
    {
        $article = $this->findModel($id);
        return $this->render('view', array(
            'article' => $article,
        ));
    }
    public function actionCreate()
    {
        $n = rand(0, 1000);
        $article = new Article();
        $article->title = 'Title #' . $n;
        $article->text = 'Text #' . $n;
        $article->save();
        echo 'OK';
    }
    public function actionUpdate($id)
    {
        $article = $this->findModel($id);
        $n = rand(0, 1000);
        $article->title = 'Title #' . $n;
        $article->text = 'Text #' . $n;
        $article->save();
        echo 'OK';
    }
    private function findModel($id)
    {
        if (($model = Article::findOne($id)) !== null) {
            return $model;
        } else {
            throw new NotFoundHttpException('The requested page does not exist.');
        }
    }
}
```

```
<?php
use yii\helpers\Html;
$this->title = 'Articles';;
$this->params['breadcrumbs'][] = $this->title;
?>
<?php foreach($articles as $article): ?>
    <h3><?= Html::a(Html::encode($article->title), ['view',
            'id' => $article->id]) ?></h3>
    <div>Created <?= Yii::$app->formatter->asDatetime($article->created_at) ?></div>
    <div>Updated <?= Yii::$app->formatter->asDatetime($article->updated_at) ?></div>
<?php endforeach ?>
```

```
<?php
use yii\helpers\Html;
$this->title = $article->title;
$this->params['breadcrumbs'][] = ['label' => 'Articles', 'url' => ['index']];
$this->params['breadcrumbs'][] = $this->title;
?>
<h1><?= Html::encode($article->title) ?></h1>
<div>Created <?= Yii::$app->formatter->asDatetime($article->created_at) ?></div>
<div>Updated <?= Yii::$app->formatter->asDatetime($article->updated_at) ?></div>
<hr />
<p><?= Yii::$app->formatter->asNtext($article->text) ?></p>
```



### 如何做...

![](../images/910.png)

![](../images/911.png)

```
<?php
class BlogController extends Controller
{
    public function behaviors()
    {
        return [
            [
                'class' => 'yii\filters\HttpCache',
                'only' => ['index'],
                'lastModified' => function ($action, $params) {
                    return Article::find()->max('updated_at');
                },
            ],
            [
                'class' => 'yii\filters\HttpCache',
                'only' => ['view'],
                'etagSeed' => function ($action, $params) {
                    $article = $this->findModel(\Yii::$app->request->get('id'));
                    return serialize([$article->title, $article->text]);
                },
            ],
        ];
    }
    // ...
}
```

![](../images/912.png)

![](../images/913.png)

![](../images/914.png)

### 工作原理...



#### 上次修改时间

```
<?php
class BlogController extends Controller
{
    public function behaviors()
    {
        return [
            [
                'class' => 'yii\filters\HttpCache',
                'only' => ['index'],
                'lastModified' => function ($action, $params) {
                    return Article::find()->max('updated_at');
                },
            ],
            // ...
        ];
    }
    // ...
}
```

#### Entity标签

```
class BlogController extends Controller
{
    public function behaviors()
    {
        return [
            [
                'class' => 'yii\filters\HttpCache',
                'only' => ['view'],
                'etagSeed' => function ($action, $params) {
                    $article =
                        $this->findModel(\Yii::$app->request->get('id'));
                    return serialize([$article->title,
                        $article->text]);
                },
            ],
        ];
    }
    // ...
}
```

### 参考

## 和并和最小化assets


### 准备

### 如何做...

```
<!DOCTYPE html>
<html lang="en-US">
<head>
    ...
    <title>My Yii Application</title>
    <link href="/assets/9b3b2888/css/bootstrap.css"
          rel="stylesheet">
    <link href="/css/site.css" rel="stylesheet">
</head>
<body>
...
<script src="/assets/25f82b8a/jquery.js"></script>
<script src="/assets/f4307424/yii.js"></script>
<script src="/assets/9b3b2888/js/bootstrap.js"></script>
</body>
</html>
```


```
<?php
Yii::setAlias('@webroot', __DIR__ . '/../web');
Yii::setAlias('@web', '/');
```

```
yii asset/template assets.php
```


```
<?php
return [
    'jsCompressor' => 'java -jar compiler.jar --js {from}
--js_output_file {to}',
    'cssCompressor' => 'java -jar yuicompressor.jar --type css
{from} -o {to}',
    'bundles' => [
        'app\assets\AppAsset',
        'yii\bootstrap\BootstrapPluginAsset',
    ],
    'targets' => [
        'all' => [
            'class' => 'yii\web\AssetBundle',
            'basePath' => '@webroot/assets',
            'baseUrl' => '@web/assets',
            'js' => 'all-{hash}.js',
            'css' => 'all-{hash}.css',
        ],
    ],
    'assetManager' => [
        'basePath' => '@webroot/assets',
        'baseUrl' => '@web/assets',
    ],
];
```

```
<?php
return [
    'all' => [
        'class' => 'yii\\web\\AssetBundle',
        'basePath' => '@webroot/assets',
        'baseUrl' => '@web/assets',
        'js' => [
            'all-fe792d4766bead53e7a9d851adfc6ec2.js',
        ],
        'css' => [
            'all-37cfb42649f74eb0a4bfe0d0e715c420.css',
        ],
    ],
    'yii\\web\\JqueryAsset' => [
        'sourcePath' => null,
        'js' => [],
        'css' => [],
        'depends' => [
            'all',
        ],
    ],
    'yii\\web\\YiiAsset' => [
        'sourcePath' => null,
        'js' => [],
        'css' => [],
        'depends' => [
            'yii\\web\\JqueryAsset',
            'all',
        ],
    ],
    'yii\\bootstrap\\BootstrapAsset' => [
        'sourcePath' => null,
        'js' => [],
        'css' => [],
        'depends' => [
            'all',
        ],
    ],
    'app\\assets\\AppAsset' => [
        'sourcePath' => null,
        'js' => [],
        'css' => [],
        'depends' => [
            'yii\\web\\YiiAsset',
            'yii\\bootstrap\\BootstrapAsset',
            'all',
        ],
    ],
    'yii\\bootstrap\\BootstrapPluginAsset' => [
        'sourcePath' => null,
        'js' => [],
        'css' => [],
        'depends' => [
            'yii\\web\\JqueryAsset',
            'yii\\bootstrap\\BootstrapAsset',
            'all',
        ],
    ],
];
```


```
'components' => [
    // ...
    'assetManager' => [
        'bundles' => YII_ENV_PROD ? require(__DIR__ . '/assets-prod.php') : [],
    ],
],
```


```
defined('YII_ENV') or define('YII_ENV', 'prod');
```

```
<!DOCTYPE html>
<html lang="en-US">
<head>
...
<title>My Yii Application</title>
<link href="/assets/
all-37cfb42649f74eb0a4bfe0d0e715c420.css" rel="stylesheet">
</head>
<body>
...
<script src="/assets/
all-fe792d4766bead53e7a9d851adfc6ec2.js"></script>
</body>
</html>
```

### 工作原理...


```
<link href="/assets/9b3b2888/css/bootstrap.css" rel="stylesheet">
<link href="/css/site.css" rel="stylesheet">
...
<script src="/assets/25f82b8a/jquery.js"></script>
<script src="/assets/f4307424/yii.js"></script>
<script src="/assets/9b3b2888/js/bootstrap.js"></script>
```

```
'bundles' => [
    'app\assets\AppAsset',
    'yii\bootstrap\BootstrapPluginAsset',
],
```

```
@font-face {
    font-family: 'Glyphicons Halflings';
    src: url('../fonts/glyphicons-halflings-regular.eot');
}
```

```
@font-face{
    font-family: 'Glyphicons Halflings';
    src: url('9b3b2888/fonts/glyphicons-halflings-regular.eot');
}
```

```
return [
    'all' => [
        'class' => 'yii\\web\\AssetBundle',
        'basePath' => '@webroot/assets',
        'baseUrl' => '@web/assets',
        'js' => [
            'all-fe792d4766bead53e7a9d851adfc6ec2.js',
        ],
        'css' => [
            'all-37cfb42649f74eb0a4bfe0d0e715c420.css',
        ],
    ],
    'yii\\web\\JqueryAsset' => [
        'sourcePath' => null,
        'js' => [],
        'css' => [],
        'depends' => [
            'all',
        ],
    ],
    // ...
]
```


```
'components' => [
    // ...
    'assetManager' => [
        'bundles' => require(__DIR__ . '/assets-prod.php'),
    ],
],
```


```
'components' => [
    // ...
    'assetManager' => [
        'bundles' => YII_ENV_PROD ? require(__DIR__ . '/assets-prod.php') : [],
    ],
],
```

### 参考


## 在HHVM上运行Yii2

### 准备

### 如何做...

```
sudo apt-get install software-properties-common
sudo apt-key adv --recv-keys --keyserver
hkp://keyserver.ubuntu.com:80 0x5a16e7281be7a449
sudo add-apt-repository "deb http://dl.hhvm.com/ubuntu
$(lsb_release -sc) main"
sudo apt-get update
sudo apt-get install hhvm
After installing, you will see the following tips in your
terminal:
****************************************************************
****
* HHVM is installed.
*
* Running PHP web scripts with HHVM is done by having your
* webserver talk to HHVM over FastCGI. Install nginx or Apache,
* and then:
* $ sudo /usr/share/hhvm/install_fastcgi.sh
* $ sudo /etc/init.d/hhvm restart
* (if using nginx) $ sudo /etc/init.d/nginx restart
* (if using apache) $ sudo /etc/init.d/apache restart
*
* Detailed FastCGI directions are online at:
* https://github.com/facebook/hhvm/wiki/FastCGI
*
* If you're using HHVM to run web scripts, you probably want it
* to start at boot:
* $ sudo update-rc.d hhvm defaults
*
* Running command-line scripts with HHVM requires no special
setup:
* $ hhvm whatever.php
*
* You can use HHVM for /usr/bin/php even if you have php-cli
* installed:
* $ sudo /usr/bin/update-alternatives \
* --install /usr/bin/php php /usr/bin/hhvm 60
****************************************************************
****
```


```
cd web
hhvm -m server -p 8080
```

![](../images/915.png)

```
server {
    listen 127.0.0.1:80;
    server_name .yii-book-hhvm.app;
    root /var/www/yii-book-hhvm.app/web;
    charset utf-8;
    index index.php index.html index.htm;
    include /etc/nginx/hhvm.conf;
}
```

```
127.0.0.1 yii-book-hhvm.app
```

```
sudo service nginx restart
```

![](../images/916.png)

### 工作原理...

```
hhvm.server.port = 9000
```

### 参考

