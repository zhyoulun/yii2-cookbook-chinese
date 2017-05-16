# 第四章 表单

在本章中，我们将讨论如下话题：

- 自定义校验器
- 上传文件
- 添加和自定义CaptchaWidget
- 自定义Captcha
- 创建一个自定义输入小部件
- Tabular输入
- 条件校验器
- 带有多个模型的复杂表单
- 依赖AJAX的下拉列表
- AJAX校验器
- 创建一个自定义客户端的校验器

## 介绍

Yii使得使用forms非常容易，并且关于它的文档非常完整。但仍有一些问题需要说明和例子。我们将在本章中介绍说明。

## 自定义校验器

Yii提供了一套内置表单校验器，基本覆盖了所有典型的开发需求，并且是高度可配置的。但是，在一些情况下，开发者可能需要创建一个自定义校验器。

本小节会给出一个例子，创建一个检查单词个数的独立校验器。

### 准备

按照官方指南[http://www.yiiframework.com/doc-2.0/guide-start-installation.html](http://www.yiiframework.com/doc-2.0/guide-start-installation.html)的描述，使用Composer包管理器创建一个新的应用。

### 如何做...

1. 创建一个独立校验器`@app/components/WordsValidator.php`：

```
<?php
namespace app\components;
use yii\validators\Validator;
class WordsValidator extends Validator
{
    public $size = 50;
    public function validateValue($value){
        if (str_word_count($value) > $this->size) {
            return ['The number of words must be less than {size}', ['size' => $this->size]];
        }
        return false;
    }
}
```

2. 创建一个`Article`模型`@app/models/Article.php`：

```
<?php
namespace app\models;
use app\components\WordsValidator;
use yii\base\Model;
class Article extends Model
{
    public $title;
    public function rules()
    {
        return [
            ['title', 'string'],
            ['title', WordsValidator::className(), 'size' =>
                10],
        ];
    }
}
```

3. 创建`@app/controllers/ModelValidationController.php`：

```
<?php
namespace app\controllers;
use app\models\Article;
use yii\helpers\Html;
use yii\web\Controller;
class ModelValidationController extends Controller
{
    private function getLongTitle()
    {
        return 'There is a very long content for current article, '.'it should be less then ten words';
    }
    private function getShortTitle()
    {
        return 'There is a shot title';
    }
    private function renderContentByModel($title)
    {
        $model = new Article();
        $model->title = $title;
        if ($model->validate()) {
            $content = Html::tag('div', 'Model is valid.',[
                'class' => 'alert alert-success',
            ]);
        } else {
            $content = Html::errorSummary($model, [
                'class' => 'alert alert-danger',
            ]);
        }
        return $this->renderContent($content);
    }
    public function actionSuccess()
    {
        $title = $this->getShortTitle();
        return $this->renderContentByModel($title);
    }
    public function actionFailure()
    {
        $title = $this->getLongTitle();
        return $this->renderContentByModel($title);
    }
}
```

4. 访问`index.php?r=model-validation/success`来运行`modelValidation`控制器的`success`动作：

![](../images/401.png)

5. 访问`index.php?r=model-validation/failure`来运行`modelValidation`控制器的`failure`动作：

![](../images/402.png)

6. 创建`@app/controllers/AdhocValidationController.php`：

```
<?php
namespace app\controllers;
use app\components\WordsValidator;
use app\models\Article;
use yii\helpers\Html;
use yii\web\Controller;
class AdhocValidationController extends Controller
{
    private function getLongTitle()
    {
        return 'There is a very long content for current article, '.'it should be less then ten words';
    }
    private function getShortTitle()
    {
        return 'There is a shot title';
    }
    private function renderContentByTitle($title)
    {
        $validator = new WordsValidator([
            'size' => 10,
        ]);
        if ($validator->validate($title, $error)) {
            $content = Html::tag('div', 'Value is valid.',[
                'class' => 'alert alert-success',
            ]);
        } else {
            $content = Html::tag('div', $error, [
                'class' => 'alert alert-danger',
            ]);
        }
        return $this->renderContent($content);
    }
    public function actionSuccess()
    {
        $title = $this->getShortTitle();
        return $this->renderContentByTitle($title);
    }
    public function actionFailure()
    {
        $title = $this->getLongTitle();
        return $this->renderContentByTitle($title);
    }
}
```

7. 访问`index.php?r=adhoc-validation/success`来运行`adhocValidation`控制器的`success`动作：

![](../images/403.png)

8. 访问`index.php?r=adhoc-validation/failure`来运行`adhocValidation`控制器的`failure`动作：

![](../images/404.png)

### 工作原理

首先我们创建了一个独立的校验器，它会使用`str_word_count`函数来检查单词的数量，然后演示了两个使用例子：

- 作为`Article`模型的校验规则使用这个校验器
- 作为一个特定的校验器使用这个校验器

### 参考

欲了解更多信息，参考如下链接：

- [http://www.yiiframework.com/doc-2.0/guide-input-validation.html](http://www.yiiframework.com/doc-2.0/guide-input-validation.html)
- [http://www.yiiframework.com/doc-2.0/guide-tutorial-corevalidators.html](http://www.yiiframework.com/doc-2.0/guide-tutorial-corevalidators.html)

## 上传文件

处理文件上传对于web应用是非常常见的一个任务。Yii有一些非常有用的内置类。让我们创建一个简单的表单，它允许上传ZIP压缩包，并保存到`/uploads`文件夹中。

### 准备

1. 按照官方指南[http://www.yiiframework.com/doc-2.0/guide-start-installation.html](http://www.yiiframework.com/doc-2.0/guide-start-installation.html)的描述，使用Composer包管理器创建一个新的应用。
2. 创建`@app/web/uploads`文件夹

### 如何做...

1. 我们将会以一个模型开始，创建`@app/models/Upload.php`：

```
<?php
namespace app\models;
use yii\base\Model;
use yii\web\UploadedFile;
class UploadForm extends Model
{
    /**
     * @var UploadedFile
     */
    public $file;
    public function rules()
    {
        return [
            ['file', 'file', 'skipOnEmpty' => false,
                'extensions' => 'zip'],
        ];
    }
    public function upload()
    {
        if ($this->validate()) {
            $this->file->saveAs('uploads/' .
                $this->file->baseName . '.' . $this->file->extension);
            return true;
        } else {
            return false;
        }
    }
}
```

2. 现在我们来看控制器，创建`@app/controllers/UploadController.php`：

```
<?php
namespace app\controllers;
use Yii;
use yii\web\Controller;
use app\models\UploadForm;
use yii\web\UploadedFile;
class UploadController extends Controller
{
    public function actionUpload()
    {
        $model = new UploadForm();
        if (Yii::$app->request->isPost) {
            $model->file = UploadedFile::getInstance($model,
                'file');
            if ($model->upload()) {
                return $this->renderContent("File {$model->file->name} is uploaded successfully");
            }
        }
        return $this->render('index', ['model' => $model]);
    }
}
```

3. 最后是`@app/views/upload/index.php`：

```
<?php
use yii\widgets\ActiveForm;
use yii\helpers\Html;
?>
<?php $form = ActiveForm::begin(['options' => ['enctype' => 'multipart/form-data']]) ?>
    <?= $form->field($model, 'file')->fileInput() ?>
    <?= Html::submitButton('Upload', ['class' => 'btn-success'])?>
<?php ActiveForm::end() ?>
```

4. 现在运行upload控制器，并尝试上传ZIP压缩包和其他文件：

![](../images/405.png)

### 工作原理...

我们使用的模型非常简单，我们只是定义了一个字段，名叫`$file`，以及一个使用`FileValidator`文件校验器的校验规则，它只读取ZIP文件。

我们创建一个模型的实例，并在提交表单的时候使用`$_POST`中的数据填充它：

```
$model->file = UploadedFile::getInstance($model, 'file');
if ($model->upload()) {
    return $this->renderContent("File {$model->file->name} is uploaded successfully");
}
```

然后我们使用`UploadFile::getInstance`，它给我们`UploadFile`的实例。当上传文件的时候，它是对`$_FILE`数组的封装。通过调用模型的`validate`方法，我们确保这个文件是一个ZIP压缩包，然后我们使用`UploadFile::saveAs`保存文件。

为了上传文件，HTML表单必须满足如下两个重要的需求：

- 必须使用`POST`方法
- `enctype`属性必须设置为`multipart/form-data`

记住你需要添加`enctype`选项到表单，这样文件才能正确上传。

我们可以使用`Html`帮助类或者带有`htmlOptions`集合的`ActiveForm`来生成HTML。这里使用的HTML是：

```
<?= Html::beginForm('', 'post', ['enctype'=>'multipart/form-data'])?>
```

最后，我们为模型的file属性展示了一个错误和一个字段，并渲染了一个提交按钮。

### 更多...

为了上传多个文件，Yii2实现了两个特殊的方法。

例如，你已经定义了`$imageFiles`，在你的模型、视图文件中所有都是一样的，除了一些细小的差别：

```
...
<?= $form->field($model, 'imageFiles[]')->fileInput(['multiple' => true, 'accept' => 'image/*']) ?>
...
```

为了获取所有文件的实例，你必须调用`UploadFile::getInstances()`而不是`UploadFile::getInstance()`：

```
..
$model->imageFiles = UploadedFile::getInstances($model, 'imageFiles');
..
```

可以使用简单的代码来处理并保存多个文件：

```
foreach ($this->imageFiles as $file) {
    $file->saveAs('uploads/' . $file->baseName . '.' .$file->extension);
}
```

### 参考

欲了解更多信息，参考：

- [http://www.yiiframework.com/doc-2.0/guide-input-file-upload.html](http://www.yiiframework.com/doc-2.0/guide-input-file-upload.html)
- [http://www.yiiframework.com/doc-2.0/guide-input-file-upload.html#uploading-multiple-files](http://www.yiiframework.com/doc-2.0/guide-input-file-upload.html#uploading-multiple-files)

## 添加和自定义CaptchaWidget

现如今在互联网上，如果你放出了一个没有做垃圾信息防护的表单，你将会在短时间内收到大量的垃圾数据。Yii有一个验证码组件，它可以让添加这样的防护非常简单。唯一的问题是没有系统的使用说明。

在接下来的例子中，我们将会给一个简单的表单添加验证码防护。

### 准备

1. 按照官方指南[http://www.yiiframework.com/doc-2.0/guide-start-installation.html](http://www.yiiframework.com/doc-2.0/guide-start-installation.html)的描述，使用Composer包管理器创建一个新的应用。
2. 创建一个表单模型，`@app/models/EmailForm.php`：

```
<?php
namespace app\models;
use yii\base\Model;
class EmailForm extends Model
{
    public $email;
    public function rules()
    {
        return [
            ['email', 'email']
        ];
    }
}
```

3. 创建控制器`@app/controllers/EmailController.php`：

```
<?php
namespace app\controllers;
use Yii;
use yii\web\Controller;
use app\models\EmailForm;
class EmailController extends Controller
{
    public function actionIndex(){
        $success = false;
        $model = new EmailForm();
        if ($model->load(Yii::$app->request->post()) && $model->validate()) {
            Yii::$app->session->setFlash('success', 'Success!');
        }
        return $this->render('index', [
            'model' => $model,
            'success' => $success,
        ]);
    }
}
```

4. 创建一个视图，`@app/views/email/index.php`：

```
<?php
use yii\helpers\Html;
use yii\captcha\Captcha;
use yii\widgets\ActiveForm;
?>
<?php if (Yii::$app->session->hasFlash('success')): ?>
    <div class="alert alert-success"><?=Yii::$app->session->getFlash('success')?></div>
<?php else: ?>
    <?php $form = ActiveForm::begin()?>
    <div class="control-group">
        <div class="controls">
            <?= $form->field($model, 'email')->textInput(['class' => 'form-control']); ?>
            <?php echo Html::error($model, 'email', ['class' => 'help-block'])?>
        </div>
    </div>
    <?php if (Captcha::checkRequirements() &&
        Yii::$app->user->isGuest): ?>
        <div class="control-group">
            <?= $form->field($model, 'verifyCode')-widget(\yii\captcha\Captcha::classname(), [
                'captchaAction' => 'email/captcha'
            ]) ?>
        </div>
    <?php endif; ?>
    <div class="control-group">
        <label class="control-label" for=""></label>
        <div class="controls">
            <?=Html::submitButton('Submit', ['class' => 'btn btn-success'])?>
        </div>
    </div>
    <?php ActiveForm::end()?>
<?php endif;?>
```

5. 现在，我们有了一个电子邮件提交表单，如下截图所示，它验证了电子邮件字段。让我们添加验证码：

![](../images/406.png)

### 如何做...

1. 首先我们需要自定义表单模型。我们需要添加`$verifyCode`，它会保存输入的验证码，并为它添加一个验证规则：

```
<?php
namespace app\models;
use yii\base\Model;
use yii\captcha\Captcha;
class EmailForm extends Model
{
    public $email;
    public $verifyCode;
    public function rules()
    {
        return [
            ['email', 'email'],
            ['verifyCode', 'captcha', 'skipOnEmpty' => !Captcha::checkRequirements(), 'captchaAction' => 'email/captcha']
        ];
    }
}
```

2. 然后添加一个外部动作到控制器中：

```
public function actions()
{
    return [
        'captcha' => [
            'class' => 'yii\captcha\CaptchaAction',
        ],
    ];
}
```

3. 在视图中，我们需要展示一个额外的字段和验证码图片：

```
...
<?php if (Captcha::checkRequirements() &&
    Yii::$app->user->isGuest): ?>
    <div class="control-group">
        <?=Captcha::widget([
            'model' => $model,
            'attribute' => 'verifyCode',
        ]);?>
        <?php echo Html::error($model, 'verifyCode')?>
    </div>
<?php endif; ?>
...
```

4. 同时，不要忘记在视图的头部添加`Captcha`导入：

```
<?php
    use yii\helpers\Html;
    use yii\captcha\Captcha;
?>
```

5. 完成了。现在你可以运行电子邮件控制器，可以在动作动看到验证码，如下截图所示：

![](../images/407.png)

如果屏幕上没有错误，表单中没有`Captcha`字段，很有可能是因为你没有安装PHP扩展GD或者Imagick。验证码依赖于GD或者Imagick生成图片。我们添加了几个`Captcha::checkRequirement()`检查，所以当图片不会展示时，不使用验证码，应用仍可以正常工作。

### 工作原理...

在视图中，我们调用验证码小部件渲染`img`标签，将`src`属性指向控制器中的验证码动作。在这个动作中，生成了一张带有随机单词的图片。生成的单词需要用户输入到表单中。它被存储在一个用户session中，并向用户展示了一张图片。党用户输入电子邮箱和验证码到表单中时，我们将这些值赋给表单模型，并进行校验。对于验证码的校验，我们使用`CaptchaValidator`。它会从用户session获取验证码，并和输入的验证码进行比较。如果不匹配，模型数据会被认为是不合法的。

### 更多...

如果你使用`accessRules`控制器方法来限制对控制器动作的访问，不要忘记授权每一个人都能访问他们：

```
public function behaviors()
{
    return [
        'access' => [
            'class' => AccessControl::className(),
            'rules' => [
                [
                    'actions' => ['index', 'captcha'],
                    'allow' => true,
                ]
            ],
        ],
    ];
}
```

## 自定义Captcha

标准的Yii验证码已经足够防护垃圾信息，但是有些情况下，你可能需要自定义验证码，例如：

- 你面对一个垃圾机器人，它可以从图片中读取文字，你需要添加更多的安全措施
- 你希望让验证码更加简单和有趣

在我们的例子中，我们将会修改Yii的验证码，它要求用户解决一个简单的算术问题，而不只是简单的重复图片中文字的内容。

### 准备

这个例子一开始，我们会利用*添加和自定义CaptchaWidget*的结果。或者也可以使用其它使用了验证码的表单，因为我们不需要修改很多已有的代码。

### 如何做...

我们需要自定义`CaptchaAction`，它会生成验证码并将其生成图片。这个验证码应该是一个随机数字，并且图片应该是一个有相同结果的算术表达式：

1. 创建`@app/components/MathCaptchaAction.php`：

```
<?php
namespace app\components;
use \Yii;
use yii\captcha\CaptchaAction;
class MathCaptchaAction extends CaptchaAction
{
    protected function renderImage($code)
    {
        return parent::renderImage($this->getText($code));
    }
    protected function generateVerifyCode()
    {
        return mt_rand((int)$this->minLength,
            (int)$this->maxLength);
    }
    protected function getText($code)
    {
        $code = (int) $code;
        $rand = mt_rand(1, $code-1);
        $op = mt_rand(0, 1);
        if ($op) {
            return $code - $rand . " + " . $rand;
        }
        else {
            return $code + $rand . " - " . " " . $rand;
        }
    }
}
```

2. 在我们的控制器`actions`方法中，我们需要将`CaptchaAction`替换成自己的验证码动作，如下：

```
public function actions()
{
    return [
        'captcha' => [
            'class' => 'app\components\MathCaptchaAction',
            'minLength' => 1,
            'maxLength' => 10,
        ],
    ];
}
```

3. 运行你的表单，尝试新的验证码。它将会展示一个算术表达式，你需要输入它的答案，如下截图所示：

![](../images/408.png)

我们重写了两个`CaptchaAction`方法，在`generateVerifyCode()`中，我们生成了一个随机数而不是文本。然后我们需要渲染的是一个表达式，而不是文本，我们需要重写`renderImage`。表达式是由我们自定义`getText()`方法生成的。`$minLength`和`$maxLength`属性已经在`CaptchaAction`定义了，所以我们不需要将它们加入到`MathCaptchaAction`类中。

### 参考

欲了解更多信息，参考如下链接：

- [http://www.yiiframework.com/doc-2.0/yii-captcha-captcha.html](http://www.yiiframework.com/doc-2.0/yii-captcha-captcha.html)
- [http://www.yiiframework.com/doc-2.0/yii-captcha-captchaaction.html](http://www.yiiframework.com/doc-2.0/yii-captcha-captchaaction.html)
- 第二章*路由，控制器，视图*中的*使用独立动作*小节

## 创建一个自定义输入小部件

Yii有一套非常好的表单小部件，但和其它框架一样，Yii并不能涵盖所有。在本小节中，我们将会学习如何创建自己的输入小部件。这里我们将创建一个范围输入小部件。

### 准备

按照官方指南[http://www.yiiframework.com/doc-2.0/guide-start-installation.html](http://www.yiiframework.com/doc-2.0/guide-start-installation.html)的描述，使用Composer包管理器创建一个新的应用。

### 如何做...

1. 创建一个小组件文件`@app/components/RangeInputWidget.php`：

```
<?php
namespace app\components;
use yii\base\Exception;
use yii\base\Model;
use yii\base\Widget;
use yii\helpers\Html;
class RangeInputWidget extends Widget
{
    public $model;
    public $attributeFrom;
    public $attributeTo;
    public $htmlOptions = [];
    protected function hasModel()
    {
        return $this->model instanceof Model&&
        $this->attributeFrom !== null&& $this->attributeTo !== null;
    }
    public function run()
    {
        if (!$this->hasModel()) {
            throw new Exception('Model must be set');
        }
        return Html::activeTextInput($this->model, $this->attributeFrom, $this->htmlOptions) 
            .' &rarr; ' 
            .Html::activeTextInput($this->model, $this->attributeTo, $this->htmlOptions);
    }
}
```

2. 创建一个控制器文件`@app/controllers/RangeController.php`：

```
<?php
namespace app\controllers;
use Yii;
use yii\web\Controller;
use app\models\RangeForm;
class RangeController extends Controller
{
    public function actionIndex()
    {
        $model = new RangeForm();
        if ($model->load(Yii::$app->request->post()) &&
            $model->validate()) {
            Yii::$app->session->setFlash('rangeFormSubmitted',
                'The form was successfully processed!'
            );
        }
        return $this->render('index', array(
            'model' => $model,
        ));
    }
}
```

3. 创建一个表单文件`@app/models/RangeForm.php`：

```
<?php
namespace app\models;
use yii\base\Model;
class RangeForm extends Model
{
    public $from;
    public $to;
    public function rules()
    {
        return [
            [['from', 'to'], 'number', 'integerOnly' => true],
            ['from', 'compare', 'compareAttribute' => 'to',
                'operator' => '<='],
        ];
    }
}
```

4. 创建一个视图文件`@app/views/range/index.php`：

```
<?php
use yii\helpers\Html;
use yii\bootstrap\ActiveForm;
use app\components\RangeInputWidget;
?>

<h1>Range form</h1>
<?php if (Yii::$app->session->hasFlash('rangeFormSubmitted')):
    ?>
    <div class="alert alert-success">
        <?= Yii::$app->session->getFlash('rangeFormSubmitted');
        ?>
    </div>
<?php endif?>
<?= Html::errorSummary($model, ['class'=>'alert alert-danger'])?>
<?php $form = ActiveForm::begin([
    'options' => [
        'class' => 'form-inline'
    ]
]); ?>
<div class="form-group">
    <?= RangeInputWidget::widget([
        'model' => $model,
        'attributeFrom' => 'from',
        'attributeTo' => 'to',
        'htmlOptions' => [
            'class' =>'form-control'
        ]
    ]) ?>
</div>
<?= Html::submitButton('Submit', ['class' => 'btn btn-primary', 'name' => 'contact-button']) ?>
<?php ActiveForm::end(); ?>

```

5. 打开网页`index.php?r=range`运行`range`控制器：

![](../images/409.png)

6. 第一个文本输入字段输入200，第二个输入300：

![](../images/410.png)

7. 如果第一个值比第二个值大，小部件会输出一个错误。尝试输入正确的值，分别输入100和200：

![](../images/411.png)

### 工作原理...

范围输入小部件需要如下四个参数：

- `model`：如果没有设置，会抛出一个异常
- `attributeFrom`：用于设置范围的最小值
- `attributeTo`：用于设置范围的最大值
- `htmlOptions`：会被传递给每一个输入

这个小部件用在表单验证，被用于检查第一个值是否小于等于第二个值。

### 更多...

Yii2框架有一个官方Twitter Bootstrap扩展，它提供了一系列Twitter Bootstrap小部件的封装。在你使用自己的小部件时，检查有否有Bootstrap可用[http://www.yiiframework.com/doc-2.0/extbootstrap-index.html](http://www.yiiframework.com/doc-2.0/extbootstrap-index.html)。

### 参考

欲了解更多关于小部件的信息，可以使用如下资源：

- [http://www.yiiframework.com/doc-2.0/yii-base-widget.html](http://www.yiiframework.com/doc-2.0/yii-base-widget.html)
- [https://github.com/yiisoft/yii2-bootstrap/blob/master/docs/guide/usage-widgets.md](https://github.com/yiisoft/yii2-bootstrap/blob/master/docs/guide/usage-widgets.md)

## Tabular输入

### 准备

### 如何做...

### 工作原理...

### 参考


## 条件校验器

### 准备

### 如何做...

### 工作原理...

### 参考


## 带有多个模型的复杂表单

### 准备

### 如何做...

### 工作原理...

### 参考


## 依赖AJAX的下拉列表

### 准备

### 如何做...

### 工作原理...



## AJAX校验器

### 准备

### 如何做...

### 工作原理...

### 参考


## 创建一个自定义客户端的校验器

### 准备

### 如何做...

### 工作原理...

### 更多...

### 参考













![](../images/412.png)

![](../images/413.png)

![](../images/414.png)

![](../images/415.png)

![](../images/416.png)

![](../images/417.png)

![](../images/418.png)

![](../images/419.png)

![](../images/420.png)

![](../images/421.png)