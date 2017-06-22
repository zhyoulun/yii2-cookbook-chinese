# 性能调优

在本章中，我们将会讨论如下话题：

- 使用最佳实践
- 加速session处理
- 使用缓存依赖和chains
- profiling一个Yii应用
- Leveraging HTTP缓存
- 合并和最小化资源
- 在HHVM上运行Yii2

Yii是现有最快的框架中的一个。然后，当开发和部署一个应用时，有一些免费的额外性能和遵守最佳实践是有好处的。在本章中，你将会看到如何配置Yii来获取额外的性能。此外，你将会了解到一些最佳实践，可用于开发你的应用，能顺畅的运行，知道你有非常高的负载。

## 使用最佳实践

在本小节中，你将会看到如何配置Yii2，得到最好的性能，以及额外的创建响应式应用的原则。这些原则既是常用的也是Yii相关的。因此，我们将能使用这些原则，甚至不使用Yii2时也可以。

### 准备

按照官方指南[http://www.yiiframework.com/doc-2.0/guide-start-installation.html](http://www.yiiframework.com/doc-2.0/guide-start-installation.html)的描述，使用Composer包管理器创建一个新的`yii2-app-basic`应用。

### 如何做...

1. 更新你的PHP到最新的稳定版本。PHP的主发布版可能会带来非常大的性能提升。关掉调试模式，并设置为`prod`环境。这可以通过编辑`web/index.php`：

```
defined('YII_DEBUG') or define('YII_DEBUG', false);
defined('YII_ENV') or define('YII_ENV', 'prod');
```

**注意**：在`yii2-app-advanced`应用框架中，你可以使用shell命令`php init`，以及选择生产环境，用于加载优化的`index.php`和配置文件。

2. 激活`cache`组件：

```
'components' => [
    'cache' => [
        'class' => 'yii\caching\FileCache',
    ],
],
```

你可以使用任何缓存存储，不只是`FileCache`。此外，你可以注册多个缓存应用组件，并使用`Yii::$app->cache`和`Yii::$app->cache2`来获取不同的数据类型：

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

这个框架默认在它自己的类中使用`cache`组件。

3. 为`db`组件激活表schema缓存：

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

4. 使用纯数据，而不是ActiveRecord对象来列出元素的集合。

```
$categoriesArray = Categories::find()->asArray()->all();
```

5. 在`foreach`中使用`each()`而不是`all()`来获取大量的结果：

```
foreach (Post::find()->each() as $post) {
    // ...
}
```

6. 因为Composer的autoloader被用于包含大部分的第三方类文件，你应该考虑通过如下命令优化它：

```
composer dump-autoload -o
```

### 工作原理...

当`YII_DEBUG`被设置为`false`时，Yii关闭了所有trace级别的日志，并使用较少的错误处理代码。此外，当你设置`YII_ENV`为`prod`时，你的应用不会加载Yii和Debug面板模块。

设置`schemaCachingDuration`为一个以秒为单位的数字，允许Yii的ActiveRecord缓存数据的schema。对于生产环境，我们非常建议这样做，它会大幅提高ActiveRecord的性能。为了使它能功能，你需要正确的配置`cache`：

```
'cache' => [
    'class' => 'yii\cache\FileCache',
],
```

激活缓存对其它Yii组件也有正面的影响。例如，Yii路由或者urlManager从cache路由开始。

当然，你可以进入到一种情况，先前的设置对于显著的提升性能没有帮助。在大部分情况下，这意味着这个应用本身是一个瓶颈，你需要更多的硬件。

- **服务端性能只是重点中的一部分**：服务端性能只是所有能影响全局性能中的一个点。通过优化客户端，例如CSS、图像和Javascript文件，正确的缓存和减少HTTP请求的数量，可以有一个很好的可见的性能提升，即使是没有优化PHP代码。
- **不使用Yii做事**：有些事情如果不使用Yii可以很好的完成。例如，实时修改图像大小在一个独立的PHP脚本中进行会更快，可以避免额外的负载。
- **Active Record和Query Builder以及SQL对比**：在对性能比较敏感的应用部分使用Query Builder和SQL。一般情况下，AR对于添加和编辑记录非常有用，因为它添加一个很方便的校验层，但当查询记录时并没有什么用。
- **经常检查慢查询**：如果开发者意外忘记给一个表格添加索引，当数据库经常被读取时，数据库就会成为性能瓶颈；反之亦然，如果添加太多的索引，而又要经常写数据。同样的事情会发生在选择不必要的数据以及不需要的JOINs。
- **缓存或者保存重型过程的结果**：如果你可以在每一个页面加载过程中避免运行一个重型过程，这最好了。例如，保存或者缓存解析的markdown文本，净化一次后（这是一个非常耗资源的过程），以后就是可以直接用于展示的HTML了。
- **处理太多的过程**：有时有太多的过程需要立即处理。它可以创建复杂的报告，或者只是简单的发送电子邮件（如果你的项目加载的很重）。在这种情况下，最好将它放入到队列中，然后使用cron或者其它指定的工具来处理。

### 参考

欲了解更多关于性能调优和缓存的信息，参考如下地址：

- [http://www.yiiframework.com/doc-2.0/guide-tutorial-performance-tuning.html](http://www.yiiframework.com/doc-2.0/guide-tutorial-performance-tuning.html)
- [http://www.yiiframework.com/doc-2.0/guide-caching-overview.html](http://www.yiiframework.com/doc-2.0/guide-caching-overview.html)

## 加速session处理

在PHP中原生的session处理在大部分情况下已经非常好了。但至少有两个可能原因，你希望改变session的处理方式：

- 当使用多个服务器时，需要有统一的session存储。
- 默认的PHP session使用文件，所以最大的性能瓶颈在磁盘I/O上。
- 默认的PHP session是阻塞并发的session存储。在这个小节中，我们将会看到如何使用Yii做高效的session存储。

### 准备

按照官方指南[http://www.yiiframework.com/doc-2.0/guide-start-installation.html](http://www.yiiframework.com/doc-2.0/guide-start-installation.html)的描述，使用Composer包管理器创建一个新的`yii2-app-basic`应用。并安装Memcache服务器和`memcache` PHP扩展。

### 如何做...

我们使用apache的`ab`工具对网站做压力测试。它是和apache二进制文件一起发布，所以如果正在使用apache，你将会在`bin`文件夹中找到它。

1. 运行如下命令，并将网址替换成你的正在使用的网站的网址：

```
ab -n 1000 -c 5 http://yii-book.app/index.php?r=site/contact
```

这将会发送1000次请求，一次发送5个，并会得到如下输出统计：

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

我们对每秒请求次数指标（requests-per-second，简称QPS）感兴趣。这个值意味着这个网站在并发数为5的情况下，每秒可以处理91.24次请求。

**注意**：注意调试并没有关闭，因为我们对修改session处理速度感兴趣。

2. 现在添加如下代码到`/config/web.php`组件部分：

```
'session' => array(
    'class' => 'yii\web\CacheSession',
    'cache' => 'sessionCache',
),
'sessionCache' => array(
    'class' => 'yii\caching\MemCache',
),
```

3. 再次以相同的设置运行`ab`。这次，你应该能得到更好的结果。在我的例子中，QPS是139.07。这意味着`Memcache`，作为一个session处理器，相对于基于文件的session处理器提升了52%的性能。

**注意**：不要依赖于这里提供的精确的结果。它依赖于软件版本、设置和使用的硬件。经常尝试在你即将部署应用的环境中，运行所有的测试。

4. 通过选择正确的session处理后端，你可以得到一个显著的性能提升。Yii支持更多的缓存后端out-of-the-box，包括WinCache、XCache和Zend Data Cache，它来自于Zend Server。而且，你可以实施你自己的缓存后端，来使用快速的noSQL存储，例如Redis。

### 工作原理...

默认情况下，Yii使用原生PHP session；这意味着大部分情况下使用文件系统。文件系统并不能高效的处理高并发请求。

Memcache或者其它平台在如下情况下，能很好的执行：

```
'session' => array(
    'class' => 'yii\web\CacheSession',
    'cache' => 'sessionCache',
),
'sessionCache' => array(
    'class' => 'yii\caching\MemCache',
),
```

在先前的配置部分，我们在Yii中使用`CacheSession`作为一个session处理器。使用这个组件，我们可以委托session处理器为`cache`中指定的缓存组件。这次我们使用`MemCache`。

当使用一个memcached后端，你应该考虑到这个事实，当使用这些解决方案时，当缓存达到最大存储容量时，应用用户可能丢失session。

**注意**：注意到，当为一个session使用一个缓存后端时，你不能依赖于一个session作为一个临时数据存储，因为在memcached中将不会有更多内存来存储更多数据。在这个例子中，只需要清理所有的数据，并清除其中的一部分。

如果你在使用多个服务器，你不能使用文件存储。没有办法来分享多个服务器之间的session数据。在memcached的例子中，这非常容易，因为它可以被多个服务器访问。

此外，对于分享session数据，你可以使用`DbSession`：

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

现在，在你的数据库中创建一个张新表：

```
CREATE TABLE session (
    id CHAR(40) NOT NULL PRIMARY KEY,
    expire INTEGER,
    data BLOB
)
```

### 更多...

尽可能关闭session是一个好主意。如果你不想在当前的请求中在session中存储任何数据，你甚至可以在你的控制器动作一开始就关闭它。这样，在你的应用中即使是使用文件作为存储也是没关系的。

使用如下命令：

```
Yii:$app->session->close();
```

### 参考

欲了解更多关于性能和缓存的信息，参考如下地址：

- [http://www.yiiframework.com/doc-2.0/guide-tutorial-performance-tuning.html](http://www.yiiframework.com/doc-2.0/guide-tutorial-performance-tuning.html)
- [http://www.yiiframework.com/doc-2.0/guide-caching-overview.html](http://www.yiiframework.com/doc-2.0/guide-caching-overview.html)

## 使用缓存依赖和chains

Yii支持需要缓存后端，但是使Yii缓存灵活的是依赖和依赖chaining支持。有一些情况，你不能简单的只缓存1个小时的数据，因为信息随时可能会边。

在这个小节中，我们将会学习如何缓存整个页面，并能在有更新时获取最新的数据。这个页面是一个仪表盘类型的，将会展示5个最新添加的文章，以及总数。

**注意**：注意一个操作不能被编辑 as it is added，但是一个文章可以。

### 准备

按照官方指南[http://www.yiiframework.com/doc-2.0/guide-start-installation.html](http://www.yiiframework.com/doc-2.0/guide-start-installation.html)的描述，使用Composer包管理器创建一个新的`yii2-app-basic`应用。

1. 在`config/web.php`中激活缓存组件：

```
return [
    // ...
    'components' => [
        'cache' => ['class' => 'yii\caching\FileCache',

        ],
    ],
];
```

2. 设置一个新的数据库，并将它配置到`config/db.php`中：
3. 运行如下migration：

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

4. 使用Yii为account和article表生成模型。
5. 创建`protected/controllers/DashboardController.php`：

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

6. 创建`views/dashboard/index.php`：

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

7. 运行`dashboard/random-operation`和`dashboard/random-article`几次，然后，运行`dashboard/index`你将会看到如下所示的截图：

![](../images/901.png)

8. 在页面的底部，点击调试面板上数据库查询的数量：

![](../images/902.png)

看到一个查询列表：

![](../images/903.png)

### 如何做...

执行如下步骤：

1. 我们需要修改控制器的代码：

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

2. 完成了。现在，在加载`dashboard/index`几次以后，你将会看到只有1个查询，如下所示：

![](../images/904.png)

此外，尝试运行`dashboard/random-operation`或者`dashboard/randomarticle`，并刷新`dashboard/index`。数据将会改变：

![](../images/905.png)

### 工作原理...

为了修改最少的代码，并达到最高的性能，我们使用一个过滤器来作全页缓存：

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

先前的代码意味着我们在`index`动作中应用一个全页缓存。这个页面将会缓存1年，并且如果数据改变了，这个缓存将会刷新。因此，一般情况下，依赖工作如下：

- 按依赖中的描述，第一次运行会获取最新的数据，保存以备之后调用，并更新缓存
- 按依赖中的描述，获取最新的数据，获取保存的数据，然后比较两者
- 如果相等，使用缓存的数据
- 如果不相等，更新缓存，使用最新的数据，并保存最新依赖数据，以备以后调用

在我们的例子中，使用了两个类型的依赖——标签和DB。一个标签依赖使用自定义字符串标签标记数据，并检查它，来决定缓存是否无效，一个DB依赖使用SQL查询结果来达到相同的目的。

现在你可能有的问题是，“为什么有时候使用DB而有时候使用标签？”这是一个好问题。

使用DB依赖的目的是，替换重的计算，并选择一个轻的查询，尽可能获取少的数据。关于这种依赖最好的事情是，我们不需要在已有的代码中嵌入任何额外的逻辑。在我们的例子中，我们可以使用这种类型的依赖用于账户操作，但是不能用于文章，因为文章的内容会改变。因此，对于文章，我们设置一个全局标签，名叫文章，它表示我们可以手动调用来刷新文章缓存：

```
TagDependency::invalidate(\Yii::$app->cache, 'articles');
```

### 参考

欲了解更多关于缓存的信息，并使用缓存依赖，参考[http://www.yiiframework.com/doc-2.0/guide-caching-overview.html](http://www.yiiframework.com/doc-2.0/guide-caching-overview.html)。

## 使用Yii profiling一个应用

如果在部署一个Yii应用时，你使用了所有的最佳实践，但是你仍然得不到你想要的性能，很有可能是应用本身存在一些性能瓶颈。在处理这些性能瓶颈时最主要的原则是你不应该假设任何事请，并在尝试优化它之前去测试和profile代码。

在本小节中，我们将会尝试找出Yii2最小应用的性能瓶颈。

### 准备

按照官方指南[http://www.yiiframework.com/doc-2.0/guide-start-installation.html](http://www.yiiframework.com/doc-2.0/guide-start-installation.html)的描述，使用Composer包管理器创建一个新的`yii2-app-basic`应用。

1. 设置数据库连接，并应用如下migration：

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

2. 在Yii中为每一个表生成模型。
3. 写如下控制台命令：

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

并执行它：

```
./yii data/init
```

4. 添加`ArticleController`类：

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

5. 添加`views/article/index.php`视图：

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

然后添加`views/article/_item.php`：

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

跟随如下步骤，profile基于Yii的应用：

1. 打开文章页面：

![](../images/906.png)

2. 打开`views/article/index.php`并在`ListView`小部件之前和之后添加profiler调用：

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

现在刷新这个页面。

3. 展开页面底部的调试面板，点击timing badge(在我们的例子中是**73ms**)：

![](../images/907.png)

现在检查**Profiling**报告：

![](../images/908.png)

我们可以看到我们的文章块花费了将近40ms。

4. 打开我们的控制器，并为文章的`category`关系添加主动加载：

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

5. 回到网站，刷新页面，再次打开**Profiling**报告：

![](../images/909.png)

现在这个文章列表花费了将近25ms，因为应用使用主动加载做了更少的SQL查询。

### 工作原理...

你可以使用`Yii::beginProfile`和`Yii::endProfile`查看源代码的任何片段：

```
Yii::beginProfile('articles');
// ...
Yii::endProfile('articles');
```

在执行过页面以后，你可以在调试模块的**Profiling**页面看到这个报告，有所有的执行时间。

此外，你可以使用嵌套profiling调用：

```
Yii::beginProfile('outer');
    Yii::beginProfile('inner');
        // ...
    Yii::endProfile('inner');
Yii::endProfile('outer');
```

**注意**：注意要正确的打开和关闭调用，以及正确命名block名称。如果你忘记调用`Yii::endProfile`，或者颠倒了`Yii::endProfile('inner')`和`Yii::endProfile('outer')`的嵌套顺序，性能Profiling将不会工作。

### 参考

- 欲了解更多关于logger的信息，参考[http://www.yiiframework.com/doc-2.0/guide-runtime-logging.html#performance-profiling](http://www.yiiframework.com/doc-2.0/guide-runtime-logging.html#performance-profiling)
- 关于应用性能的调优，参考[http://www.yiiframework.com/doc-2.0/guide-tutorial-performance-tuning.html](http://www.yiiframework.com/doc-2.0/guide-tutorial-performance-tuning.html)

## Leveraging HTTP缓存

不只是服务端的缓存，你通过设置HTTP头可以使用客户端缓存。

在这个小节中，我们将会讨论基于`Last-Modified`和`ETag`头的全页缓存。

### 准备

按照官方指南[http://www.yiiframework.com/doc-2.0/guide-start-installation.html](http://www.yiiframework.com/doc-2.0/guide-start-installation.html)的描述，使用Composer包管理器创建一个新的`yii2-app-basic`应用。

1. 创建并运行migration：

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

2. 创建`Article`模型：

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

3. 创建带有如下动作的博客控制器：

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

4. 添加`views/blog/index.php`视图：

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

5. 添加视图`views/blog/view.php`：

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

执行如下步骤来leverage HTTP缓存：

1. 访问`http://yii-book.app/index.php?r=blog/create`三次，来创建三个文章。
2. 打开如下博客地址：

![](../images/910.png)

3. 在你的浏览器中打开开发者控制台，每次刷新页面都可以看到`200 OK`的响应状态：

![](../images/911.png)

4. 打开`BlogController`并附加如下行为：

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

5. 接下来，刷新页面几次，并检查服务器返回的是`304 Not Modified`，而不是`200 OK`：

![](../images/912.png)

6. 使用如下URL打开相关页面，更新相关文章：`http://yiibook.app/index.php?r=blog/update`。
7. 更新过博客页面以后，检查服务器首次返回的是`200 OK`，接着就是`304 Not Modified`，并确认你在页面上看到了新的更新时间：

![](../images/913.png)

8. 从我们的页面上打开任何页面：

![](../images/914.png)

确认服务器首次返回的是`200 OK`，以及接下来的请求是`304 Not Modified`。

### 工作原理...

在HTTP头的帮助下，你的浏览器用基于时间和基于内容的方法，来检查缓存响应内容的可用性。

#### 上次修改时间

这个方法建议服务端必须返回每个文章的上次修改时间。在存储这个这个日期之后，我们的浏览器可以在接下来的每次请求中，在`If-Modified-Since`头设置这个值。

我们必须附加这个`action`过滤器到我们的控制器中，并指定`lastModified`回到：

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

`\yii\filters\HttpCache`类调用这个回调，并将返回值和`$_SERVER['HTTP_IF_MODIFIED_SINCE']`系统变量进行比较。如果这个文章没有改变，`HttpCache`将会发送一个轻量级的`304`响应头，而且不需要运行这个动作。

但是，如果文档更新了，这个缓存将会失效，服务端将会返回一个完整的响应。

![](../images/917.png)

作为`Last-Modified`的备选或者补充，你可以使用`ETag`。

#### Entity标签

如果我们没有存储上次修改时间，我们可以使用自定义hash，它可以基于文章的内容生成。

例如，我们可以为我们的文档使用一个内容标题，来做了一个hash值：

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

这个`HttpCache`过滤器会将这个tag附加到服务器响应的`ETag`头变量上。

在存储了`ETag`之后，我们的浏览器会为接下来的每次请求附加它在`If-None-Match`头上。

如果这个文档仍然为改变，`HttpCache`将会发送一个轻量级的`304`响应头，并且不需要运行这个动作。

![](../images/918.png)

![](../images/919.png)

当这个缓存是合法的，我们的应用将会发送`304 Not Modified`响应头，而不是页面内容，而且不会重复运行控制器和动作。

### 参考

- 欲了解更多关于HTTP缓存，参考[https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching)
- 对于Yii2中的HTTP缓存，参考[http://www.yiiframework.com/doc-2.0/guide-cachinghttp.html](http://www.yiiframework.com/doc-2.0/guide-cachinghttp.html)

## 和并和最小化assets

如果你的页面包含很多CSS和/或Javascript文件，这个页面将会打开的比较慢，因为浏览器发送了大量的HTTP请求来下载每一个文件。为了减少请求和连接的数量，我们可以在生产模式下合并和压缩多个CSS/Javascript文件到一个或者非常少的几个文件，然后将这些压缩的文件包含在页面上。

### 准备

- 按照官方指南[http://www.yiiframework.com/doc-2.0/guide-start-installation.html](http://www.yiiframework.com/doc-2.0/guide-start-installation.html)的描述，使用Composer包管理器创建一个新的`yii2-app-basic`应用。
- 从[https://developers.google.com/closure/compiler/](https://developers.google.com/closure/compiler/)下载`compiler.jar`文件
- 从[https://github.com/yui/yuicompressor/releases](https://github.com/yui/yuicompressor/releases)下载`yuicompressor.jar`文件
- 从[http://www.java.com](http://www.java.com)下载jre。

### 如何做...

跟随如下步骤，来和并和最小化资源：

1. 打开你的应用`index`页面的源HTML代码。检查是否和如下结构比较相似：

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

这个页面包含三个Javascript文件。

2. 打开`config/console.php`文件，并添加`@webroot`和`@web` alias定义：

```
<?php
Yii::setAlias('@webroot', __DIR__ . '/../web');
Yii::setAlias('@web', '/');
```

3. 打开一个控制台，并运行如下命令：

```
yii asset/template assets.php
```

4. 打开生成的`assets.php`文件，并按如下配置：

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

5. 运行合并命令`yii asset assets.php config/assets-prod.php`。如果成功，你就能得到带有如下配置的`config/assets-prod.php`文件：

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

6. 在`config/web.php`文件中为`assetManager`组件添加配置：

```
'components' => [
    // ...
    'assetManager' => [
        'bundles' => YII_ENV_PROD ? require(__DIR__ . '/assets-prod.php') : [],
    ],
],
```

7. 在`web/index.php`打开生产模式：

```
defined('YII_ENV') or define('YII_ENV', 'prod');
```

8. 在你的浏览器中刷新这个页面，就能看到HTML代码。现在应该有一条包含我们压缩文件的一行：

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

首先，我们的页面有包含文件的集合：

```
<link href="/assets/9b3b2888/css/bootstrap.css" rel="stylesheet">
<link href="/css/site.css" rel="stylesheet">
...
<script src="/assets/25f82b8a/jquery.js"></script>
<script src="/assets/f4307424/yii.js"></script>
<script src="/assets/9b3b2888/js/bootstrap.js"></script>
```

接下来，我们生成`assets.php`配置文件，并制定需要压缩的东西：

```
'bundles' => [
    'app\assets\AppAsset',
    'yii\bootstrap\BootstrapPluginAsset',
],
```

**注意**：我们可以指定所有中间资源包，例如`yii\web\JqueryAsset`和`yii\web\YiiAsset`，但是这些资源已经作为`AppAsset`和`BootstrapPluginAsset`的依赖被指定了，这个压缩命令会自动解析所有的依赖。

AssetManager发布所有的资源到`web/assets`经典子文件夹中，在发布过以后，它会运行压缩器，将所有的CSS和JS文件压缩到`all-{hash}.js`和`all-{hash}.css`文件中。

检查这个CSS文件是否包含其它带有相对路径的资源，例如`bootstrap.css`文件中：

```
@font-face {
    font-family: 'Glyphicons Halflings';
    src: url('../fonts/glyphicons-halflings-regular.eot');
}
```

如果是这样的话，在和并的文件中，我们的压缩器会修改所有的相对路径：

```
@font-face{
    font-family: 'Glyphicons Halflings';
    src: url('9b3b2888/fonts/glyphicons-halflings-regular.eot');
}
```

处理过以后，我们得到了`assets-prod.php`文件，里边有`assetManager`组件的配置。它定义了新的virtual资源作为原始包的干净拷贝：

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

现在，我们可以require这个配置到`config/web.php`文件中：

```
'components' => [
    // ...
    'assetManager' => [
        'bundles' => require(__DIR__ . '/assets-prod.php'),
    ],
],
```

或者，我们可以只在生产环境中require这个文件：

```
'components' => [
    // ...
    'assetManager' => [
        'bundles' => YII_ENV_PROD ? require(__DIR__ . '/assets-prod.php') : [],
    ],
],
```

**注意**：不要忘记在更新了原始资源后重新生成所有的压缩和合并文件。

### 参考

- 欲了解更多关于assets的信息，参考[http://www.yiiframework.com/doc-2.0/guide-structure-assets.html](http://www.yiiframework.com/doc-2.0/guide-structure-assets.html)
- Closure Compiler的信息，参考[https://developers.google.com/closure/compiler/](https://developers.google.com/closure/compiler/)
- 对于YUI压缩器的信息，参考[https://github.com/yui/yuicompressor/](https://github.com/yui/yuicompressor/)

## 在HHVM上运行Yii2

**HipHop Virtual Machine (HHVM)**是一个处理虚拟机器，来自Facebook，基于just-in-time（JIT）编译。HHVM将PHP代码翻译成功中间的**HipHop bytecode (HHBC)**，并动态翻译PHP代码为机器码，它可以被优化并原生的执行。

### 准备

按照官方指南[http://www.yiiframework.com/doc-2.0/guide-start-installation.html](http://www.yiiframework.com/doc-2.0/guide-start-installation.html)的描述，使用Composer包管理器创建一个新的`yii2-app-basic`应用。

### 如何做...

根据如下步骤，在HHVM上运行Yii：

1. 安装Apache2或者Nginx web服务器：
2. 跟随这个指南[https://docs.hhvm.com/hhvm/installation/introduction](https://docs.hhvm.com/hhvm/installation/introduction)，在Linux或者Mac上安装HHVM。例如在Ubuntu上，你需要运行如下命令：

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

3. 尝试为你的网站手动启动内置服务器：

```
cd web
hhvm -m server -p 8080
```

在你的浏览器中打开`localhost:8080`：

![](../images/915.png)

现在你就可以使用HHVM来开发你的项目了。

4. 如果你使用Nginx或者Apache2服务器，HHVM会在`/etc/nginx`和`/etc/apache2`目录中自动创建它自己的配置文件。在Nginx的例子中，它会创建`/etc/nginx/hhvm.conf`模板，来包含你的项目的配置。例如，我们来创建一个新的虚拟托管名叫`yii-book-hhvm.app`：

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

添加hostname到你的`/etc/hosts`：

```
127.0.0.1 yii-book-hhvm.app
```

现在重启这个Nginx服务器：

```
sudo service nginx restart
```

最后，在浏览器中打开这个新的host：

![](../images/916.png)

你的服务器被成功设置。

### 工作原理...

你可以在`fastcgi`模式下使用HHVM作为PHP处理的备选项。默认情况下，它会监听9000端口。你可以在`/etc/hhvm/server.ini`文件中修改`fastcgi`进程的这个默认端口：

```
hhvm.server.port = 9000
```

在`/etc/hhvm/php.ini`文件中配置这个指定的PHP选项：

### 参考

欲了解更多关于安装HHVM的信息，参考如下地址：
- [https://docs.hhvm.com/hhvm/installation/linux](https://docs.hhvm.com/hhvm/installation/linux)
- [https://docs.hhvm.com/hhvm/installation/mac](https://docs.hhvm.com/hhvm/installation/mac)

欲了解更多关于HHVM使用方法的信息，参考[https://docs.hhvm.com/hhvm/](https://docs.hhvm.com/hhvm/)