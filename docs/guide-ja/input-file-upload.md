ファイルをアップロードする
==========================

Yii におけるファイルのアップロードは、フォームモデル、その検証規則、そして、いくらかのコントローラコードによって行われます。
アップロードを適切に処理するために何が必要とされるのか、見ていきましよう。


一つのファイルをアップロードする
--------------------------------

まず最初に、ファイルのアップロードを処理するモデルを作成する必要があります。
次の内容を持つ `models/UploadForm.php` を作って作成してください。

```php
namespace app\models;

use yii\base\Model;
use yii\web\UploadedFile;

/**
 * UploadForm がアップロードのフォームの背後にあるモデルである。
 */
class UploadForm extends Model
{
    /**
     * @var UploadedFile file 属性
     */
    public $file;

    /**
     * @return array 検証規則
     */
    public function rules()
    {
        return [
            [['file'], 'file'],
        ];
    }
}
```

上記のコードにおいて作成した `UploadForm` というモデルは、HTML フォームで `<input type="file">` となる `$file` という属性を持ちます。
この属性は [[yii\validators\FileValidator|FileValidator]] を使用する `file` という検証規則を持ちます。

### フォームのビュー

次に、フォームを表示するビューを作成します。

```php
<?php
use yii\widgets\ActiveForm;
?>

<?php $form = ActiveForm::begin(['options' => ['enctype' => 'multipart/form-data']]) ?>

<?= $form->field($model, 'file')->fileInput() ?>

<button>送信</button>

<?php ActiveForm::end() ?>
```

ファイルのアップロードを可能にする `'enctype' => 'multipart/form-data'` は不可欠です。
`fileInput()` がフォームの入力フィールドを表します。

### コントローラ

そして、フォームとモデルを結び付けるコントローラを作成します。

```php
namespace app\controllers;

use Yii;
use yii\web\Controller;
use app\models\UploadForm;
use yii\web\UploadedFile;

class SiteController extends Controller
{
    public function actionUpload()
    {
        $model = new UploadForm();

        if (Yii::$app->request->isPost) {
            $model->file = UploadedFile::getInstance($model, 'file');

            if ($model->file && $model->validate()) {                
                $model->file->saveAs('uploads/' . $model->file->baseName . '.' . $model->file->extension);
            }
        }

        return $this->render('upload', ['model' => $model]);
    }
}
```

`model->load(...)` の代りに `UploadedFile::getInstance(...)` を使っています。
[[\yii\web\UploadedFile|UploadedFile]] はモデルの検証を実行せず、アップロードされたファイルに関する情報を提供するだけです。
そのため、`$model->validate()` を手作業で実行して、[[yii\validators\FileValidator|FileValidator]] を起動する必要があります。
[[yii\validators\FileValidator|FileValidator]] は、下記のコアコードが示しているように、属性がファイルであることを要求します。

```php
if (!$file instanceof UploadedFile || $file->error == UPLOAD_ERR_NO_FILE) {
    return [$this->uploadRequired, []];  // "ファイルをアップロードしてください。" というエラーメッセージ
}
```

検証が成功したら、ファイルを保存します。

```php
$model->file->saveAs('uploads/' . $model->file->baseName . '.' . $model->file->extension);
```

「ベーシック」アプリケーションテンプレートを使っている場合は、`uploads` フォルダを `web` の下に作成しなければなりません。

以上です。ページをロードして、アップロードを試して見てください。ファイルは `basic/web/uploads` にアップロードされます。

検証
----

It's often required to adjust validation rules to accept certain files only or require uploading. Below we'll review
some common rule configurations.

### Required

If you need to make the file upload mandatory, use `skipOnEmpty` like the following:

```php
public function rules()
{
    return [
        [['file'], 'file', 'skipOnEmpty' => false],
    ];
}
```

### MIME type

It is wise to validate the type of file uploaded. FileValidator has the property `$extensions` for this purpose:

```php
public function rules()
{
    return [
        [['file'], 'file', 'extensions' => 'gif, jpg',],
    ];
}
```

Keep in mind that only the file extension will be validated, but not the actual file content. In order to validate the content as well, use the `mimeTypes` property of `FileValidator`:

```php
public function rules()
{
    return [
        [['file'], 'file', 'extensions' => 'jpg, png', 'mimeTypes' => 'image/jpeg, image/png',],
    ];
}
```

[List of common media types](http://en.wikipedia.org/wiki/Internet_media_type#List_of_common_media_types)

### Image properties

If you upload an image, [[yii\validators\ImageValidator|ImageValidator]] may come in handy. It verifies if an attribute
received a valid image that can be then either saved or processed using the [Imagine Extension](https://github.com/yiisoft/yii2/tree/master/extensions/imagine).

Uploading multiple files
------------------------

If you need to upload multiple files at once, some adjustments are required.
 
Model:

```php
class UploadForm extends Model
{
    /**
     * @var UploadedFile|Null file attribute
     */
    public $file;

    /**
     * @return array the validation rules.
     */
    public function rules()
    {
        return [
            [['file'], 'file', 'maxFiles' => 10], // <--- here!
        ];
    }
}
```

View:

```php
<?php
use yii\widgets\ActiveForm;

$form = ActiveForm::begin(['options' => ['enctype' => 'multipart/form-data']]);
?>

<?= $form->field($model, 'file[]')->fileInput(['multiple' => true]) ?>

    <button>Submit</button>

<?php ActiveForm::end(); ?>
```

The difference is the following line:

```php
<?= $form->field($model, 'file[]')->fileInput(['multiple' => true]) ?>
```

Controller:

```php
namespace app\controllers;

use Yii;
use yii\web\Controller;
use app\models\UploadForm;
use yii\web\UploadedFile;

class SiteController extends Controller
{
    public function actionUpload()
    {
        $model = new UploadForm();

        if (Yii::$app->request->isPost) {
            $model->file = UploadedFile::getInstances($model, 'file');
            
            if ($model->file && $model->validate()) {
                foreach ($model->file as $file) {
                    $file->saveAs('uploads/' . $file->baseName . '.' . $file->extension);
                }
            }
        }

        return $this->render('upload', ['model' => $model]);
    }
}
```

There are two differences from single file upload. First is that `UploadedFile::getInstances($model, 'file');` used
instead of `UploadedFile::getInstance($model, 'file');`. The former returns instances for **all** uploaded files while
the latter gives you only a single instance. The second difference is that we're doing `foreach` and saving each file.
