# 第八章 扩展Yii

## 介绍

## 创建帮助类

### 准备

### 如何做...

```
<?php
namespace app\helpers;
class NumberHelper
{
    public static function format($value, $decimal = 2)
    {
        return number_format($value, $decimal, '.', ',');
    }
}
```


```
<?php
...
class SiteController extends Controller
{
    …
    public function actionNumbers()
    {
        return $this->render('numbers', ['value' => 18878334526.3]);
    }
}
```


```
<?php
use app\helpers\NumberHelper;
use yii\helpers\Html;
/* @var $this yii\web\View */
/* @var $value float */
$this->title = 'Numbers';
$this->params['breadcrumbs'][] = $this->title;
?>
<div class="site-numbers">
    <h1><?= Html::encode($this->title) ?></h1>
    <p>
        Raw number:<br />
        <b><?= $value ?></b>
    </p>
    <p>
        Formatted number:<br />
        <b><?= NumberHelper::format($value) ?></b>
    </p>
</div>
```


### 工作原理...

#### 注意

### 参考

## 创建模型行为

### 准备

```
<?php
use yii\db\Migration;
class m160427_103115_create_post_table extends Migration
{
    public function up()
    {
        $this->createTable('{{%post}}', [
            'id' => $this->primaryKey(),
            'title' => $this->string()->notNull(),
            'content_markdown' => $this->text(),
            'content_html' => $this->text(),
        ]);
    }
    public function down()
    {
        $this->dropTable('{{%post}}');
    }
}
```


```
./yii migrate
tests/codeception/bin/yii migrate
```

```
<?php
namespace app\models;
use app\behaviors\MarkdownBehavior;
use yii\db\ActiveRecord;
/**
 * @property integer $id
 * @property string $title
 * @property string $content_markdown
 * @property string $content_html
 */
class Post extends ActiveRecord
{
    public static function tableName()
    {
        return '{{%post}}';
    }
    public function rules()
    {
        return [
            [['title'], 'required'],
            [['content_markdown'], 'string'],
            [['title'], 'string', 'max' => 255],
        ];
    }
}
```





### 如何做...

```
<?php
namespace app\tests\codeception\unit\fixtures;
use yii\test\ActiveFixture;
class PostFixture extends ActiveFixture
{
    public $modelClass = 'app\models\Post';
    public $dataFile = '@tests/codeception/unit/fixtures/data/post.php';
}
```


```
<?php
return [
    [
        'id' => 1,
        'title' => 'Post 1',
        'content_markdown' => 'Stored *markdown* text 1',
        'content_html' => "<p>Stored <em>markdown</em> text 1</p>\n",
    ],
];
```


```
<?php
namespace app\tests\codeception\unit;
use app\models\Post;
use app\tests\codeception\unit\fixtures\PostFixture;
use yii\codeception\DbTestCase;
class MarkdownBehaviorTest extends DbTestCase
{
    public function testNewModelSave()
    {
        $post = new Post();
        $post->title = 'Title';
        $post->content_markdown = 'New *markdown* text';
        $this->assertTrue($post->save());
        $this->assertEquals("<p>New <em>markdown</em> text</p>\n", $post->content_html);
    }
    public function testExistingModelSave()
    {
        $post = Post::findOne(1);
        $post->content_markdown = 'Other *markdown* text';
        $this->assertTrue($post->save());
        $this->assertEquals("<p>Other <em>markdown</em> text</p>\n", $post->content_html);
    }
    public function fixtures()
    {
        return [
            'posts' => [
                'class' => PostFixture::className(),
            ]
        ];
    }
}
```




```
codecept run unit MarkdownBehaviorTest
Ensure that tests has not passed:
Codeception PHP Testing Framework v2.0.9
Powered by PHPUnit 4.8.27 by Sebastian Bergmann and
contributors.
Unit Tests (2)
----------------------------------------------------------------
-----------
Trying to test ...
MarkdownBehaviorTest::testNewModelSave Error
Trying to test ...
MarkdownBehaviorTest::testExistingModelSave Error
----------------------------------------------------------------
-----------
Time: 289 ms, Memory: 16.75MB
```


```
<?php
namespace app\behaviors;
use yii\base\Behavior;
use yii\base\Event;
use yii\base\InvalidConfigException;
use yii\db\ActiveRecord;
use yii\helpers\Markdown;
class MarkdownBehavior extends Behavior
{
    public $sourceAttribute;
    public $targetAttribute;
    public function init()
    {
        if (empty($this->sourceAttribute) ||
            empty($this->targetAttribute)) {
            throw new InvalidConfigException('Source and target must be set.');
        }
        parent::init();
    }
    public function events()
    {
        return [
            ActiveRecord::EVENT_BEFORE_INSERT => 'onBeforeSave',
            ActiveRecord::EVENT_BEFORE_UPDATE => 'onBeforeSave',
        ];
    }
    public function onBeforeSave(Event $event)
    {
        if
        ($this->owner->isAttributeChanged($this->sourceAttribute)) {
            $this->processContent();
        }
    }
    private function processContent()
    {
        $model = $this->owner;
        $source = $model->{$this->sourceAttribute};
        $model->{$this->targetAttribute} =
            Markdown::process($source);
    }
}
```



