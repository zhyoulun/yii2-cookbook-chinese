# 第二章 路由，控制器和视图

## 介绍

## 配置URL规则

### 准备

```
<?php
namespace app\controllers;
use yii\helpers\Html;
use yii\web\Controller;
class TestController extends Controller
{
    public function actionIndex()
    {
        return $this->renderContent(Html::tag('h2',
            'Index action'
        ));
    }
    public function actionPage($alias)
    {
        return $this->renderContent(Html::tag('h2',
            'Page is '. Html::encode($alias)
        ));
    }
}
```


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



```
'components' => [
    // ..
    'urlManager' => [
        'enablePrettyUrl' => true,
        'rules' => [
            'home' => 'test/index',
            '<alias:about>' => 'test/page',
            'page/<alias>' => 'test/page',
        ]
    ],
    // ..
],
```

```
‰/home
‰/about
‰/page/about
/page/test
```


![](../images/201.png)


![](../images/202.png)

### 工作原理...

```
'home' => 'test/index',
```


### 更多...

```
'page/<alias>' => test/page',
```

```
TestController::actionPage($alias)
```

```
'<alias:about>' => test/page'
```

### 参考

## 生成URLs

### 准备

```
'urlManager' => array(
    'enablePrettyUrl' => true,
    'showScriptName' => false,
),
```

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

```
<?php
namespace app\controllers;
use yii\web\Controller;
class BlogController extends Controller
{
    public function actionIndex()
    {
        return $this->render('index');
    }
    public function actionRssFeed($param)
    {
        return $this->renderContent('This is RSS feed for our blog and ' . $param);
    }
    public function actionArticle($alias)
    {
        return $this->renderContent('This is an article with alias ' . $alias);
    }
    public function actionList()
    {
        return $this->renderContent('Blog\'s articles here');
    }
    public function actionHiTech()
    {
        return $this->renderContent('Just a test of action which contains more than one words in the name') ;
    }
}
```


```
<?php
namespace app\controllers;
use Yii;
use yii\web\Controller;
class TestController extends Controller
{
    public function actionUrls()
    {
        return $this->render('urls');
    }
}
```


```
<?php
use yii\helpers\Url;
use yii\helpers\Html;
?>
    <h1>Generating URLs</h1>
    <h3>Generating a link with URL to <i>blog</i> controller and
        <i>article</i> action with alias as param</h3>
<?= Html::a('Link Name', ['blog/article', 'alias' => 'someAlias']); ?>
    <h3>Current url</h3>
<?=Url::to('')?>
    <h3>Current Controller, but you can specify an action</h3>
<?=Url::toRoute(['view', 'id' => 'contact']);?>
    <h3>Current module, but you can specify controller and
        action</h3>
<?= Url::toRoute('blog/article')?>
    <h3>An absolute route to blog/list </h3>
<?= Url::toRoute('/blog/list')?>
    <h3> URL for <i>blog</i> controller and action <i>HiTech</i>
    </h3>
<?= Url::toRoute('blog/hi-tech')?>
    <h3>Canonical URL for current page</h3>
<?= Url::canonical()?>
<h3>Getting a home URL</h3>
<?= Url::home()?>
<h3>Saving a URL of the current page and getting it for
    re-use</h3>
<?php Url::remember()?>
<?=Url::previous()?>
<h3>Creating URL to <i>blog</i> controller and <i>rss-feed</i>
    action while URL helper isn't available</h3>
<?=Yii::$app->urlManager->createUrl(['blog/rss-feed', 'param' => 'someParam'])?>
<h3>Creating an absolute URL to <i>blog</i> controller and
    <i>rss-feed</i></h3>
<p>It's very useful for emails and console applications</p>
<?=Yii::$app->urlManager->createAbsoluteUrl(['blog/rss-feed', 'param' => 'someParam'])?>
```


![](../images/203.png)

### 工作原理...

```
<?= Html::a('Link Name', ['blog/article', 'alias' => 'someAlias']); ?>
```


```
<?=Yii::$app->urlManager->createUrl(['blog/rss-feed', 'param' => 'someParam'])?>
<?=Yii::$app->urlManager->createAbsoluteUrl(['blog/rss-feed','param' => 'someParam'])?>
```


### 更多...


### 参考

## 在URL规则中使用正则表达式

### 准备


