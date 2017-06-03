# 官方扩展

在本章中，我们将会讨论如下话题：

- 身份认证客户端
- SwitchMailer电子邮件库
- Faker fixture数据生成器
- Imagine库
- MongoDB驱动
- ElasticSearch引擎适配器
- Gii代码生成器
- Pjax JQuery插件
- Redis数据库驱动

## 介绍

Yii2官方库为一些流行的库、数据库和搜索引擎提供了适配器。在本章中，我们将会向你展示如何在你的项目中安装和使用官方扩展。你也将会了解到如何写自己的扩展，并分享给其它开发者。

## 身份认证客户端

这个扩展为Yii2框架添加了OpenID、OAuth和OAuth2 consumers。

### 准备

1. 按照官方指南[http://www.yiiframework.com/doc-2.0/guide-start-installation.html](http://www.yiiframework.com/doc-2.0/guide-start-installation.html)的描述，使用Composer包管理器创建一个新的应用。
2. 使用如下命令安装扩展：

```
composer require yiisoft/yii2-authclient
```

### 如何做...

1. 打开你的Github应用页面`https://github.com/settings/applications`并添加自己的新应用：

![](../images/701.png)

2. 获取**Client ID**和**Client Secret**：

![](../images/702.png)

3. 配置你的web配置，并为`authClientCollection`组件设置相应的选项：

```
'components' => [
    // ...
    'authClientCollection' => [
        'class' => 'yii\authclient\Collection',
        'clients' => [
            'google' => [
                'class' =>'yii\authclient\clients\GoogleOpenId'
            ],
            'github' => [
                'class' => 'yii\authclient\clients\GitHub',
                'clientId' => '87f0784aae2ac48f78a',
                'clientSecret' =>'fb5953a54dea4640f3a70d8abd96fbd25592ff18',
            ],
            // etc.
        ],
    ],
],
```

4. 打开你的`SiteController`并添加`auth`独立动作和成功回调方法：

```
use yii\authclient\ClientInterface;
public function actions()
{
    return [
        // ...
        'auth' => [
            'class' => 'yii\authclient\AuthAction',
            'successCallback' => [$this, 'onAuthSuccess'],
        ],
    ];
}

public function onAuthSuccess(ClientInterface $client)
{
    $attributes = $client->getUserAttributes();
    \yii\helpers\VarDumper::dump($attributes, 10, true);
    exit;
}
```

5. 打开`views/site/login.php`文件并插入`AuthChoice`小组件：

```
<div class="site-login">
    <h1><?= Html::encode($this->title) ?></h1>
    <div class="panel panel-default">
        <div class="panel-body">
            <?= yii\authclient\widgets\AuthChoice::widget(['baseAuthUrl' => ['site/auth'], 'popupMode' => false,]) ?>
        </div>
    </div>
    <p>Please fill out the following fields to login:</p>
    ...
</div>
```

6. 你将会看到你配置的图标：

![](../images/703.png)

7. 为了使用Github provider进行验证：

![](../images/704.png)

8. 如果成功，你的回调将会展示验证的用户属性：

```
[
    'login' => 'Name'
    'id' => 0000000
    'avatar_url' =>'https://avatars.githubusercontent.com/u/0000000?v=3'
    'gravatar_id' => ''
    'url' => 'https://api.github.com/users/Name'
    'html_url' => 'https://github.com/Name'
    //...
    'name' => 'YourName'
    'blog' =>site.com'
    'email => mail@site.com'
    //...
]
```

9. 在`onAuthSuccess`方法中创建你自己认证的代码，可参考例子[https://github.com/yiisoft/yii2-authclient/blob/master/docs/guide/quick-start.md](https://github.com/yiisoft/yii2-authclient/blob/master/docs/guide/quick-start.md)。

### 工作原理...

这个扩展为你的应用提供了OpenID、OAuth和OAuth2认证客户端。

`AuthChoice`小组件在一个选择的服务网站上打开了一个身份认证页面，存储`auth`动作URL。身份认证以后，通过一个POST请求发送认证数据时，当前的服务将用户重定向回去。`AuthAction`收到这个请求，并调用相应的回调。

你可以使用任何存在的客户端或者创建自己的。

### 参考

- 为了了解更多关于扩展使用的信息，参考：
    + [https://github.com/yiisoft/yii2-authclient/tree/master/docs/guide](https://github.com/yiisoft/yii2-authclient/tree/master/docs/guide)
    + [http://www.yiiframework.com/doc-2.0/ext-authclient-index.html](http://www.yiiframework.com/doc-2.0/ext-authclient-index.html)
- 欲了解更多关于OpenID、OAuth和OAuth2认证技术，参考：
    + [http://openid.net](http://openid.net)
    + [http://oauth.net](http://oauth.net)

## SwitchMailer电子邮件库

许多web应用因为安全原因需要通过电子邮件发送通知和确认客户端动作。Yii2框架为已存在的SwitchMailer库提供了一个warpper，`yiisoft/yii2-swiftmailer`。

### 准备

按照官方指南[http://www.yiiframework.com/doc-2.0/guide-start-installation.html](http://www.yiiframework.com/doc-2.0/guide-start-installation.html)的描述，使用Composer包管理器创建一个新的应用。

基础应用和高级应用都包含这个扩展。

### 如何做...

现在我们将会尝试从我们自己的应用中发送任何种类的电子邮件。

#### 发送纯文本电子邮件

1. 在`config/console.php`文件中设置mailer配置：

```
'components' => [
    // ...
    'mailer' => [
        'class' => 'yii\swiftmailer\Mailer',
        'useFileTransport' => true,
    ],
    // ...
],
```

2. 创建一个测试控制台控制器，`MailController`：

```
<?php
namespace app\commands;
use yii\console\Controller;
use Yii;
class MailController extends Controller
{
    public function actionSend()
    {
        Yii::$app->mailer->compose()
            ->setTo('to@yii-book.app')
            ->setFrom(['from@yii-book.app' => Yii::$app->name])
            ->setSubject('My Test Message')
            ->setTextBody('My Text Body')
            ->send();
    }
}
```

3. 运行如下控制台命令：

```
php yii main/send
```

4. 检查你的`runtime/mail`目录。它应该包含你的邮件文件。

**注意**：邮件文件包含了特殊电子邮件源格式的信息，兼容任何邮件软件。你可以按纯文本文件打开。

5. 设置`useFileTransport`参数为false，或者从配置中移除这个字符串：

```
'mailer' => [
    'class' => 'yii\swiftmailer\Mailer',
],
```

然后将你真实的电子邮件ID放入`setTo()`方法：

```
->setTo('my@real-email.com')
```

6. 再次运行控制台命令：

```
php yii mail/send
```

7. 检查你的`inbox`目录。

**注意**：默认情况下，SwiftMailer使用了一个标准的PHP函数，`mail()`，来发送邮件。请检查你的服务器是否正确设置，从而可以使用`mail()`函数发送邮件。

需要邮箱系统拒绝没有DKIM和SPF签名的邮件（例如使用`mail()`函数发送的邮件）或者将他们放到垃圾文件夹中。

#### 发送HTML内容

1. 检查你应用中的`mail/layouts/html.php`文件并使用如下内容添加`mail/layouts/text.php`文件：

```
<?php
/* @var $this \yii\web\View */
/* @var $message \yii\mail\MessageInterface */
/* @var $content string */
?>
<?php $this->beginPage() ?>
<?php $this->beginBody() ?>
<?= $content ?>
<?php $this->endBody() ?>
<?php $this->endPage() ?>
```

2. 在`mail/message-html.php`文件中创建你自己的视图：

```
<?php
use yii\helpers\Html;
/* @var $this yii\web\View */
/* @var $name string */
?>
<p>Hello, <?= Html::encode($name) ?>!</p>
Create a mail/message-text.php file with the same content, but without HTML tags:
<?php
use yii\helpers\Html;
/* @var $this yii\web\View */
/* @var $name string */
?>
Hello, <?= Html::encode($name) ?>!
```

3. 使用如下代码创建一个控制台控制器`MailController`：

```
<?php
namespace app\commands;
use yii\console\Controller;
use Yii;
class MailController extends Controller
{
    public function actionSendHtml()
    {
        $name = 'John';
        Yii::$app->mailer->compose('message-html',['name' => $name])
            ->setTo('to@yii-book.app')
            ->setFrom(['from@yii-book.app' => Yii::$app->name])
            ->setSubject('My Test Message')
            ->send();
    }
    public function actionSendCombine()
    {
        $name = 'John';
        Yii::$app->mailer->compose(['html' => 'message-html', 'text' => 'message-text'], ['name' => $name,])
            ->setTo('to@yii-book.app')
            ->setFrom(['from@yii-book.app'
            => Yii::$app->name])
            ->setSubject('My Test Message')
            ->send();
    }
}
```

4. 运行如下控制台命令：

```
php yii mail/send-html
php yii mail/send-combine
```

#### 使用SMTP传输

1. 为`mailer`组件设置`transport`参数：

```
'mailer' => [
    'class' => 'yii\swiftmailer\Mailer',
    'transport' => [
        'class' => 'Swift_SmtpTransport',
        'host' => 'smtp.gmail.com',
        'username' => 'username@gmail.com',
        'password' => 'password',
        'port' => '587',
        'encryption' => 'tls',
    ],
],
```

2. 书写并运行如下代码：

```
Yii::$app->mailer->compose()
    ->setTo('to@yii-book.app')
    ->setFrom('username@gmail.com')
    ->setSubject('My Test Message')
    ->setTextBody('My Text Body')
    ->send();
```

3. 检查你的Gmail收件箱。

**注意**：Gmail自动重写`From`字段为你的默认电子邮件ID，但其他电子邮件系统没有这么做。在传输配置中总是使用一个唯一电子邮件ID，并在`setFrom()`方法中为其它电子邮件系统中传递反垃圾邮件政策。

#### 添加附件和图片

添加相关的方法来附加任何文件到你的邮件中：

```
class MailController extends Controller
{
    public function actionSendAttach()
    {
        Yii::$app->mailer->compose()
            ->setTo('to@yii-book.app')
            ->setFrom(['from@yii-book.app' => Yii::$app->name])
            ->setSubject('My Test Message')
            ->setTextBody('My Text Body')
            ->attach(Yii::getAlias('@app/README.md'))
            ->send();
    }
}
```

或者在你的电子邮件视图文件中使用`embed()`方法来粘贴一个图片到电子邮件内容中：

```
<img src="<?= $message->embed($imageFile); ?>">
```

它会自动添加图片文件附件并插入它的唯一标识。

### 工作原理...

wrapper实现了`\yii\mail\MailerInterface`。它的`compose()`方法返回了一个消息对象（`\yii\mail\MessageInterface`的一个实现）。

你可以使用`setTextBody()`和`setHtmlBody()`手动设置纯文本和HTML内容，或者你可以将你的视图和视图参数传递给`compose()`方法。在这个例子中，mailer调用`\yii\web\View::render()`方法来渲染相应的内容。

`useFileTransport`参数在文件中存储电子邮件而不是真正的发送。它对于本地开发和应用测试非常有用。

### 参考

- 欲了解更多关于`yii2-swiftmailer`扩展的信息，访问如下地址：
    + [http://www.yiiframework.com/doc-2.0/guide-tutorial-mailing.html](http://www.yiiframework.com/doc-2.0/guide-tutorial-mailing.html)
    + [http://www.yiiframework.com/doc-2.0/ext-swiftmailer-index.html](http://www.yiiframework.com/doc-2.0/ext-swiftmailer-index.html)
- 欲了解更多关于`SwiftMailer`库，参考如下地址：
    + [http://swiftmailer.org/docs/introduction.html](http://swiftmailer.org/docs/introduction.html)
    + [https://github.com/swiftmailer/swiftmailer](https://github.com/swiftmailer/swiftmailer)

## Fake fixture 数据生成器

`fzaninotto/faker`是一个PHP扩展，它可以生成需要种类的假数据：名称、电话、地址，以及随机字符串和数字等等。它可以帮助你生成需要随机记录，用于性能和逻辑测试。你可以通过写自己的formatters和generators来扩展你支持的类型集合。

在Yii2应用骨架中，`yiisoft/yii2-faker` wrapper被包含在`composer.json`文件的`require-dev`部分中，这部分用于测试代码（第十一章，*测试*）。这个wrapper为控制台应用和测试环境提供`FixtureController`控制台。

### 准备

按照官方指南[http://www.yiiframework.com/doc-2.0/guide-start-installation.html](http://www.yiiframework.com/doc-2.0/guide-start-installation.html)的描述，使用Composer包管理器创建一个新的应用。

### 如何做...

1. 打开目录`tests/codeception/templates`并添加fixture模板文件`users.txt`：

```
<?php
/**
* @var $faker \Faker\Generator
* @var $index integer
*/
return [
    'name' => $faker->firstName,
    'phone' => $faker->phoneNumber,
    'city' => $faker->city,
    'about' => $faker->sentence(7, true),
    'password' => Yii::$app->getSecurity()->generatePasswordHash('password_' . $index),
    'auth_key' => Yii::$app->getSecurity()->generateRandomString(),
];
```

2. 运行测试控制台`yii`命令：

```
php tests/codeception/bin/yii fixture/generate users --count=2
```

3. 确认migration生成。

4. 检查`tests/codeception/fixtures`是否包含新的`users.php`文件，以及自动生成的数据：

```
return [
    [
        'name' => 'Isadore',
        'phone' => '952.877.8545x190',
        'city' => 'New Marvinburgh',
        'about' => 'Ut quidem voluptatem itaque veniam voluptas dolores.',
        'password' => '$2y$13$Fi3LOl/sKlomUH.DLgqBkOB/uCLmgCoPPL1KXiW0hffnkrdkjCzAC',
        'auth_key' => '1m05hlgaAG8zfm0cyDyoRGMkbQ9W6hj1',
    ],
    [
        'name' => 'Raleigh',
        'phone' => '1-655-488-3585x699',
        'city' => 'Reedstad',
        'about' => 'Dolorem quae impedit tempore libero doloribus nobis dicta tempora facere.',
        'password' => '$2y$13$U7Qte5Y1jVLrx/pnhwdwt.1uXDegGXuNVzEQyUsb65WkBtjyjUuYm',
        'auth_key' => 'uWWJDgy5jNRk6KjqpxS5JuPv0OHearqE',
    ],
],
```

#### 使用你自己的数据类型

1. 使用你自定义生成逻辑创建你自己的provider：

```
<?php
namespace tests\codeception\faker\providers;
use Faker\Provider\Base;
class UserStatus extends Base
{
    public function userStatus()
    {
        return $this->randomElement([0, 10, 20, 30]);
    }
}
```

2. 在`/tests/codeception/config/config.php`文件中添加provider到provider列表中：

```
return [
    'controllerMap' => [
        'fixture' => [
            'class' => 'yii\faker\FixtureController',
            'fixtureDataPath' => '@tests/codeception/fixtures',
            'templatePath' => '@tests/codeception/templates',
            'namespace' => 'tests\codeception\fixtures',
            'providers' => [
                'tests\codeception\faker\providers\UserStatus',
            ],
        ],
    ],
// ...
];
```

3. 添加`status`字段到你的fixture模板文件中：

```
<?php
/**
 * @var $faker \Faker\Generator
 * @var $index integer
 */
return [
    'name' => $faker->firstName,
    'status' => $faker->userStatus,
];
```

4. 使用控制台命令生成fixture：

```
php tests/codeception/bin/yii fixture/generate users --count=2
```

5. 检查`fixtures/users.php`生成的代码是否包含你的自定义值：

```
return [
    [
        'name' => 'Christelle',
        'status' => 30,
    ],
    [
        'name' => 'Theo',
        'status' => 10,
    ],
];
```

### 工作原理...

`yii2-faker`扩展包含一个控制台生成器（它使用你的模板来生成fixture数据文件），并给了你一个准备好的原始`Faker`对象实例。你可以生成所有或者指定的fixtures，并且你可以在控制台参数中传递自定义数值或者语言。

**注意**

如果你的测试使用这些fixtures的话，小心已存在的测试文件，因为自动生成会完全覆盖旧数据。

### 参考

- 源代码以及关于扩展的更多信息，参考：
- [https://github.com/yiisoft/yii2-faker/tree/master/docs/guide](https://github.com/yiisoft/yii2-faker/tree/master/docs/guide)
- [http://www.yiiframework.com/doc-2.0/ext-faker-index.html](http://www.yiiframework.com/doc-2.0/ext-faker-index.html)
- 欲了解更多关于原始库的信息，参考：
- [https://github.com/fzaninotto/Faker](https://github.com/fzaninotto/Faker)
- 第十一章，*测试*

## Imagine库

Imagine是用于操作图片的OOP库。它可以让你在GD、Imagic和Gmagic PHP扩展的帮助下，对多种格式的图片进行裁剪、缩放以及执行其它操作。Yii2-Imagine是队这个库的轻量静态封装。

### 准备

1. 按照官方指南[http://www.yiiframework.com/doc-2.0/guide-start-installation.html](http://www.yiiframework.com/doc-2.0/guide-start-installation.html)的描述，使用Composer包管理器创建一个新的应用。
2. 使用如下命令安装扩展：

```
composer require yiisoft/yii2-imagine
```

### 如何做...

在你的项目中，你可以以两种方式使用扩展：

- 作为工厂使用
- 使用内部方法

#### 作为工厂使用

你可以使用`Imagine`库类的一个实例：

```
$imagine = new Imagine\Gd\Imagine();
// or
$imagine = new Imagine\Imagick\Imagine();
// or
$imagine = new Imagine\Gmagick\Imagine();
```

但是，这依赖于你系统中已存在的PHP扩展。你可以使用`getImagine()`方法：

```
$imagine = \yii\imagine\Image::getImagine();
```

#### 使用内部方法

你可以使用`corp()`、`thumbnail()`、`watermark()`、`text()`、`frame()`方法用于常见的高级操作：

```
<?php
use yii\imagine\Image;
Image::crop('path/to/image.jpg', 100, 100, ManipulatorInterface::THUMBNAIL_OUTBOUND)
->save('path/to/destination/image.jpg', ['quality' => 90]);
```



### 工作原理...

### 参考


## MongoDB驱动

### 准备

###如何做...

#### 基本用法

#### 注意

### 工作原理...

### 参考

## ElasticSearch引擎适配器

### 准备

### 如何做...

#### 使用查询类

#### 使用ActiveRecord

#### 使用ElasticSearch调试板

### 工作原理...

#### 注意

### 参考

## Gii代码生成器

### 准备

### 如何做...

#### 使用GUI

#### 使用CLI

### 工作原理...

### 参考

## Pjax JQuery插件

### 准备

### 如何做...

#### 指定一个自定义ID

#### 使用ActiveForm

#### 使用客户端脚本

### 工作原理...

### 参考

## Redis数据库驱动

### 准备

### 如何做...

#### 直接使用方法

#### 使用ActiveRecord

### 工作原理...

### 参考





![](../images/705.png)

![](../images/706.png)

![](../images/707.png)

![](../images/708.png)

![](../images/709.png)

![](../images/710.png)

![](../images/711.png)

![](../images/712.png)

![](../images/713.png)

![](../images/714.png)

![](../images/715.png)

![](../images/716.png)

![](../images/717.png)

![](../images/718.png)