```
class Post extends ActiveRecord
{
    ...
    public function behaviors()
    {
        return [
            'markdown' => [
                'class' => MarkdownBehavior::className(),
                'sourceAttribute' => 'content_markdown',
                'targetAttribute' => 'content_html',
            ],
        ];
    }
}
```

```
Codeception PHP Testing Framework v2.0.9
Powered by PHPUnit 4.8.27 by Sebastian Bergmann and
contributors.
Unit Tests (2)
----------------------------------------------------------------
-----------
Trying to test ...
MarkdownBehaviorTest::testNewModelSave Ok
Trying to test ...
MarkdownBehaviorTest::testExistingModelSave Ok
----------------------------------------------------------------
-----------
Time: 329 ms, Memory: 17.00MB
```

### 工作原理...

```
public function events()
{
    return [
        ActiveRecord::EVENT_BEFORE_INSERT => 'onBeforeSave',
        ActiveRecord::EVENT_BEFORE_UPDATE => 'onBeforeSave',
    ];
}
```




```
public function onBeforeSave(Event $event)
{
    if ($this->owner->isAttributeChanged($this->sourceAttribute))
    {
        $this->processContent();
    }
}
```


```
class Post extends ActiveRecord
{
    ...
    public function behaviors()
    {
        return [
            [
                'class' => MarkdownBehavior::className(),
                'sourceAttribute' => 'description_markdown',
                'targetAttribute' => 'description_html',
            ],
            [
                'class' => MarkdownBehavior::className(),
                'sourceAttribute' => 'content_markdown',
                'targetAttribute' => 'content_html',
            ],
        ];
    }
}
```

### 参考

## 创建组件

### 准备

### 如何做...

```
{
    "base":"USD",
    "date":"2016-05-13",
    "rates": {
        "AUD":1.3728,
        "BGN":1.7235,
        ...
        "ZAR":15.168,
        "EUR":0.88121
    }
}
```


```
<?php
namespace app\components;
use yii\base\Component;
class Exchange extends Component
{
    public function getRate($source, $destination, $date = null)
    {
    }
}
```



```
<?php
namespace app\components;
use yii\base\Component;
use yii\base\InvalidConfigException;
use yii\base\InvalidParamException;
use yii\caching\Cache;
use yii\di\Instance;
use yii\helpers\Json;
class Exchange extends Component
{
    /**
     * @var string remote host
     */
    public $host = 'http://api.fixer.io';
    /**
     * @var bool cache results or not
     */
    public $enableCaching = false;
    /**
     * @var string|Cache component ID
     */
    public $cache = 'cache';
    public function init()
    {
        if (empty($this->host)) {
            throw new InvalidConfigException('Host must be set.');
        }
        if ($this->enableCaching) {
            $this->cache = Instance::ensure($this->cache,
                Cache::className());
        }
        parent::init();
    }
    public function getRate($source, $destination, $date = null)
    {
        $this->validateCurrency($source);
        $this->validateCurrency($destination);
        $date = $this->validateDate($date);
        $cacheKey = $this->generateCacheKey($source,
            $destination, $date);
        if (!$this->enableCaching || ($result =
                $this->cache->get($cacheKey)) === false) {
            $result = $this->getRemoteRate($source,
                $destination, $date);
            if ($this->enableCaching) {
                $this->cache->set($cacheKey, $result);
            }
        }
        return $result;
    }
    private function getRemoteRate($source, $destination, $date)
    {
        $url = $this->host . '/' . $date . '?base=' . $source;
        $response = Json::decode(file_get_contents($url));
        if (!isset($response['rates'][$destination])) {
            throw new \RuntimeException('Rate not found.');
        }
        return $response['rates'][$destination];
    }
    private function validateCurrency($source)
    {
        if (!preg_match('#^[A-Z]{3}$#s', $source)) {
            throw new InvalidParamException('Invalid currency format.');
        }
    }
    private function validateDate($date)
    {
        if (!empty($date) &&
            !preg_match('#\d{4}\-\d{2}-\d{2}#s', $date)) {
            throw new InvalidParamException('Invalid date format.');
        }
        if (empty($date)) {
            $date = date('Y-m-d');
        }
        return $date;
    }
    private function generateCacheKey($source, $destination,
                                      $date)
    {
        return [__CLASS__, $source, $destination, $date];
    }
}
```