```
<?php
namespace app\controllers;
use yii\helpers\Html;
use yii\web\Controller;
class PostController extends Controller
{
    public function actionView($alias)
    {
        return $this->renderContent(Html::tag('h2', 'Showing post with alias ' . Html::encode($alias)
        ));
    }
    public function actionIndex($type = 'posts', $order = 'DESC')
    {
        return $this->renderContent(Html::tag('h2', 'Showing ' . Html::encode($type) . ' ordered ' . Html::encode($order)));
    }
    public function actionHello($name)
    {
        return $this->renderContent(Html::tag('h2', 'Hello, ' . Html::encode($name) . '!'));
    }
}
```

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


```
'components' => [
    // ..
    'urlManager' => [
        'enablePrettyUrl' => true,
        'rules' => [
            'post/<alias:[-a-z]+>' => 'post/view',
            '<type:(archive|posts)>' => 'post/index',
            '<type:(archive|posts)>/<order:(DESC|ASC)>' => 'post/index',
            'sayhello/<name>' => 'post/hello',
        ]
    ],
    // ..
],
```

![](../images/204.png)


![](../images/205.png)


![](../images/206.png)

### 工作原理...


```
'post/<alias:[-a-z]+>' => 'post/view',
```

```
'(posts|archive)' => 'post/index',
'(posts|archive)/<order:(DESC|ASC)>' => 'post/index',
```


```
'sayhello/<name>' => 'post/hello',
```

![](../images/207.png)

### 更多...


### 参考

## 使用一个基础控制器

### 准备

### 如何做...

```
<?php
namespace app\components;
use Yii;
use yii\web\Controller;
use yii\filters\AccessControl;
class BaseController extends Controller
{
    public function actions()
    {
        return [
            'error' => ['class' => 'yii\web\ErrorAction'],
        ];
    }
    public function behaviors()
    {
        return [
            'access' => [
                'class' => AccessControl::className(),
                'rules' => [
                    [
                        'allow' => true,
                        'actions' => 'error'
                    ],
                    [
                        'allow' => true,
                        'roles' => ['@'],
                    ],
                ],
            ]
        ];
    }
}
```


![](../images/208.png)

```
<?php
namespace app\controllers;
class TestController extends \app\components\BaseController
{
    public function actionIndex()
    {
        return $this->render('index');
    }
}
```

### 工作原理...

### 更多...

```
<?php
namespace app\controllers;
use yii\helpers\ArrayHelper;
use app\components\BaseController;
class TestController extends BaseController
{
    public function actions()
    {
        return ArrayHelper::merge(parent::actions(), [
            'page' => [
                'class' => 'yii\web\ViewAction',
            ],
        ]);
    }
    public function behaviors()
    {
        $behaviors = parent::behaviors();
        $rules = $behaviors['access']['rules'];
        $rules = ArrayHelper::merge(
            $rules,
            [
                [
                    'allow' => true,
                    'actions' => ['page']
                ]
            ]
        );
        $behaviors['access']['rules'] = $rules;
        return $behaviors;
    }
    public function actionIndex()
    {
        return $this->render('index');
    }
}
```

## 使用独立动作

### 准备

```
./yii migrate/create create_post_table
```


```
<?php
use yii\db\Schema;
use yii\db\Migration;
class m150719_152435_create_post_table extends Migration
{
    const TABLE_NAME = '{{%post}}';
    public function up()
    {
        $tableOptions = null;
        if ($this->db->driverName === 'mysql') {
            $tableOptions = 'CHARACTER SET utf8 COLLATE utf8_general_ci ENGINE=InnoDB';
        }
        $this->createTable(self::TABLE_NAME, [
            'id' => Schema::TYPE_PK,
            'title' => Schema::TYPE_STRING.'(255) NOT NULL',
            'content' => Schema::TYPE_TEXT.' NOT NULL',
        ], $tableOptions);
        for ($i = 1; $i < 7; $i++) {
            $this->insert(self::TABLE_NAME, [
                'title' => 'Test article #'.$i,
                'content' => 'Lorem ipsum dolor sit amet, consectetur adipiscing elit. '
                    .'Sed sit amet mauris est. Sed at dignissim dui. '
                    .'Phasellus arcu massa, facilisis a fringilla sit amet, '
                    .'rhoncus ut enim.',
            ]);
        }
    }
    public function down()
    {
        $this->dropTable(self::TABLE_NAME);
    }
}
```


```
./yii migrate up
```

### 如何做...

