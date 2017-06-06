# 第十一章 测试

在本章中，我们将会讨论如下话题：

- 使用Codeception测试应用
- 使用PHPUnit做单元测试
- 使用Atoum做单元测试
- 使用Behat做单元测试

## 介绍

在本章中，你将会学习如何如何使用最好技术用于测试，例如Codeception，PHPUnit，Atoum和Behat。你将会看到如何写简单的测试和如何在你的应用中避免拟合错误。

## 使用Codeception测试应用

默认情况下，基础和高级Yii2应用skeletons使用*Codeception*作为一个测试框架。Codeception支持写单元，函数，以及接受box之外的测试。对于单元测试，它使用PHPUnit测试框架，它将被在下个小节中讨论。

### 准备

1. 按照官方指南[http://www.yiiframework.com/doc-2.0/guide-start-installation.html](http://www.yiiframework.com/doc-2.0/guide-start-installation.html)的描述，使用Composer包管理器创建一个新`yii2-app-basic`的应用。

**注意**：如果你用的是基础应用的2.0.9版本（或者更早），只需要手动升级`tests`文件夹，并添加`config/test.php`，`config/test_db.php`和`web/index-test.php`文件。此外你需要复制`composer.json`文件的`require`和`require-dev`部分，并运行`composer update`。

2. 复制并应用如下migration：

```
<?php
use yii\db\Migration;
class m160309_070856_create_post extends Migration
{
    public function up()
    {
        $this->createTable('{{%post}}', [
            'id' => $this->primaryKey(),
            'title' => $this->string()->notNull(),
            'text' => $this->text()->notNull(),
            'status' => $this->smallInteger()->notNull()->defaultValue(0),
        ]);
    }
    public function down()
    {
        $this->dropTable('{{%post}}');
    }
}
```

3. 创建`Post`模型：

```
<?php
namespace app\models;
use Yii;
use yii\db\ActiveRecord;
/**
 * @property integer $id
 * @property string $title
 * @property string $text
 * @property integer $status
 * @property integer $created_at
 * @property integer $updated_at
 */
class Post extends ActiveRecord
{
    const STATUS_DRAFT = 0;
    const STATUS_ACTIVE = 1;
    public static function tableName()
    {
        return '{{%post}}';
    }
    public function rules()
    {
        return [
            [['title', 'text'], 'required'],
            [['text'], 'string'],
            ['status', 'in', 'range' => [self::STATUS_DRAFT,
                self::STATUS_ACTIVE]],
            ['status', 'default', 'value' =>
                self::STATUS_DRAFT],
            [['title'], 'string', 'max' => 255],
        ];
    }
    public function behaviors()
    {
        return [
            TimestampBehavior::className(),
        ];
    }
    public static function getStatusList()
    {
        return [
            self::STATUS_DRAFT => 'Draft',
            self::STATUS_ACTIVE => 'Active',
        ];
    }
    public function publish()
    {
        if ($this->status == self::STATUS_ACTIVE) {
            throw new \DomainException('Post is already published.');
        }
        $this->status = self::STATUS_ACTIVE;
    }
    public function draft()
    {
        if ($this->status == self::STATUS_DRAFT) {
            throw new \DomainException('Post is already drafted.');
        }
        $this->status = self::STATUS_DRAFT;
    }
}
```

4. 生成CRUD：

![](../images/a1101.png)

5. 此外，在`views/admin/posts/_form.php`文件中为`status`字段添加状态下拉菜单，以及提交按钮的名称：

```
<div class="post-form">
    <?php $form = ActiveForm::begin(); ?>
    <?= $form->field($model, 'title')->textInput(['maxlength'
    => true]) ?>
    <?= $form->field($model, 'text')->textarea(['rows' => 6]) ?>
<?= $form->field($model, 'status')->dropDownList(Post::getStatusList()) ?>
    <div class="form-group">
        <?= Html::submitButton($model->isNewRecord ? 'Create' :
            'Update', [
            'class' => $model->isNewRecord ? 'btn btn-success' : 'btn btn-primary',
            'name' => 'submit-button',
        ]) ?>
    </div>
    <?php ActiveForm::end(); ?>
</div>
```

6. 现在检查控制器是否工作：

![](../images/a1102.png)

创建任意示例帖子。

### 如何做...

#### 为测试做准备

跟着如下步骤来为测试做准备：

1. 创建`yii2_basic_tests`或者其它测试数据库，通过应用migration来更新它：

```
tests/bin/yii migrate
```

这个命令需要在测试文件夹中运行。你可以在配置文件`/config/test_db.php`中指定你的测试数据库选项。

2. Codeception为我们的测试套件使用自动生成的Actor类。使用下面的命令创建他们：

```
composer exec codecept build
```

#### 运行单元和功能测试

我们可以立即运行任何类型的应用测试：

```
# run all available tests
composer exec codecept run

# run functional tests
composer exec codecept run functional

# run unit tests
composer exec codecept run unit
```

结果是，你可以查看测试报告如下所示：

![](../images/a1103.png)

#### 获取覆盖报告

你可以为你的代码获取代码覆盖率报告。默认情况下，代码覆盖率在配置文件`tests/codeception.yml`中是禁用的；你需要取消必要的注释，从而可以收集代码覆盖率：

```
coverage:
    enabled: true
    whitelist:
        include:
            - models/*
            - controllers/*
            - commands/*
            - mail/*
    blacklist:
        include:
            - assets/*
            - config/*
            - runtime/*
            - vendor/*
            - views/*
            - web/*
            - tests/*
```

你需要安装XDebug PHP扩展，这可以从[https://xdebug.org](https://xdebug.org)获取。例如，在Ubuntu或者Debian上，你可以在控制台中输入如下命令：

```
sudo apt-get install php5-xdebug
```

在Windows上，你必须打开`php.ini`文件，并添加到你PHP安装路径的自定义代码：

```
[xdebug]
zend_extension_ts=C:/php/ext/php_xdebug.dll
```

或者，如果你使用的是非线程安全的版本，输入如下命令：

```
[xdebug]
zend_extension=C:/php/ext/php_xdebug.dll
```

最后，你可以运行测试，并使用如下命令收集覆盖率报告：

```
#collect coverage for all tests
composer exec codecept run --coverage-html

#collect coverage only for unit tests
composer exec codecept run unit --coverage-html

#collect coverage for unit and functional tests
composer exec codecept run functional,unit --coverage-html
```

你可以在终端上看到文本的代码覆盖率输出：

```
Code Coverage Report:
    2016-03-31 08:13:05
Summary:
    Classes: 20.00% (1/5)
    Methods: 40.91% (9/22)
    Lines: 30.65% (38/124)
\app\models::ContactForm
    Methods: 33.33% ( 1/ 3) Lines: 80.00% ( 12/ 15)
\app\models::LoginForm
    Methods: 100.00% ( 4/ 4) Lines: 100.00% ( 18/ 18)
\app\models::User
    Methods: 57.14% ( 4/ 7) Lines: 53.33% ( 8/ 15)
Remote CodeCoverage reports are not printed to console

HTML report generated in coverage
```

此外，你可以在`tests/codeception/_output/coverage`路径中看到HTML格式的报告：

![](../images/a1104.png)

你可以点击任何类，并分析代码的哪些行在测试运行过程中还没有被执行到。

#### 运行验收测试

在验收测试中，你可以使用基于Curl的PhpBrowser作为请求服务器。它可以帮助你检查网站控制器，并解析HTTP和HTML响应代码。但是如果你希望测试你的CSS或者JavaScript行为，你必须使用真正的浏览器。

Selenium服务器是一个交互式的工具，它集成到了Firefox以及其它浏览器中，可以让你打开站点页面，并模拟人类动作。

为了使用真正的浏览器，我们必须安装Selenium服务器：

1. 需要完整版的Codeception包，而不是基础版的：

```
composer require --dev codeception/codeception
composer remove --dev codeception/base
```

2. 下载如下软件：

- 安装Mozilla Firefox浏览器，[https://www.mozilla.org](https://www.mozilla.org)
- 安装jre（Java运行时环境），[https://www.java.com/en/download/](https://www.java.com/en/download/)
- 下载Selenium独立服务器，[http://www.seleniumhq.org/download/](http://www.seleniumhq.org/download/)
- 下载Geckodriver，[https://github.com/mozilla/geckodriver/releases](https://github.com/mozilla/geckodriver/releases)

3. 在新的控制台窗口中启动带有driver的服务器：

```
java -jar -Dwebdriver.gecko.driver=~/geckodriver ~/selenium-server-standalone-x.xx.x.jar
```

4. 复制`tests/acceptance.suite.yml.example`到`tests/acceptance.suite.yml`，并按如下配置：

```
class_name: AcceptanceTester
modules:
    enabled:
        - WebDriver:
            url: http://127.0.0.1:8080/
            browser: firefox
        - Yii2:
            part: orm
            entryScript: index-test.php
            cleanup: false
```

5. 打开新的终端，并启动web服务器：

```
tests/bin/yii serve
```

6. 运行验收测试：

```
composer exec codecept run acceptance
```

你应该能看到Selenium是如何启动浏览器的，并检查所有的站点页面。

#### 创建数据库fixtures

在运行自己的测试之前，我们必须清楚自己的测试数据库，并加载指定的测试数据。`yii2-codeception`扩展提供`ActiveFixture`基类，用于为自己的模型创建测试数据集。运行如下步骤创建数据库fixtures：

1. 为`Post`模型创建fixture类：

```
<?php
namespace tests\fixtures;
use yii\test\ActiveFixture;
class PostFixture extends ActiveFixture
{
    public $modelClass = 'app\modules\Post';
    public $dataFile = '@tests/_data/post.php';
}
```

2. 在`test/_data/post.php`文件中添加一个演示数据集：

```
<?php
return [
    [
        'id' => 1,
        'title' => 'First Post',
        'text' => 'First Post Text',
        'status' => 1,
        'created_at' => 1457211600,
        'updated_at' => 1457211600,
    ],
    [
        'id' => 2,
        'title' => 'Old Title For Updating',
        'text' => 'Old Text For Updating',
        'status' => 1,
        'created_at' => 1457211600,
        'updated_at' => 1457211600,
    ],
    [
        'id' => 3,
        'title' => 'Title For Deleting',
        'text' => 'Text For Deleting',
        'status' => 1,
        'created_at' => 1457211600,
        'updated_at' => 1457211600,
    ],
];
```

3. 为单元测试和验收测试激活fixtures支持。只需要添加`fixtures`部分到`unit.suite.yml`文件中：

```
class_name: UnitTester
modules:
    enabled:
        - Asserts
        - Yii2:
            part: [orm, fixtures, email]
```

此外，添加`fixtures`部分到`acceptance.suite.yml`文件中：

```
class_name: AcceptanceTester
modules:
    enabled:
        - WebDriver:
            url: http://127.0.0.1:8080/
            browser: firefox
        - Yii2:
            part: [orm, fixtures]
            entryScript: index-test.php
            cleanup: false
```

4. 运行如下命令，重新生成`tester`类，应用这些修改：

```
composer exec codecept build
```

#### 写单元或综合测试

单元和综合测试检查我们项目的源代码。

```
<?php
namespace tests\unit\models;
use app\models\Post;
use Codeception\Test\Unit;
use tests\fixtures\PostFixture;
class PostTest extends Unit
{
    /**
     * @var \UnitTester
     */
    protected $tester;
    public function _before()
    {
        $this->tester->haveFixtures([
            'post' => [
                'class' => PostFixture::className(),
                'dataFile' => codecept_data_dir() . 'post.php'
            ]
        ]);
    }
    public function testValidateEmpty()
    {
        $model = new Post();
        expect('model should not validate',
            $model->validate())->false();
        expect('title has error',
            $model->errors)->hasKey('title');
        expect('title has error',
            $model->errors)->hasKey('text');
    }
    public function testValidateCorrect()
    {
        $model = new Post([
            'title' => 'Other Post',
            'text' => 'Other Post Text',
        ]);
        expect('model should validate',
            $model->validate())->true();
    }
    public function testSave()
    {
        $model = new Post([
            'title' => 'Test Post',
            'text' => 'Test Post Text',
        ]);
        expect('model should save', $model->save())->true();
        expect('title is correct', $model->title)->equals('Test Post');
        expect('text is correct', $model->text)->equals('Test Post Text');
        expect('status is draft',
            $model->status)->equals(Post::STATUS_DRAFT);
        expect('created_at is generated',
            $model->created_at)->notEmpty();
        expect('updated_at is generated',
            $model->updated_at)->notEmpty();
    }
    public function testPublish()
    {
        $model = new Post(['status' => Post::STATUS_DRAFT]);
        expect('post is drafted',
            $model->status)->equals(Post::STATUS_DRAFT);
        $model->publish();
        expect('post is published',
            $model->status)->equals(Post::STATUS_ACTIVE);
    }
    public function testAlreadyPublished()
    {
        $model = new Post(['status' => Post::STATUS_ACTIVE]);
        $this->setExpectedException('\LogicException');
        $model->publish();
    }
    public function testDraft()
    {
        $model = new Post(['status' => Post::STATUS_ACTIVE]);
        expect('post is published',
            $model->status)->equals(Post::STATUS_ACTIVE);
        $model->draft();
        expect('post is drafted',
            $model->status)->equals(Post::STATUS_DRAFT);
    }
    public function testAlreadyDrafted()
    {
        $model = new Post(['status' => Post::STATUS_ACTIVE]);
        $this->setExpectedException('\LogicException');
        $model->publish();
    }
}
```


```
<?php
namespace tests\functional\admin;
use app\models\Post;
use FunctionalTester;
use tests\fixtures\PostFixture;
use yii\helpers\Url;
class PostsCest
{
    function _before(FunctionalTester $I)
    {
        $I->haveFixtures([
            'user' => [
                'class' => PostFixture::className(),
                'dataFile' => codecept_data_dir() . 'post.php'
            ]
        ]);
    }
    public function testIndex(FunctionalTester $I)
    {
        $I->amOnPage(['admin/posts/index']);
        $I->see('Posts', 'h1');
    }
    public function testView(FunctionalTester $I)
    {
        $I->amOnPage(['admin/posts/view', 'id' => 1]);
        $I->see('First Post', 'h1');
    }
    public function testCreateInvalid(FunctionalTester $I)
    {
        $I->amOnPage(['admin/posts/create']);
        $I->see('Create', 'h1');
        $I->submitForm('#post-form', [
            'Post[title]' => '',
            'Post[text]' => '',
        ]);
        $I->expectTo('see validation errors');
        $I->see('Title cannot be blank.', '.help-block');
        $I->see('Text cannot be blank.', '.help-block');
    }
    public function testCreateValid(FunctionalTester $I)
    {
        $I->amOnPage(['admin/posts/create']);
        $I->see('Create', 'h1');
        $I->submitForm('#post-form', [
            'Post[title]' => 'Post Create Title',
            'Post[text]' => 'Post Create Text',
            'Post[status]' => 'Active',
        ]);
        $I->expectTo('see view page');
        $I->see('Post Create Title', 'h1');
    }
    public function testUpdate(FunctionalTester $I)
    {
        // ...
    }
    public function testDelete(FunctionalTester $I)
    {
        $I->amOnPage(['/admin/posts/view', 'id' => 3]);
        $I->see('Title For Deleting', 'h1');
        $I->amGoingTo('delete item');
        $I->sendAjaxPostRequest(Url::to(['/admin/posts/delete',
            'id' => 3]));
        $I->expectTo('see that post is deleted');
        $I->dontSeeRecord(Post::className(), [
            'title' => 'Title For Deleting',
        ]);
    }
}
```


```
<?php
namespace tests\acceptance\admin;
use AcceptanceTester;
use tests\fixtures\PostFixture;
use yii\helpers\Url;
class PostsCest
{
    function _before(AcceptanceTester $I)
    {
        $I->haveFixtures([
            'post' => [
                'class' => PostFixture::className(),
                'dataFile' => codecept_data_dir() . 'post.php'
            ]
        ]);
    }
    public function testIndex(AcceptanceTester $I)
    {
        $I->wantTo('ensure that post index page works');
        $I->amOnPage(Url::to(['/admin/posts/index']));
        $I->see('Posts', 'h1');
    }
    public function testView(AcceptanceTester $I)
    {
        $I->wantTo('ensure that post view page works');
        $I->amOnPage(Url::to(['/admin/posts/view', 'id' => 1]));
        $I->see('First Post', 'h1');
    }
    public function testCreate(AcceptanceTester $I)
    {
        $I->wantTo('ensure that post create page works');
        $I->amOnPage(Url::to(['/admin/posts/create']));
        $I->see('Create', 'h1');
        $I->fillField('#post-title', 'Post Create Title');
        $I->fillField('#post-text', 'Post Create Text');
        $I->selectOption('#post-status', 'Active');
        $I->click('submit-button');
        $I->wait(3);
        $I->expectTo('see view page');
        $I->see('Post Create Title', 'h1');
    }
    public function testDelete(AcceptanceTester $I)
    {
        $I->amOnPage(Url::to(['/admin/posts/view', 'id' => 3]));
        $I->see('Title For Deleting', 'h1');
        $I->click('Delete');
        $I->acceptPopup();
        $I->wait(3);
        $I->see('Posts', 'h1');
    }
}
```


```
<?php
namespace app\controllers\api;
use yii\rest\ActiveController;
class PostsController extends ActiveController
{
    public $modelClass = '\app\models\Post';
}
```



```
'components' => [
    // ...
    'urlManager' => [
        'enablePrettyUrl' => true,
        'showScriptName' => false,
        'rules' => [
            ['class' => 'yii\rest\UrlRule', 'controller' => 'api/posts'],
        ],
    ],
],
```

```
'components' => [
    // ...
    'urlManager' => [
        'enablePrettyUrl' => true,
        'showScriptName' => true,
        'rules' => [
            ['class' => 'yii\rest\UrlRule', 'controller' => 'api/posts'],
        ],
    ],
],
```



```
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . index.php
```



```
<?php
namespace tests\api;
use ApiTester;
use tests\fixtures\PostFixture;
use yii\helpers\Url;
class PostsCest
{
    function _before(ApiTester $I)
    {
        $I->haveFixtures([
            'post' => [
                'class' => PostFixture::className(),
                'dataFile' => codecept_data_dir() . 'post.php'
            ]
        ]);
    }
    public function testGetAll(ApiTester $I)
    {
        $I->sendGET('/api/posts');
        $I->seeResponseCodeIs(200);
        $I->seeResponseIsJson();
        $I->seeResponseContainsJson([0 => ['title' => 'First Post']]);
    }
    public function testGetOne(ApiTester $I)
    {
        $I->sendGET('/api/posts/1');
        $I->seeResponseCodeIs(200);
        $I->seeResponseIsJson();
        $I->seeResponseContainsJson(['title' => 'First Post']);
    }
    public function testGetNotFound(ApiTester $I)
    {
        $I->sendGET('/api/posts/100');
        $I->seeResponseCodeIs(404);
        $I->seeResponseIsJson();
        $I->seeResponseContainsJson(['name' => 'Not Found']);
    }
    public function testCreate(ApiTester $I)
    {
        $I->sendPOST('/api/posts', [
            'title' => 'Test Title',
            'text' => 'Test Text',
        ]);
        $I->seeResponseCodeIs(201);
        $I->seeResponseIsJson();
        $I->seeResponseContainsJson(['title' => 'Test Title']);
    }
    public function testUpdate(ApiTester $I)
    {
        $I->sendPUT('/api/posts/2', [
            'title' => 'New Title',
        ]);
        $I->seeResponseCodeIs(200);
        $I->seeResponseIsJson();
        $I->seeResponseContainsJson([
            'title' => 'New Title',
            'text' => 'Old Text For Updating',
        ]);
    }
    public function testDelete(ApiTester $I)
    {
        $I->sendDELETE('/api/posts/3');
        $I->seeResponseCodeIs(204);
    }
}
```



```
{
    "name": "book/cart",
    "type": "yii2-extension",
    "require": {
        "yiisoft/yii2": "~2.0"
    },
    "require-dev": {
        "phpunit/phpunit": "4.*"
    },
    "autoload": {
        "psr-4": {
            "book\\cart\\": "src/",
            "book\\cart\\tests\\": "tests/"
        }
    },
    "extra": {
        "asset-installer-paths": {
            "npm-asset-library": "vendor/npm",
            "bower-asset-library": "vendor/bower"
        }
    }
}
```



```
<?xml version="1.0" encoding="utf-8"?>
<phpunit bootstrap="./tests/bootstrap.php"
         colors="true"
         convertErrorsToExceptions="true"
         convertNoticesToExceptions="true"
         convertWarningsToExceptions="true"
         stopOnFailure="false">
    <testsuites>
        <testsuite name="Test Suite">
            <directory>./tests</directory>
        </testsuite>
    </testsuites>
    <filter>
        <whitelist>
            <directory suffix=".php">./src/</directory>
        </whitelist>
    </filter>
</phpunit>
```


```
<?php
namespace book\cart;
use book\cart\storage\StorageInterface;
use yii\base\Component;
use yii\base\InvalidConfigException;
class Cart extends Component
{
    /**
     * @var StorageInterface
     */
    private $_storage;
    /**
     * @var array
     */
    private $_items;
    public function setStorage($storage)
    {
        if (is_array($storage)) {
            $this->_storage = \Yii::createObject($storage);
        } else {
            $this->_storage = $storage;
        }
    }
    public function add($id, $amount = 1)
    {
        $this->loadItems();
        if (isset($this->_items[$id])) {
            $this->_items[$id] += $amount;
        } else {
            $this->_items[$id] = $amount;
        }
        $this->saveItems();
    }
    public function set($id, $amount)
    {
        $this->loadItems();
        $this->_items[$id] = $amount;
        $this->saveItems();
    }
    public function remove($id)
    {
        $this->loadItems();
        if (isset($this->_items[$id])) {
            unset($this->_items[$id]);
        }
        $this->saveItems();
    }
    public function clear()
    {
        $this->loadItems();
        $this->_items = [];
        $this->saveItems();
    }
    public function getItems()
    {
        $this->loadItems();
        return $this->_items;
    }
    public function getCount()
    {
        $this->loadItems();
        return count($this->_items);
    }
    public function getAmount()
    {
        $this->loadItems();
        return array_sum($this->_items);
    }
    private function loadItems()
    {
        if ($this->_storage === null) {
            throw new InvalidConfigException('Storage must be set');
        }
        if ($this->_items === null) {
            $this->_items = $this->_storage->load();
        }
    }
    private function saveItems()
    {
        $this->_storage->save($this->_items);
    }
}
```


```
<?php
namespace book\cart\storage;
interface StorageInterface
{
    /**
     * @return array
     */
    public function load();
    /**
     * @param array $items
     */
    public function save(array $items);
}
```


```
<?php
namespace book\cart\storage;
use Yii;
class SessionStorage implements StorageInterface
{
    public $sessionKey = 'cart';
    public function load()
    {
        return Yii::$app->session->get($this->sessionKey, []);
    }
    public function save(array $items)
    {
        Yii::$app->session->set($this->sessionKey, $items);
    }
}}
```

```
<?php
defined('YII_DEBUG') or define('YII_DEBUG', true);
defined('YII_ENV') or define('YII_ENV', 'test');
require(__DIR__ . '/../vendor/autoload.php');
require(__DIR__ . '/../vendor/yiisoft/yii2/Yii.php');
```

```
<?php
namespace book\cart\tests;
use yii\di\Container;
use yii\web\Application;
abstract class TestCase extends \PHPUnit_Framework_TestCase
{
    protected function setUp()
    {
        parent::setUp();
        $this->mockApplication();
    }
    protected function tearDown()
    {
        $this->destroyApplication();
        parent::tearDown();
    }
    protected function mockApplication()
    {
        new Application([
            'id' => 'testapp',
            'basePath' => __DIR__,
            'vendorPath' => dirname(__DIR__) . '/vendor',
        ]);
    }
    protected function destroyApplication()
    {
        \Yii::$app = null;
        \Yii::$container = new Container();
    }
}
```

```
<?php
namespace book\cart\tests\storage;
use book\cart\storage\StorageInterface;
class FakeStorage implements StorageInterface
{
    private $items = [];
    public function load()
    {
        return $this->items;
    }
    public function save(array $items)
    {
        $this->items = $items;
    }
}
```



```
<?php
namespace book\cart\tests;
use book\cart\Cart;
use book\cart\tests\storage\FakeStorage;
class CartTest extends TestCase
{
    /**
     * @var Cart
     */
    private $cart;
    public function setUp()
    {
        parent::setUp();
        $this->cart = new Cart(['storage' => new
        FakeStorage()]);
    }
    public function testEmpty()
    {
        $this->assertEquals([], $this->cart->getItems());
        $this->assertEquals(0, $this->cart->getCount());
        $this->assertEquals(0, $this->cart->getAmount());
    }
    public function testAdd()
    {
        $this->cart->add(5, 3);
        $this->assertEquals([5 => 3], $this->cart->getItems());
        $this->cart->add(7, 14);
        $this->assertEquals([5 => 3, 7 => 14],
            $this->cart->getItems());
        $this->cart->add(5, 10);
        $this->assertEquals([5 => 13, 7 => 14],
            $this->cart->getItems());
    }
    public function testSet()
    {
        $this->cart->add(5, 3);
        $this->cart->add(7, 14);
        $this->cart->set(5, 12);
        $this->assertEquals([5 => 12, 7 => 14],
            $this->cart->getItems());
    }
    public function testRemove()
    {
        $this->cart->add(5, 3);
        $this->cart->remove(5);
        $this->assertEquals([], $this->cart->getItems());
    }
    public function testClear()
    {
        $this->cart->add(5, 3);
        $this->cart->add(7, 14);
        $this->cart->clear();
        $this->assertEquals([], $this->cart->getItems());
    }
    public function testCount()
    {
        $this->cart->add(5, 3);
        $this->assertEquals(1, $this->cart->getCount());
        $this->cart->add(7, 14);
        $this->assertEquals(2, $this->cart->getCount());
    }
    public function testAmount()
    {
        $this->cart->add(5, 3);
        $this->assertEquals(3, $this->cart->getAmount());
        $this->cart->add(7, 14);
        $this->assertEquals(17, $this->cart->getAmount());
    }
    public function testEmptyStorage()
    {
        $cart = new Cart();
        $this->setExpectedException('yii\base\InvalidConfigException');
        $cart->getItems();
    }
}
```






```
<?php
namespace book\cart\tests\storage;
use book\cart\storage\SessionStorage;
use book\cart\tests\TestCase;
class SessionStorageTest extends TestCase
{
    /**
     * @var SessionStorage
     */
    private $storage;
    public function setUp()
    {
        parent::setUp();
        $this->storage = new SessionStorage(['key' => 'test']);
    }
    public function testEmpty()
    {
        $this->assertEquals([], $this->storage->load());
    }
    public function testStore()
    {
        $this->storage->save($items = [1 => 5, 6 => 12]);
        $this->assertEquals($items, $this->storage->load());
    }
}
```




```
class Cart extends Component
{
    …
    public function remove($id)
    {
        $this->loadItems();
        if (isset($this->_items[$id])) {
            // unset($this->_items[$id]);
        }
        $this->saveItems();
    }
    ...
}
```


```
'components' => [
    // …
    'cart' => [
        'class' => 'book\cart\Cart',
        'storage' => [
            'class' => 'book\cart\storage\SessionStorage',
        ],
    ],
],
```



```
$config = [
    'id' => 'basic',
    'basePath' => dirname(__DIR__),
    'bootstrap' => ['log'],
    'aliases' => [
        '@book' => dirname(__DIR__) . '/book',
    ],
    'components' => [
        'cart' => [
            'class' => 'book\cart\Cart',
            'storage' => [
            'class' => 'book\cart\storage\SessionStorage',
            ],
        ],
        // ...
    ],
]
```





```
<?xml version="1.0" encoding="utf-8"?>
<phpunit bootstrap="./tests/bootstrap.php"
         colors="true"
         convertErrorsToExceptions="true"
         convertNoticesToExceptions="true"
         convertWarningsToExceptions="true"
         stopOnFailure="false">
    <testsuites>
        <testsuite name="Test Suite">
            <directory>./tests</directory>
        </testsuite>
    </testsuites>
    <filter>
        <whitelist>
            <directory suffix=".php">./src/</directory>
        </whitelist>
    </filter>
</phpunit>
```




```
<?php
defined('YII_DEBUG') or define('YII_DEBUG', true);
defined('YII_ENV') or define('YII_ENV', 'test');
require(__DIR__ . '/../vendor/autoload.php');
require(__DIR__ . '/../vendor/yiisoft/yii2/Yii.php');
```


```
class MyTest extends TestCase
{
    public function testSomeFunction()
    {
        $this->assertTrue(true);
    }
}
```

```
$this->assertEqual('Alex', $model->name);
$this->assertTrue($model->validate());
$this->assertFalse($model->save());
$this->assertCount(3, $items);
$this->assertArrayHasKey('username', $model->getErrors());
$this->assertNotNull($model->author);
$this->assertInstanceOf('app\models\User', $model->author);
```

```
<?php
namespace book\cart\tests;
use yii\di\Container;
use yii\web\Application;
abstract class TestCase extends \PHPUnit_Framework_TestCase
{
    protected function setUp()
    {
        parent::setUp();
        $this->mockApplication();
    }
    protected function tearDown()
    {
        $this->destroyApplication();
        parent::tearDown();
    }
    protected function mockApplication()
    {
        new Application([
            'id' => 'testapp',
            'basePath' => __DIR__,
            'vendorPath' => dirname(__DIR__) . '/vendor',
        ]);
    }
    protected function destroyApplication()
    {
        \Yii::$app = null;
        \Yii::$container = new Container();
    }
}
```


## 使用Atoum测试

```
{
    "name": "book/cart",
    "type": "yii2-extension",
    "require": {
        "yiisoft/yii2": "~2.0"
    },
    "require-dev": {
        "atoum/atoum": "^2.7"
    },
    "autoload": {
        "psr-4": {
            "book\\cart\\": "src/",
            "book\\cart\\tests\\": "tests/"
        }
    },
    "extra": {
        "asset-installer-paths": {
            "npm-asset-library": "vendor/npm",
            "bower-asset-library": "vendor/bower"
        }
    }
}
```


```
/vendor
/composer.lock
```

```
composer install
```

```
<?php
defined('YII_DEBUG') or define('YII_DEBUG', true);
defined('YII_ENV') or define('YII_ENV', 'test');
require(__DIR__ . '/../vendor/autoload.php');
require(__DIR__ . '/../vendor/yiisoft/yii2/Yii.php');
```

```
<?php
namespace book\cart\tests;
use yii\di\Container;
use yii\console\Application;
use mageekguy\atoum\test;
abstract class TestCase extends test
{
    public function beforeTestMethod($method)
    {
        parent::beforeTestMethod($method);
        $this->mockApplication();
    }
    public function afterTestMethod($method)
    {
        $this->destroyApplication();
        parent::afterTestMethod($method);
    }
    protected function mockApplication()
    {
        new Application([
            'id' => 'testapp',
            'basePath' => __DIR__,
            'vendorPath' => dirname(__DIR__) . '/vendor',
            'components' => [
                'session' => [
                    'class' => 'yii\web\Session',
                ],
            ]
        ]);
    }
    protected function destroyApplication()
    {
        \Yii::$app = null;
        \Yii::$container = new Container();
    }
}
```

```
<?php
namespace book\cart\tests;
use book\cart\storage\StorageInterface;
class FakeStorage implements StorageInterface
{
    private $items = [];
    public function load()
    {
        return $this->items;
    }
    public function save(array $items)
    {
        $this->items = $items;
    }
}
```

```
<?php
namespace book\cart\tests\units;
use book\cart\tests\FakeStorage;
use book\cart\Cart as TestedCart;
use book\cart\tests\TestCase;
class Cart extends TestCase
{
    /**
     * @var TestedCart
     */
    private $cart;
    public function beforeTestMethod($method)
    {
        parent::beforeTestMethod($method);
        $this->cart = new TestedCart(['storage' => new
        FakeStorage()]);
    }
    public function testEmpty()
    {
        $this->array($this->cart->getItems())->isEqualTo([]);
        $this->integer($this->cart->getCount())->isEqualTo(0);
        $this->integer($this->cart->getAmount())->isEqualTo(0);
    }
    public function testAdd()
    {
        $this->cart->add(5, 3);
        $this->array($this->cart->getItems())->isEqualTo([5 =>
            3]);
        $this->cart->add(7, 14);
        $this->array($this->cart->getItems())->isEqualTo([5 =>
            3, 7 => 14]);
        $this->cart->add(5, 10);
        $this->array($this->cart->getItems())->isEqualTo([5 =>
            13, 7 => 14]);
    }
    public function testSet()
    {
        $this->cart->add(5, 3);
        $this->cart->add(7, 14);
        $this->cart->set(5, 12);
        $this->array($this->cart->getItems())->isEqualTo([5 =>
            12, 7 => 14]);
    }
    public function testRemove()
    {
        $this->cart->add(5, 3);
        $this->cart->remove(5);
        $this->array($this->cart->getItems())->isEqualTo([]);
    }
    public function testClear()
    {
        $this->cart->add(5, 3);
        $this->cart->add(7, 14);
        $this->cart->clear();
        $this->array($this->cart->getItems())->isEqualTo([]);
    }
    public function testCount()
    {
        $this->cart->add(5, 3);
        $this->integer($this->cart->getCount())->isEqualTo(1);
        $this->cart->add(7, 14);
        $this->integer($this->cart->getCount())->isEqualTo(2);
    }
    public function testAmount()
    {
        $this->cart->add(5, 3);
        $this->integer($this->cart->getAmount())->isEqualTo(3);
        $this->cart->add(7, 14);
        $this->integer($this->cart->getAmount())->isEqualTo(17);
    }
    public function testEmptyStorage()
    {
        $cart = new TestedCart();
        $this->exception(function () use ($cart) {
            $cart->getItems();
        })->hasMessage('Storage must be set');
    }
}
```



```
<?php
namespace book\cart\tests\units\storage;
use book\cart\storage\SessionStorage as TestedStorage;
use book\cart\tests\TestCase;
class SessionStorage extends TestCase
{
    /**
     * @var TestedStorage
     */
    private $storage;
    public function beforeTestMethod($method)
    {
        parent::beforeTestMethod($method);
        $this->storage = new TestedStorage(['key' => 'test']);
    }
    public function testEmpty()
    {
        $this
            ->given($storage = $this->storage)
            ->then
            ->array($storage->load())
            ->isEqualTo([]);
    }
    public function testStore()
    {
        $this
            ->given($storage = $this->storage)
            ->and($storage->save($items = [1 => 5, 6 => 12]))
            ->then
            ->array($this->storage->load())
            ->isEqualTo($items)
        ;
    }
}
```




```
class Cart extends Component
{
    ...
    public function remove($id)
    {
        $this->loadItems();
        if (isset($this->_items[$id])) {
            // unset($this->_items[$id]);
        }
        $this->saveItems();
    }
    ...
}
```





```
<?php
use \mageekguy\atoum;
/** @var atoum\scripts\runner $script */
$report = $script->addDefaultReport();
$coverageField = new atoum\report\fields\runner\coverage\
html('Cart', __DIR__ . '/tests/coverage');
$report->addField($coverageField);
```

```
vendor/bin/atoum -d tests/units -bf tests/bootstrap.php -c coverage.php
```


```
public function testSome()
{
    $this
        ->given($cart = new TestedCart())
        ->and($cart->add(5, 13))
        ->then
        ->sizeof($cart->getItems())
        ->isEqualTo(1)
        ->array($cart->getItems())
        ->isEqualTo([5 => 3])
        ->integer($cart->getCount())
        ->isEqualTo(1)
        ->integer($cart->getAmount())
        ->isEqualTo(3);
}
```



```
public function testSome()
{
    $cart = new TestedCart();
    $cart->add(5, 3);
    $this->array($cart->getItems())->isEqualTo([5 => 3])
        ->integer($cart->getCount())->isEqualTo(1)
        ->integer($cart->getAmount())->isEqualTo(3);
}
```


```
{
    "name": "book/cart",
    "type": "yii2-extension",
    "require": {
        "yiisoft/yii2": "~2.0"
    },
    "require-dev": {
        "phpunit/phpunit": "4.*",
        "behat/behat": "^3.1"
    },
    "autoload": {
        "psr-4": {
            "book\\cart\\": "src/",
            "book\\cart\\features\\": "features/"
        }
    },
    "extra": {
        "asset-installer-paths": {
            "npm-asset-library": "vendor/npm",
            "bower-asset-library": "vendor/bower"
        }
    }
}
```


```
<?php
defined('YII_DEBUG') or define('YII_DEBUG', true);
defined('YII_ENV') or define('YII_ENV', 'test');
require_once __DIR__ . '/../../vendor/yiisoft/yii2/Yii.php';
```



```
Feature: Shopping cart
    In order to buy products
    As a customer
    I need to be able to put interesting products into a cart

    Scenario: Checking empty cart
        Given there is a clean cart
        Then I should have 0 products
        Then I should have 0 product
        And the overall cart amount should be 0

    Scenario: Adding products to the cart
        Given there is a clean cart
        When I add 3 pieces of 5 product
        Then I should have 3 pieces of 5 product
        And I should have 1 product
        And the overall cart amount should be 3
        When I add 14 pieces of 7 product
        Then I should have 3 pieces of 5 product
        And I should have 14 pieces of 7 product
        And I should have 2 products
        And the overall cart amount should be 17
        When I add 10 pieces of 5 product
        Then I should have 13 pieces of 5 product
        And I should have 14 pieces of 7 product
        And I should have 2 products
        And the overall cart amount should be 27

    Scenario: Change product count in the cart
        Given there is a cart with 5 pieces of 7 product
        When I set 3 pieces for 7 product
        Then I should have 3 pieces of 7 product

    Scenario: Remove products from the cart
        Given there is a cart with 5 pieces of 7 product
        When I add 14 pieces of 7 product
        And I clear cart
        Then I should have empty cart
```




```
Feature: Shopping cart storage
    I need to be able to put items into a storage
    Scenario: Checking empty storage
        Given there is a clean storage
        Then I should have empty storage

    Scenario: Save items into storage
        Given there is a clean storage
        When I save 3 pieces of 7 product to the storage
        Then I should have 3 pieces of 7 product in the storage
```

```
<?php
use Behat\Behat\Context\SnippetAcceptingContext;
use book\cart\Cart;
use book\cart\features\bootstrap\storage\FakeStorage;
use yii\di\Container;
use yii\web\Application;
require_once __DIR__ . '/bootstrap.php';
class CartContext implements SnippetAcceptingContext
{
    /**
     * @var Cart
     * */
    private $cart;
    /**
     * @Given there is a clean cart
     */
    public function thereIsACleanCart()
    {
        $this->resetCart();
    }
    /**
     * @Given there is a cart with :pieces of :product product
     */
    public function thereIsAWhichCostsPs($product, $amount)
    {
        $this->resetCart();
        $this->cart->set($product, floatval($amount));
    }
    /**
     * @When I add :pieces of :product
     */
    public function iAddTheToTheCart($product, $pieces)
    {
        $this->cart->add($product, $pieces);
    }
    /**
     * @When I set :pieces for :arg2 product
     */
    public function iSetPiecesForProduct($pieces, $product)
    {
        $this->cart->set($product, $pieces);
    }
    /**
     * @When I clear cart
     */
    public function iClearCart()
    {
        $this->cart->clear();
    }
    /**
     * @Then I should have empty cart
     */
    public function iShouldHaveEmptyCart()
    {
        PHPUnit_Framework_Assert::assertEquals(
            0,
            $this->cart->getCount()
        );
    }
    /**
     * @Then I should have :count product(s)
     */
    public function iShouldHaveProductInTheCart($count)
    {
        PHPUnit_Framework_Assert::assertEquals(
            intval($count),
            $this->cart->getCount()
        );
    }
    /**
     * @Then the overall cart amount should be :amount
     */
    public function theOverallCartPriceShouldBePs($amount)
    {
        PHPUnit_Framework_Assert::assertSame(
            intval($amount),
            $this->cart->getAmount()
        );
    }
    /**
     * @Then I should have :pieces of :product
     */
    public function iShouldHavePiecesOfProduct($pieces,
                                               $product)
    {
        PHPUnit_Framework_Assert::assertArraySubset(
            [intval($product) => intval($pieces)],
            $this->cart->getItems()
        );
    }
    private function resetCart()
    {
        $this->cart = new Cart(['storage' => new
        FakeStorage()]);
    }
}
```

```
<?php
use Behat\Behat\Context\SnippetAcceptingContext;
use book\cart\Cart;
use book\cart\features\bootstrap\storage\FakeStorage;
use book\cart\storage\SessionStorage;
use yii\di\Container;
use yii\web\Application;
require_once __DIR__ . '/bootstrap.php';
class StorageContext implements SnippetAcceptingContext
{
    /**
     * @var SessionStorage
     * */
    private $storage;
    /**
     * @Given there is a clean storage
     */
    public function thereIsACleanStorage()
    {
        $this->mockApplication();
        $this->storage = new SessionStorage(['key' => 'test']);
    }
    /**
     * @When I save :pieces of :product to the storage
     */
    public function iSavePiecesOfProductToTheStorage($pieces,
                                                     $product)
    {
        $this->storage->save([$product => $pieces]);
    }
    /**
     * @Then I should have empty storage
     */
    public function iShouldHaveEmptyStorage()
    {
        PHPUnit_Framework_Assert::assertCount(
            0,
            $this->storage->load()
        );
    }
    /**
     * @Then I should have :pieces of :product in the storage
     */
    public function
    iShouldHavePiecesOfProductInTheStorage($pieces, $product)
    {
        PHPUnit_Framework_Assert::assertArraySubset(
            [intval($product) => intval($pieces)],
            $this->storage->load()
        );
    }
    private function mockApplication()
    {
        Yii::$container = new Container();
        new Application([
            'id' => 'testapp',
            'basePath' => __DIR__,
            'vendorPath' => __DIR__ . '/../../vendor',
        ]);
    }
}
```




```
<?php
namespace book\cart\features\bootstrap\storage;
use book\cart\storage\StorageInterface;
class FakeStorage implements StorageInterface
{
    private $items = [];
    public function load()
    {
        return $this->items;
    }
    public function save(array $items)
    {
        $this->items = $items;
    }
}
```



![](../images/a1105.png)

![](../images/a1106.png)

![](../images/a1107.png)

![](../images/a1108.png)

![](../images/a1109.png)

![](../images/a1110.png)

![](../images/a1111.png)

![](../images/a1112.png)