```
'components' => [
    'cache' => [
        'class' => 'yii\caching\FileCache',
    ],
    'exchange' => [
        'class' => 'app\components\Exchange',
        'enableCaching' => true,
    ],
    // ...
    db' => $db,
],
```

```
echo \Yii::$app->exchange->getRate('USD', 'EUR');
echo \Yii::$app->get('exchange')->getRate('USD', 'EUR', '2014-04-12');
```


```
<?php
namespace app\commands;
use yii\console\Controller;
class ExchangeController extends Controller
{
    public function actionTest($currency, $date = null)
    {
        echo \Yii::$app->exchange->getRate('USD', $currency,
                $date) . PHP_EOL;
    }
}
```


```
$ ./yii exchange/test EUR
> 0.90196

$ ./yii exchange/test EUR 2015-11-24
> 0.93888

$ ./yii exchange/test OTHER
> Exception 'yii\base\InvalidParamException' with message 'Invalid currency format.'

$ ./yii exchange/test EUR 2015/24/11
Exception 'yii\base\InvalidParamException' with message 'Invalid date format.'

$ ./yii exchange/test ASD
> Exception 'RuntimeException' with message 'Rate not found.'
```

#### 覆盖已经存在的应用组件

```
<?php
namespace app\components;
class Formatter extends \yii\i18n\Formatter
{
    public function asNumber($value, $decimal = 2)
    {
        return number_format($value, $decimal, '.', ',');
    }
}
```

```
'components' => [
    // ...
    formatter => [
        'class' => 'app\components\Formatter,
    ],
    // ...
],
```


```
echo Yii::app()->formatter->asNumber(1534635.2, 3);
```


```
<?= \yii\grid\GridView::widget([
    'dataProvider' => $dataProvider,
    'columns' => [
        'id',
        'created_at:datetime',
        'title',
        'value:number',
    ],
]) ?>
```

### 工作原理...

### 参考

## 创建可重用控制器动作


### 准备

```
<?php
use yii\db\Migration;
class m160308_093233_create_post_table extends Migration
{
    public function up()
    {
        $this->createTable('{{%post}}', [
            'id' => $this->primaryKey(),
            'title' => $this->string()->notNull(),
            'text' => $this->text()->notNull(),
        ]);
    }
    public function down()
    {
        $this->dropTable('{{%post}}');
    }
}
```

### 如何做...


```
<?php
namespace app\actions;
use yii\base\Action;
use yii\base\InvalidConfigException;
use yii\web\MethodNotAllowedHttpException;
use yii\web\NotFoundHttpException;
class DeleteAction extends Action
{
    public $modelClass;
    public $redirectTo = ['index'];
    public function init()
    {
        if (empty($this->modelClass)) {
            throw new InvalidConfigException('Empty model class.');
        }
        parent::init();
    }
    public function run($id)
    {
        if (!\Yii::$app->getRequest()->getIsPost()) {
            throw new MethodNotAllowedHttpException('Method not allowed.');
        }
        $model = $this->findModel($id);
        $model->delete();
        return $this->controller->redirect($this->redirectTo);
    }
    /**
     * @param $id
     * @return \yii\db\ActiveRecord
     * @throws NotFoundHttpException
     */
    private function findModel($id)
    {
        $class = $this->modelClass;
        if (($model = $class::findOne($id)) !== null) {
            return $model;
        } else {
            throw new NotFoundHttpException('Page does not exist.');
        }
    }
}
```

```
<?php
namespace app\controllers;
use app\actions\DeleteAction;
use Yii;
use app\models\Post;
use app\models\PostSearch;
use yii\web\Controller;
use yii\web\NotFoundHttpException;
class PostController extends Controller
{
    public function actions()
    {
        return [
            'delete' => [
                'class' => DeleteAction::className(),
                'modelClass' => Post::className(),
            ],
        ];
    }
    public function actionIndex() { ... }
    public function actionView($id) { ... }
    public function actionCreate() { ... }
    public function actionUpdate($id) { ... }
    protected function findModel($id)
    {
        if (($model = Post::findOne($id)) !== null) {
            return $model;
        } else {
            throw new NotFoundHttpException('The requested page does not exist.');
        }
    }
}
```


### 工作原理...

### 参考

## 创建可重用控制器

### 准备

### 如何做...