```
<?php
namespace app\actions;
use Yii;
use yii\base\Action;
class CreateAction extends Action
{
    public $modelClass;
    public function run()
    {
        $model = new $this->modelClass();
        if ($model->load(Yii::$app->request->post()) && $model->save()) {
            $this->controller->redirect(['view', 'id' => $model->getPrimaryKey()]);
        } else {
            return $this->controller->render('//crud/create', [
                'model' => $model
            ]);
        }
    }
}
```


```
<?php
namespace app\actions;
use yii\base\Action;
use yii\web\NotFoundHttpException;
class DeleteAction extends Action
{
    public $modelClass;
    public function run($id)
    {
        $class = $this->modelClass;
        if (($model = $class::findOne($id)) === null) {
            throw new NotFoundHttpException('The requested page does not exist.');
        }
        $model->delete();
        return $this->controller->redirect(['index']);
    }
}
```



```
<?php
namespace app\actions;
use yii\base\Action;
use yii\data\Pagination;
class IndexAction extends Action
{
    public $modelClass;
    public $pageSize = 3;
    public function run()
    {
        $class = $this->modelClass;
        $query = $class::find();
        $countQuery = clone $query;
        $pages = new Pagination([
            'totalCount' => $countQuery->count(),
        ]);
        $pages->setPageSize($this->pageSize);
        $models = $query->offset($pages->offset)
            ->limit($pages->limit)
            ->all();
        return $this->controller->render('//crud/index', [
            'pages' => $pages,
            'models' => $models
        ]);
    }
}
```


```
<?php
namespace app\actions;
use yii\base\Action;
use yii\web\NotFoundHttpException;
class ViewAction extends Action
{
    public $modelClass;
    public function run($id)
    {
        $class = $this->modelClass;
        if (($model = $class::findOne($id)) === null) {
            throw new NotFoundHttpException('The requested page does not exist.');
        }
        return $this->controller->render('//crud/view', [
            'model' => $model
        ]);
    }
}
```


```
<?php
use yii\helpers\Html;
use yii\widgets\ActiveForm;
/*
* @var yii\web\View $this
*/
?>
    <h1><?= Yii::t('app', 'Create post'); ?></h1>
<?php $form = ActiveForm::begin();?>
<?php $form->errorSummary($model); ?>
<?= $form->field($model, 'title')->textInput() ?>
<?= $form->field($model, 'content')->textarea() ?>
<?= Html::submitButton(Yii::t('app', 'Create'), ['class' =>
    'btn btn-primary']) ?>
<?php ActiveForm::end(); ?>
```


```
<?php
use yii\widgets\LinkPager;
use yii\helpers\Html;
use yii\helpers\Url;
/*
* @var yii\web\View $this
* @var yii\data\Pagination $pages
* @var array $models
*/
?>
<h1>Posts</h1>
<?= Html::a('+ Create a post', Url::toRoute('post/create')); ?>
<?php foreach ($models as $model):?>
<h3><?= Html::encode($model->title);?></h3>
<p><?= Html::encode($model->content);?></p>
    <p>
        <?= Html::a('view', Url::toRoute(['post/view', 'id' =>
            $model->id]));?> |
        <?= Html::a('delete', Url::toRoute(['post/delete', 'id'
        => $model->id]));?>
    </p>
<?php endforeach; ?>
<?= LinkPager::widget([
    'pagination' => $pages,
]); ?>
```


```
<?php
use yii\helpers\Html;
use yii\helpers\Url;
/*
* @var yii\web\View $this
* @var app\models\Post $model
*/
?>
<p><?= Html::a('< back to posts', Url::toRoute('post/index'));
    ?></p>
<h2><?= Html::encode($model->title);?></h2>
<p><?= Html::encode($model->content);?></p>
```


![](../images/209.png)

### 工作原理...


### 参考

## 创建一个自定义过滤器

### 准备

### 如何做...

```
<?php
namespace app\controllers;
use app\components\CustomFilter;
use yii\helpers\Html;
use yii\web\Controller;
class TestController extends Controller
{
    public function behaviors()
    {
        return [
            'access' => [
                'class' => CustomFilter::className(),
            ],
        ];
    }
    public function actionIndex()
    {
        return $this->renderContent(Html::tag('h1',
            'This is a test content'
        ));
    }
}
```