```
<?php
namespace app\cleaner;
use Yii;
use yii\filters\VerbFilter;
use yii\helpers\FileHelper;
use yii\web\Controller;
class CleanController extends Controller
{
    public $assetPaths = ['@app/web/assets'];
    public $runtimePaths = ['@runtime'];
    public $caches = ['cache'];
    public function behaviors()
    {
        return [
            'verbs' => [
                'class' => VerbFilter::className(),
                'actions' => [
                    'assets' => ['post'],
                    'runtime' => ['post'],
                    'cache' => ['post'],
                ],
            ],
        ];
    }
    public function actionIndex()
    {
        return $this->render('@app/cleaner/views/index');
    }
    public function actionAssets()
    {
        foreach ((array)$this->assetPaths as $path) {
            $this->cleanDir($path);
            Yii::$app->session->addFlash(
                'cleaner',
                'Assets path "' . $path . '" is cleaned.'
            );
        }
        return $this->redirect(['index']);
    }
    public function actionRuntime()
    {
        foreach ((array)$this->runtimePaths as $path) {
            $this->cleanDir($path);
            Yii::$app->session->addFlash(
                'cleaner',
                'Runtime path "' . $path . '" is cleaned.'
            );
        }
        return $this->redirect(['index']);
    }
    public function actionCache()
    {
        foreach ((array)$this->caches as $cache) {
            Yii::$app->get($cache)->flush();
            Yii::$app->session->addFlash(
                'cleaner',
                'Cache "' . $cache . '" is cleaned.'
            );
        }
        return $this->redirect(['index']);
    }
    private function cleanDir($dir)
    {
        $iterator = new \DirectoryIterator(Yii::getAlias($dir));
        foreach($iterator as $sub) {
            if(!$sub->isDot() && $sub->isDir()) {
                FileHelper::removeDirectory($sub->getPathname());
            }
        }
    }
}
```

```
<?php
use yii\helpers\Html;
/* @var $this yii\web\View */
$this->title = 'Cleaner';
$this->params['breadcrumbs'][] = $this->title;
?>
<div class="clean-index">
    <h1><?= Html::encode($this->title) ?></h1>
    <?php if (Yii::$app->session->hasFlash('cleaner')): ?>
        <?php foreach
        ((array)Yii::$app->session->getFlash('cleaner', []) as
         $message): ?>
            <div class="alert alert-success">
                <?= $message ?>
            </div>
        <?php endforeach; ?>
    <?php endif; ?>
    <p>
        <?= Html::a('Clear Caches', ['cache'], [
            'class' => 'btn btn-primary',
            'data' => [
                'confirm' => 'Are you sure you want to clear all cache data?',
                'method' => 'post',
            ],
        ]) ?>
        <?= Html::a('Clear Assets', ['assets'],
            ['class' => 'btn btn-primary',
                'data' => [
                    'confirm' => 'Are you sure you want to clear all temporary assets?',
                    'method' => 'post',
                ],
            ]) ?>
        <?= Html::a('Clear Runtime', ['runtime'],
            ['class' => 'btn btn-primary',
                'data' => [
                    'confirm' => 'Are you sure you want to clear all runtime files?',
                    'method' => 'post',
                ],
            ]) ?>
    </p>
</div>

```




```
$config = [
    'id' => 'basic',
    'basePath' => dirname(__DIR__),
    'bootstrap' => ['log'],
    'controllerMap' => [
        'clean' => 'app\cleaner\CleanController',
    ],
    'components' => [
        ...
    ]
    ...
];
```

```
echo Nav::widget([
    'options' => ['class' => 'navbar-nav navbar-right'],
    'items' => [
        ['label' => 'Home', 'url' => ['/site/index']],
        ['label' => 'Cleaner', 'url' => ['/clean/index']],
        ['label' => 'About', 'url' => ['/site/about']],
        ...
    ],
]);
```

```
'controllerMap' => [
    'clean' => 'app\cleaner\CleanController',
    'assetPaths' => [
        '@backend/web/assets',
        '@frontend/web/assets',
    ],
    'runtimePaths' => [
        '@backend/runtime',
        '@frontend/runtime',
        '@console/runtime',
    ],
],
```

### 工作原理...

### 参考

## 创建一个小部件

### 准备

###如何做...

```
<?php
namespace app\widgets;
use yii\base\Widget;
class ChartWidget extends Widget
{
    public $title;
    public $width = 300;
    public $height = 200;
    public $data = [];
    public $labels = [];
    public function run()
    {
        $path = 'http://chart.apis.google.com/chart';
        $query = http_build_query([
            'chtt' => $this->title,
            'cht' => 'pc',
            'chs' => $this->width . 'x' . $this->height,
            'chd' => 't:' . implode(',', $this->data),
            'chds' => 'a',
            'chl' => implode('|', $this->labels),
            'chxt' => 'y',
            'chxl' => '0:|0|' . max($this->data)
        ]);
        $url = $path . '?' . $query;
        return $this->render('chart', [
            'url' => $url,
        ]);
    }
}
```



```
<?php
use yii\helpers\Html;
/* @var $this yii\web\View */
/* @var $url string */
?>
<div class="chart">
    <?= Html::img($url) ?>
</div>
```