```
<?php
namespace app\components;
use Yii;
use yii\base\ActionFilter;
use yii\web\HttpException;
class CustomFilter extends ActionFilter
{
    const WORK_TIME_BEGIN = 10;
    const WORK_TIME_END = 18;
    protected function canBeDisplayed()
    {
        $hours = date('G');
        return $hours >= self::WORK_TIME_BEGIN && $hours <= self::WORK_TIME_END;
    }
    public function beforeAction($action)
    {
        if (!$this->canBeDisplayed())
        {
            $error = 'This part of website works from '
                . self::WORK_TIME_BEGIN . ' to '
                . self::WORK_TIME_END . ' hours.';
            throw new HttpException(403, $error);
        }
        return parent::beforeAction($action);
    }
    public function afterAction($action, $result)
    {
        if (Yii::$app->request->url == '/test/index') {
            Yii::trace("This is the index action");
        }
        return parent::afterAction($action, $result);
    }
}
```


![](../images/210.png)

### 工作原理...

```
public function behaviors()
{
return [
    'access' => [
        'class' => CustomFilter::className(),
    ],
];
}
```


![](../images/211.png)

### 参考

## 展示静态页面

### 准备

### 如何做...


```
<?php
namespace app\controllers;
use yii\web\Controller;
class TestController extends Controller
{
    public function actions()
    {
        return [
            'page' => [
                'class' => 'yii\web\ViewAction',
            ]
        ];
    }
}
```



```
<h1>Index</h1>
content of index file
Contact.php content is:
<h2>Contacts</h2>
<p>Our contact: contact@localhost</p>
```


![](../images/212.png)

### 工作原理...


### 更多...


#### 关于ViewAction

#### 配置URL规则

```
'<view:about>' => 'test/page'
```

![](../images/213.png)

### 参考

## 使用flash消息

### 准备

### 如何做...

```
<?php
namespace app\controllers;
use Yii;
use yii\web\Controller;
use yii\filters\AccessControl;
class TestController extends Controller
{
    public function behaviors()
    {
        return [
            'access' => [
                'class' => AccessControl::className(),
                'rules' => [
                    [
                        'allow' => true,
                        'roles' => ['@'],
                        'actions' => ['user']
                    ],
                    [
                        'allow' => true,
                        'roles' => ['?'],
                        'actions' => ['index', 'success',
                            'error']
                    ],
                ],
                'denyCallback' => function ($rule, $action) {
                    Yii::$app->session->setFlash('error',
                        'This section is only for registered users.');
                    $this->redirect(['index']);
                },
            ],
        ];
    }
    public function actionUser()
    {
        return $this->renderContent('user');
    }
    public function actionSuccess()
    {
        Yii::$app->session->setFlash('success', 'Everything went fine!');
        $this->redirect(['index']);
    }
    public function actionError()
    {
        Yii::$app->session->setFlash('error', 'Everything went wrong!');
        $this->redirect(['index']);
    }
    public function actionIndex()
    {
        return $this->render('index');
    }
}
```


```
<?php
use yii\bootstrap\Alert;
?>
<?php if (Yii::$app->session->hasFlash('success')):?>
    <?= Alert::widget([
        'options' => ['class' => 'alert-success'],
        'body' => Yii::$app->session->getFlash('success'),
    ]);?>
<?php endif ?>
<?php if (Yii::$app->session->hasFlash('error')) :?>
    <?= Alert::widget([
        'options' => ['class' => 'alert-danger'],
        'body' => Yii::$app->session->getFlash('error'),
    ]);?>
<?php endif; ?>
```


```
<?php
/* @var $this yii\web\View */
?>
<?= $this->render('//common/alert') ?>
<h2>Guest page</h2>
<p>There's a content of guest page</p>
```


```
<?php
/* @var $this yii\web\View */
?>
<?= $this->render('//common/alert') ?>
<h2>User page</h2>
<p>There's a content of user page</p>
```


![](../images/214.png)

![](../images/215.png)


![](../images/216.png)

### 工作原理...

### 更多...

#### getAllFlashes()方法

```
$flashes = Yii::$app->session->getAllFlashes();
<?php foreach ($flashes as $key => $message): ?>
<?= Alert::widget([
    'options' => ['class' => 'alert-info'],
    'body' => $message,
]);
?>
<?php endforeach; ?>
```

#### removeAllFlashes()方法

```
Yii::$app->session->removeAllFlashes();
```

#### removeFlash()方法

```
Yii::$app->session->removeFlash('success');
```


### 参考

## 在一个视图中使用控制器上下文

### 准备

### 如何做...


```
<?php
namespace app\controllers;
use yii\web\Controller;
class ViewController extends Controller
{
    public $pageTitle;
    public function actionIndex()
    {
        $this->pageTitle = 'Controller context test';
        return $this->render('index');
    }
    public function hello()
    {
        if (!empty($_GET['name'])) {
            echo 'Hello, ' . $_GET['name'] . '!';
        }
    }
}
```


```
<h1><?= $this->context->pageTitle ?></h1>
<p>Hello call. <?php $this->context->hello() ?></p>
```


![](../images/217.png)

### 工作原理...


### 更多...

## 部分复用视图

### 准备

### 如何做...

```
<?php
namespace app\controllers;
use yii\web\Controller;
class BlogController extends Controller
{
    public function actionIndex()
    {
        $posts = [
            [
                'title' => 'First post',
                'content' => 'There\'s an example of reusing views with partials.',
            ],
            [
                'title' => 'Second post',
                'content' => 'We use twitter widget.'
            ],
        ];
        return $this->render('index', [
            'posts' => $posts
        ]);
    }
}
```


```
<?php
/* @var $this \yii\web\View */
/* @var $widget_id integer */
/* @var $screen_name string */
?>
    <script>!function(d,s,id){var
            js,fjs=d.getElementsByTagName(s)[0],p=/^http:/.test(d.location)?
                'http':'https';if(!d.getElementById(id)){js=d.createElement(s);j
            s.id=id;js.src=p+"://platform.twitter.com/
            widgets.js";fjs.parentNode.insertBefore(js,fjs);}}(document,"scr
            ipt","twitter-wjs");</script>
<?php if ($widget_id && $screen_name): ?>
    <a class="twitter-timeline"
       data-widget-id="<?= $widget_id?>"
       href="https://twitter.com/<?= $screen_name?>"
       height="300">
        Tweets by @<?= $screen_name?>
    </a>
<?php endif;?>
```


```
<?php
/* @var $category string */
/* @var $posts array */
/* @var $this \yii\web\View */
?>
<div class="row">
    <div class="col-xs-7">
        <h1>Posts</h1>
        <hr>
        <?php foreach ($posts as $post): ?>
            <h3><?= $post['title']?></h3>
            <p><?= $post['content']?></p>
        <?php endforeach;?>
    </div>
    <div class="col-xs-5">
        <?= $this->render('//common/twitter', [
            'widget_id' => '620531418213576704',
            'screen_name' => 'php_net',
        ]);?>
    </div>
</div>
```


```
<?php
use yii\helpers\Html;
/* @var $this yii\web\View */
$this->title = 'About';
?>
<div class="col-xs-7">
    <h1><?= Html::encode($this->title) ?></h1>
    <p>
        This is the About page. You may modify this page.
    </p>
</div>
<div class="col-xs-5">
    <?= $this->render('//common/twitter', [
        'widget_id' => '620526086343012352',
        'screen_name' => 'yiiframework'
    ]);?>
</div>
```


![](../images/218.png)

![](../images/219.png)


### 工作原理...


### 更多...

## 使用blocks

### 准备

### 如何做...


```
<?php if(!empty($this->blocks['beforeContent'])) echo $this->blocks['beforeContent']; ?>
```


```
<footer class="footer">
    <div class="container">
        <?php if (!empty($this->blocks['footer'])):
            echo $this->blocks['footer'] ?>
        <?php else: ?>
            <p class="pull-left">&copy; My Company <?= date('Y') ?></p>
            <p class="pull-right"><?= Yii::powered() ?></p>
        <?php endif; ?>
    </div>
</footer>
```


```
public function actionBlocks()
{
    return $this->render('blocks');
}
```


```
<?php
use \yii\Helpers\Html;
/* @var $this \yii\web\View */
?>
<?php $this->beginBlock('beforeContent');
echo Html::tag('pre', 'Your IP is ' . Yii::$app->request->userIP);
$this->endBlock(); ?>
<?php $this->beginBlock('footer');
echo Html::tag('h3', 'My custom footer block');
$this->endBlock(); ?>
<h1>Blocks usage example</h1>
```

![](../images/220.png)

### 工作原理...

### 更多...

## 使用装饰器

### 准备

### 如何做...