```
<?php
namespace app\controllers;
use yii\base\Controller;
class ChartController extends Controller
{
    public function actionIndex()
    {
        return $this->render('index');
    }
}
```


```
<?php
use app\widgets\ChartWidget;
use yii\helpers\Html;
/* @var $this yii\web\View */
$this->title = 'Chart';
$this->params['breadcrumbs'][] = $this->title;
?>
<div class="site-about">
    <h1><?= Html::encode($this->title) ?></h1>
    <?= ChartWidget::widget([
        'title' => 'My Chart Diagram',
        'data' => [
            100 - 32,
            32,
        ],
        'labels' => [
            'Big',
            'Small',
        ],
    ]) ?>
</div>
```

```
<?php
namespace app\commands;
use yii\console\Controller;
use yii\helpers\FileHelper;
/**
 * Removes content of assets and runtime directories.
 */
class CleanController extends Controller
{
    public $assetPaths = ['@app/web/assets'];
    public $runtimePaths = ['@runtime'];
    /**
     * Removes temporary assets.
     */
    public function actionAssets()
    {
        foreach ((array)$this->assetPaths as $path) {
            $this->cleanDir($path);
        }
        $this->stdout('Done' . PHP_EOL);
    }
    /**
     * Removes runtime content.
     */
    public function actionRuntime()
    {
        foreach ((array)$this->runtimePaths as $path) {
            $this->cleanDir($path);
        }
        $this->stdout('Done' . PHP_EOL);
    }
    private function cleanDir($dir)
    {
        $iterator = new
        \DirectoryIterator(\Yii::getAlias($dir));
        foreach($iterator as $sub) {
            if(!$sub->isDot() && $sub->isDir()) {
                $this->stdout('Removed ' . $sub->getPathname()
                    . PHP_EOL);
                FileHelper::removeDirectory($sub->getPathname());
            }
        }
    }
}
```



```
return [
    'id' => 'app-console',
    'basePath' => dirname(__DIR__),
    'bootstrap' => ['log'],
    'controllerNamespace' => 'console\controllers',
    'controllerMap' => [
        'clean' => [
            'class' => 'console\controllers\CleanController',
            'assetPaths' => [
                '@backend/web/assets',
                '@frontend/web/assets',
            ],
            'runtimePaths' => [
                '@backend/runtime',
                '@frontend/runtime',
                '@console/runtime',
            ],
        ],
    ],
// ...
];
```

### 工作原理...

```
public function getHelp()
{
    $out = "Clean command allows you to clean up various temporary data Yii and an application are generating.\n\n";
    return $out . parent::getHelp();
}
```

```
/**
* Removes content of assets and runtime directories.
*/
class CleanController extends Controller
{
    /**
    * Removes temporary assets.
    */
    public function actionAssets() { … }
    * Removes runtime content.
    */
    public function actionRuntime() { … }
}
```

### 参考


## 创建CLI命令

### 准备

### 如何做...

```
<?php
namespace app\models;
use yii\base\Model;
class AgreementForm extends Model
{
    public $accept;
    public function rules()
    {
        return [
            ['accept', 'required'],
            ['accept', 'compare', 'compareValue' => 1,
                'message' => 'You must agree the rules.'],
        ];
    }
    public function attributeLabels()
    {
        return [
            'accept' => 'I completely accept the rules.'
        ];
    }
}
```


```
<?php
namespace app\services;
use Yii;
use yii\web\Cookie;
class AgreementChecker
{
    public function isAllowed()
    {
        return Yii::$app->request->cookies->has('agree');
    }
    public function allowAccess()
    {
        Yii::$app->response->cookies->add(new Cookie([
            'name' => 'agree',
            'value' => 'on',
            'expire' => time() + 3600 * 24 * 90, // 90 days
        ]));
    }
}
```


```
<?php
namespace app\filters;
use app\services\AgreementChecker;
use Yii;
use yii\base\ActionFilter;
class AgreementFilter extends ActionFilter
{
    public function beforeAction($action)
    {
        $checker = new AgreementChecker();
        if (!$checker->isAllowed()) {
            Yii::$app->response->redirect(['/content/agreement'])->send();
            return false;
        }
        return true;
    }
}
```


```
<?php
namespace app\controllers;
use app\filters\AgreementFilter;
use app\models\AgreementForm;
use app\services\AgreementChecker;
use Yii;
use yii\web\Controller;
class ContentController extends Controller
{
    public function behaviors()
    {
        return [
            [
                'class' => AgreementFilter::className(),
                'only' => ['index'],
            ],
        ];
    }
    public function actionIndex()
    {
        return $this->render('index');
    }
    public function actionAgreement()
    {
        $model = new AgreementForm();
        if ($model->load(Yii::$app->request->post()) &&
            $model->validate()) {
            $checker = new AgreementChecker();
            $checker->allowAccess();
            return $this->redirect(['index']);
        } else {
            return $this->render('agreement', [
                'model' => $model,
            ]);
        }
    }
}
```