```
<div class="quote">
    <h2>&ldquo;<?= $content?>&rdquo;, <?= $author?></h2>
</div>
```


```
<?php
use yii\widgets\ContentDecorator;
/* @var */
?>
<?php ContentDecorator::begin([
        'viewFile' => '@app/views/decorators/quote.php',
        'view' => $this,
        'params' => ['author' => 'S. Freud']
    ]
);?>
    Time spent with cats is never wasted.
<?php ContentDecorator::end();?>
```

![](../images/221.png)

### 工作原理...

### 参考

## 定义多个布局

### 准备

### 如何做...


```
<?php $this->beginContent('//layouts/main')?>
    <div>
        <?= $content ?>
    </div>
    <div class="sidebar tags">
        <ul>
            <li><a href="#php">PHP</a></li>
            <li><a href="#yii">Yii</a></li>
        </ul>
    </div>
    <div class="sidebar links">
        <ul>
            <li><a href="http://yiiframework.com/">
                    Yiiframework</a></li>
            <li><a href="http://php.net/">PHP</a></li>
        </ul>
    </div>
<?php $this->endContent()?>
```

```
<?php
/* @var $this yii\web\View */
?>
<?php $this->beginContent('@app/views/layouts/main.php'); ?>
    <div class="container">
        <div class="col-xs-8">
            <?= $content ?>
        </div>
        <div class="col-xs-4">
            <h4>Table of contents</h4>
            <ol>
                <li><a href="#intro">Introduction</a></li>
                <li><a href="#quick-start">Quick start</a></li>
                <li>..</li>
            </ol>
        </div>
    </div>
<?php $this->endContent() ?>
```

```
<h1>Title</h1>
<p>Lorem ipsum dolor sit amet, consectetur adipisicing elit,
sed do eiusmod tempor incididunt ut labore et dolore magna
aliqua. Ut enim ad minim veniam, quis nostrud exercitation
ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis
aute irure dolor in reprehenderit in voluptate velit esse
cillum dolore eu fugiat nulla pariatur.</p>
```



```
<?php
namespace app\controllers;
use yii\web\Controller;
class BlogController extends Controller
{
    public $layout = 'blog';
    public function actionIndex()
    {
        return $this->render('//site/content');
    }
}
```



```
<?php
namespace app\controllers;
use yii\web\Controller;
class ArticleController extends Controller
{
    public $layout = 'articles';
    public function actionIndex()
    {
        return $this->render('//site/content');
    }
}
```



```
<?php
namespace app\controllers;
use yii\web\Controller;
class PortfolioController extends Controller
{
    public function actionIndex()
    {
        return $this->render('//site/content');
    }
}
```

![](../images/222.png)

![](../images/223.png)

![](../images/224.png)

### 工作原理...

### 参考

## 页码和数据排序

### 准备

### 如何做...


```
<?php
namespace app\controllers;
use app\models\Film;
use yii\web\Controller;
use yii\data\Pagination;
use yii\data\Sort;
class FilmController extends Controller
{
    public function actionIndex()
    {
        $query = Film::find();
        $countQuery = clone $query;
        $pages = new Pagination(['totalCount' => $countQuery->count()]);
        $pages->pageSize = 5;
        $sort = new Sort([
            'attributes' => [
                'title',
                'rental_rate'
            ]
        ]);
        $models = $query->offset($pages->offset)
            ->limit($pages->limit)
            ->orderBy($sort->orders)
            ->all();
        return $this->render('index', [
            'models' => $models,
            'sort' => $sort,
            'pages' => $pages
        ]);
    }
}
```


```
<?php
use yii\widgets\LinkPager;
/**
 * @var \app\models\Film $models
 * @var \yii\web\View $this
 * @var \yii\data\Pagination $pages
 * @var \yii\data\Sort $sort
 */
?>
    <h1>Films List</h1>
    <p><?=$sort->link('title')?> |
        <?=$sort->link('rental_rate')?></p>
<?php foreach ($models as $model): ?>
    <div class="list-group">
        <h4 class="list-group-item-heading"> <?=$model->title ?>
            <label class="label label-default">
                <?=$model->rental_rate ?>
            </label>
        </h4>
        <p class="list-group-item-text"><?=$model->description
            ?></p>
    </div>
<?php endforeach ?>
<?=LinkPager::widget([
    'pagination' => $pages
]); ?>
```

![](../images/225.png)

### 工作原理...

### 参考