```
<?php
use yii\helpers\Html;
/* @var $this yii\web\View */
$this->title = 'Content';
$this->params['breadcrumbs'][] = $this->title;
?>
<div class="site-about">
    <h1><?= Html::encode($this->title) ?></h1>
    <div class="well">
        This is our private page.
    </div>
</div>
```


```
<?php
use yii\helpers\Html;
use yii\bootstrap\ActiveForm;
/* @var $this yii\web\View */
/* @var $form yii\bootstrap\ActiveForm */
/* @var $model app\models\AgreementForm */
$this->title = 'User agreement';
$this->params['breadcrumbs'][] = $this->title;
?>
<div class="site-login">
    <h1><?= Html::encode($this->title) ?></h1>
    <p>Please agree with our rules:</p>
    <?php $form = ActiveForm::begin(); ?>
    <?= $form->field($model, 'accept')->checkbox() ?>
    <div class="form-group">
        <?= Html::submitButton('Accept', ['class' => 'btn btn-success']) ?>
        <?= Html::a('Cancel', ['/site/index'], ['class' => 'btn btn-danger']) ?>
    </div>
    <?php ActiveForm::end(); ?>
</div>
```


```
echo Nav::widget([
    'options' => ['class' => 'navbar-nav navbar-right'],
    'items' => [
        ['label' => 'Home', 'url' => ['/site/index']],
        ['label' => 'Content', 'url' => ['/content/index']],
        ['label' => 'About', 'url' => ['/site/about']],
    //...
    ],
]);
```

### 工作原理...

```
class AgreementFilter extends ActionFilter
{
    public function beforeAction($action)
    {
        $checker = new AgreementChecker();
        if (!$checker->isAllowed()) {
            Yii::$app->response->redirect(['/content/agreement'])->send();
            return false;
        }
        return true;
    }
}
```


```
public function behaviors()
{
    return [
        [
            'class' => AgreementFilter::className(),
            'only' => ['index'],
        ],
    ];
}
```

### 参考


## 创建过滤器

### 准备

### 如何做...

### 工作原理...

#### 注意

### 参考

## 创建模块



### 准备

### 如何做...

```
<?php
namespace app\modules\log;
class Module extends \yii\base\Module
{
    public $file = '@runtime/logs/app.log';
}
```


```
<?php
namespace app\modules\log\models;
use yii\base\Object;
class LogRow extends Object
{
    public $time;
    public $ip;
    public $userId;
    public $sessionId;
    public $level;
    public $category;
    public $text;
}
```

```
<?php
namespace app\modules\log\services;
use app\modules\log\models\LogRow;
class LogReader
{
    public function getRows($file)
    {
        $result = [];
        $handle = @fopen($file, "r");
        if ($handle) {
            while (($row = fgets($handle)) !== false) {
                $pattern =
                    '#^' .
                    '(?P<time>\d{4}\-\d{2}\-\d{2}\d{2}:\d{2}:\d{2}) ' .
                    '\[(?P<ip>[^\]]+)\]' .
                    '\[(?P<userId>[^\]]+)\]' .
                    '\[(?P<sessionId>[^\]]+)\]' .
                    '\[(?P<level>[^\]]+)\]' .
                    '\[(?P<category>[^\]]+)\]' .
                    ' (?P<text>.*?)' .
                    '(\$\_(GET|POST|REQUEST|COOKIE|SERVER) = \[)?' .
                    '$#i';
                if (preg_match($pattern, $row, $matches)) {
                    if ($matches['text']) {
                        $result[] = new LogRow([
                            'time' => $matches['time'],
                            'ip' => $matches['ip'],
                            'userId' => $matches['userId'],
                            'sessionId' =>
                                $matches['sessionId'],
                            'level' => $matches['level'],
                            'category' => $matches['category'],
                            'text' => $matches['text'],
                        ]);
                    }
                }
            }
            fclose($handle);
        }
        return array_reverse($result);
    }
}
```


```
<?php
namespace app\modules\log\helpers;
use yii\helpers\ArrayHelper;
use yii\helpers\Html;
class LogHelper
{
    public static function levelLabel($level)
    {
        $classes = [
            'error' => 'danger',
            'warning' => 'warning',
            'info' => 'primary',
            'trace' => 'default',
            'profile' => 'success',
            'profile begin' => 'info',
            'profile end' => 'info',
        ];
        $class = ArrayHelper::getValue($classes, $level,
            'default');
        return Html::tag('span', Html::encode($level), ['class' => 'label-' . $class]);
    }
}
```


```
<?php
namespace app\modules\log\controllers;
use app\modules\log\services\LogReader;
use yii\data\ArrayDataProvider;
use yii\web\Controller;
class DefaultController extends Controller
{
    public function actionIndex()
    {
        $reader = new LogReader();
        $dataProvider = new ArrayDataProvider([
            'allModels' => $reader->getRows($this->getFile()),
        ]);
        return $this->render('index', [
            'dataProvider' => $dataProvider,
        ]);
    }
    private function getFile()
    {
        return \Yii::getAlias($this->module->file);
    }
}
```


```
<?php
use app\modules\log\helpers\LogHelper;
use app\modules\log\models\LogRow;
use yii\grid\GridView;
use yii\helpers\Html;
/* @var $this yii\web\View */
/* @var $dataProvider yii\data\ArrayDataProvider */
$this->title = 'Application log';
$this->params['breadcrumbs'][] = $this->title;
?>
<div class="log-index">
    <h1><?= Html::encode($this->title) ?></h1>
    <?= GridView::widget([
        'dataProvider' => $dataProvider,
        'columns' => [
            [
                'attribute' => 'time',
                'format' => 'datetime',
                'contentOptions' => [
                    'style' => 'white-space: nowrap',
                ],
            ],
            'ip:text:IP',
            'userId:text:User',
            [
                'attribute' => 'level',
                'value' => function (LogRow $row) {
                    return LogHelper::levelLabel($row->level);
                },
                'format' => 'raw',
            ],
            'category',
            'text',
        ],
    ]) ?>
</div>
```


```
$config = [
    'id' => 'basic',
    'basePath' => dirname(__DIR__),
    'bootstrap' => ['log'],
    'modules' => [
        'log' => 'app\modules\log\Module',
    ],
    'components' => [
    ],
    //...
];
```

```
echo Nav::widget([
    'options' => ['class' => 'navbar-nav navbar-right'],
    'items' => [
        ['label' => 'Home', 'url' => ['/site/index']],
        ['label' => 'Log', 'url' => ['/log/default/index']],
        ['label' => 'About', 'url' => ['/site/about']],
        ['label' => 'Contact', 'url' => ['/site/contact']],
        //...
    ],
]);
NavBar::end();
```

### 工作原理...



### 参考


## 创建一个自定义视图渲染器

### 准备

### 如何做...

```
<?php
namespace app\smarty;
use Smarty;
use Yii;
class ViewRenderer extends \yii\base\ViewRenderer
{
    public $cachePath = '@runtime/smarty/cache';
    public $compilePath = '@runtime/smarty/compile';
    /**
     * @var Smarty
     */
    private $smarty;
    public function init()
    {
        $this->smarty = new Smarty();
        $this->smarty->setCompileDir(Yii::getAlias($this->compilePath));
        $this->smarty->setCacheDir(Yii::getAlias($this->cachePath));
        $this->smarty->setTemplateDir([
            dirname(Yii::$app->getView()->getViewFile()),
            Yii::$app->getViewPath(),
        ]);
    }
    public function render($view, $file, $params)
    {
        $templateParams = empty($params) ? null : $params;
        $template = $this->smarty->createTemplate($file, null,
            null, $templateParams, false);
        $template->assign('app', \Yii::$app);
        $template->assign('this', $view);
        return $template->fetch();
    }
}
```


```
'components' => [
    //....
    'view' => [
        'renderers' => [
            'tpl' => [
                'class' => 'app\smarty\ViewRenderer',
            ],
        ],
    ],
    //...
];
```

```
<?php
namespace app\controllers;
use yii\web\Controller;
class SmartyController extends Controller
{
    public function actionIndex()
    {
        return $this->render('index.tpl', [
            'name' => 'Bond',
        ]);
    }
}
```

```
<div class="smarty-index">
    <h1>Smarty Example</h1>
    <p>Hello, {$name}!</p>
</div>
```

```
<?php
namespace yii\base;
abstract class ViewRenderer extends Component
{
    /**
     * Renders a view file.
     *
     * This method is invoked by [[View]] whenever it tries to render a view.
     * Child classes must implement this method to render the given view file.
     *
     * @param View $view the view object used for rendering the file.
     * @param string $file the view file.
     * @param array $params the parameters to be passed to the view file.
     * @return string the rendering result
     */
    abstract public function render($view, $file, $params);
}
```

```
class ViewRenderer extends \yii\base\ViewRenderer
{
    public $cachePath = '@runtime/smarty/cache';
    public $compilePath = '@runtime/smarty/compile';
    private $smarty;
    public function init()
    {
        $this->smarty = new Smarty();
        $this->smarty->setCompileDir(Yii::getAlias($this->compilePath));
        $this->smarty->setCacheDir(Yii::getAlias($this->cachePath));
        $this->smarty->setTemplateDir([
            dirname(Yii::$app->getView()->getViewFile()),
            Yii::$app->getViewPath(),
        ]);
    }
//…
}
```

```
public function render($view, $file, $params)
{
    $templateParams = empty($params) ? null : $params;
    $template = $this->smarty->createTemplate($file, null, null,
        $templateParams, false);
    $template->assign('app', \Yii::$app);
    $template->assign('this', $view);
    return $template->fetch();
}
```

### 工作原理...

### 参考

## 创建一个多语言应用

### 准备

### 如何做...

```
echo Nav::widget([
    'options' => ['class' => 'navbar-nav navbar-right'],
    'items' => [
        ['label' => Yii::t('app/nav', 'Home'), 'url' => ['/site/index']],
        ['label' => Yii::t('app/nav', 'About'), 'url' => ['/site/about']],
        ['label' => Yii::t('app/nav', 'Contact'), 'url' => ['/site/contact']],
        //...
    ],
]);
```


```
$this->title = Yii::t('app', 'Contact');
$this->params['breadcrumbs'][] = $this->title;
```

```
<div class="form-group">
<?= Html::submitButton(Yii::t('app', 'Submit'), ['class' => 'btn btn-primary'']) ?>
</div>
```

```
<p>
<?= Yii::t('app', 'The above error occurred while the Web server was processing your request.') ?>
</p>
```

```
class LoginForm extends Model
{
    ...
    public function attributeLabels()
    {
        return [
            'username' => Yii::t('app/user', 'Username'),
            'password' => Yii::t('app/user', 'Password'),
            'rememberMe' => Yii::t('app/user', 'Remember Me'),
        ];
    }
}
```


```
class ContactForm extends Model
{
    ...
    public function attributeLabels()
    {
        return [
            'name' => Yii::t('app/contact', 'Name'),
            'email' => Yii::t('app/contact', 'Email'),
            'subject' => Yii::t('app/contact', 'Subject'),
            'body' => Yii::t('app/contact', 'Body'),
            'verifyCode' => Yii::t('app', 'Verification Code'),
        ];
    }
}
```


```
./yii message/config-template config/messages.php
```

```
<?php
return [
    'sourcePath' => '@app',
    'languages' => ['de', 'fr'],
    'translator' => 'Yii::t',
    'sort' => false,
    'removeUnused' => false,
    'markUnused' => true,
    'only' => ['*.php'],
    'except' => [
        '.svn',
        '.git',
        '.gitignore',
        '.gitkeep',
        '.hgignore',
        '.hgkeep',
        '/messages',
        '/vendor',
    ],
    'format' => 'php',
    'messagePath' => '@app/messages',
    'overwrite' => true,
    'ignoreCategories' => [
        'yii',
    ],
];
```


```
./yii message config/messages.php
```


```
messages
├── de
│   ├── app
│   │   ├── contact.php
│   │   ├── nav.php
│   │   └── user.php
│   └── app.php
└── fr
    ├── app
    │   ├── contact.php
    │   ├── nav.php
    │   └── user.php
    └── app.php
```

```
<?php
...
return [
    'Body' => '',
    'Email' => '',
    'Name' => '',
    'Subject' => '',
];
```

```
<?php
...
return [
    'Password' => 'Passwort',
    'Remember Me' => 'Erinnere dich an mich',
    'Username' => 'Benutzername',
];
```


```
$config = [
    'id' => 'basic',
    'basePath' => dirname(__DIR__),
    'bootstrap' => ['log'],
    'components' => [
        //…
        'i18n' => [
            'translations' => [
                'app*' => [
                    'class' => 'yii\i18n\PhpMessageSource',
                    'sourceLanguage' => 'en-US',
                ],
            ],
        ],
        'db' => require(__DIR__ . '/db.php'),
    ],
    'params' => $params,
];
```

```
$config = [
    'id' => 'basic',
    'language' => 'de',
    'basePath' => dirname(__DIR__),
    'bootstrap' => ['log'],
    ...
];
```

### 工作原理...

```
$config = [
    'id' => 'basic',
    'language' => 'de',
    ...
];
```

```
Yii::$app->language = 'fr';
```


```
<?php
namespace app\bootstrap;
use yii\base\BootstrapInterface;
class LanguageBootstrap implements BootstrapInterface
{
    public function bootstrap($app)
    {
        if (!$app->user->isGuest) {
            $app->language = $app->user->identity->lang;
        }
    }
}
```


```
$config = [
    'id' => 'basic',
    'basePath' => dirname(__DIR__),
    'bootstrap' => ['log', 'app'bootstrap\LanguageBoostrap'],
    ...
];
```



### 参考


## 制作可发布的扩展

### 准备

### 如何做...


### 工作原理...

### 参考