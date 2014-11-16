エイリアス
=======

ファイルパスや URL を表すのにエイリアスを使用すると、あなたはプロジェクト内で絶対パスや URL をハードコードする必要がなくなります。エイリアスは、通常のファイルパスや URL とは区別するために、 `@` 文字で始まる必要があります。Yii はすでに利用可能な多くの事前定義エイリアスを持っています。
たとえば、 `@yii` というエイリアスは Yii フレームワークのインストールパスを表し、 `@web` は現在実行中の Web アプリケーションのベース URL を表します。


エイリアスの定義 <a name="defining-aliases"></a>
----------------

[[Yii::setAlias()]] を呼び出すことにより、ファイルパスまたは URL のエイリアスを定義することができます。

```php
// ファイルパスのエイリアス
Yii::setAlias('@foo', '/path/to/foo');

// URL のエイリアス
Yii::setAlias('@bar', 'http://www.example.com');
```

> 補足: エイリアスされているファイルパスやURLは、必ずしも既存のファイルまたはリソースを参照しない場合があります。

定義済みのエイリアスがあれば、スラッシュ `/` に続けて1つ以上のパスセグメントを追加することで（[[Yii::setAlias()]]
の呼び出しを必要とせずに) 新しいエイリアスを導出することができます。 [[Yii::setAlias()]] を通じて定義されたエイリアスは
*ルートエイリアス* となり、それから派生したエイリアスは *派生エイリアス* になります。たとえば、 `@foo` がルートエイリアスなら、
`@foo/bar/file.php` は派生エイリアスです。

エイリアスを、他のエイリアス (ルートまたは派生のいずれか) を使用して定義することができます:

```php
Yii::setAlias('@foobar', '@foo/bar');
```

ルートエイリアスは通常、 [ブートストラップ](runtime-bootstrapping.md) 段階で定義されます。
たとえば、[エントリスクリプト](structure-entry-scripts.md) で [[Yii::setAlias()]] を呼び出すことができます。
便宜上、 [アプリケーション](structure-applications.md) は、`aliases` という名前の書き込み可能なプロパティを提供しており、
それをアプリケーション [コンフィギュレーション](concept-configurations.md) で設定することが可能です。

```php
return [
    // ...
    'aliases' => [
        '@foo' => '/path/to/foo',
        '@bar' => 'http://www.example.com',
    ],
];
```


エイリアスの解決 <a name="resolving-aliases"></a>
-----------------

You can call [[Yii::getAlias()]] to resolve a root alias into the file path or URL it represents.
The same method can also resolve a derived alias into the corresponding file path or URL:

```php
echo Yii::getAlias('@foo');               // displays: /path/to/foo
echo Yii::getAlias('@bar');               // displays: http://www.example.com
echo Yii::getAlias('@foo/bar/file.php');  // displays: /path/to/foo/bar/file.php
```

The path/URL represented by a derived alias is determined by replacing the root alias part with its corresponding
path/URL in the derived alias.

> Note: The [[Yii::getAlias()]] method does not check whether the resulting path/URL refers to an existing file or resource.


A root alias may also contain slash `/` characters. The [[Yii::getAlias()]] method
is intelligent enough to tell which part of an alias is a root alias and thus correctly determines
the corresponding file path or URL:

```php
Yii::setAlias('@foo', '/path/to/foo');
Yii::setAlias('@foo/bar', '/path2/bar');
Yii::getAlias('@foo/test/file.php');  // displays: /path/to/foo/test/file.php
Yii::getAlias('@foo/bar/file.php');   // displays: /path2/bar/file.php
```

If `@foo/bar` is not defined as a root alias, the last statement would display `/path/to/foo/bar/file.php`.


Using Aliases <a name="using-aliases"></a>
-------------

Aliases are recognized in many places in Yii without needing to call [[Yii::getAlias()]] to convert
them into paths or URLs. For example, [[yii\caching\FileCache::cachePath]] can accept both a file path
and an alias representing a file path, thanks to the `@` prefix which allows it to differentiate a file path
from an alias.

```php
use yii\caching\FileCache;

$cache = new FileCache([
    'cachePath' => '@runtime/cache',
]);
```

Please pay attention to the API documentation to see if a property or method parameter supports aliases.


Predefined Aliases <a name="predefined-aliases"></a>
------------------

Yii predefines a set of aliases to easily reference commonly used file paths and URLs:

- `@yii`, the directory where the `BaseYii.php` file is located (also called the framework directory)
- `@app`, the [[yii\base\Application::basePath|base path]] of the currently running application
- `@runtime`, the [[yii\base\Application::runtimePath|runtime path]] of the currently running application. Defaults to `@app/runtime`.
- `@webroot`, the Web root directory of the currently running Web application. It is determined based on the directory
  containing the entry script.
- `@web`, the base URL of the currently running Web application. It has the same value as [[yii\web\Request::baseUrl]].
- `@vendor`, the [[yii\base\Application::vendorPath|Composer vendor directory]]. Defaults to `@app/vendor`.
- `@bower`, the root directory that contains [bower packages](http://bower.io/). Defaults to `@vendor/bower`.
- `@npm`, the root directory that contains [npm packages](https://www.npmjs.org/). Defaults to `@vendor/npm`.

The `@yii` alias is defined when you include the `Yii.php` file in your [entry script](structure-entry-scripts.md). The rest of the aliases are defined in the application constructor when applying the application
[configuration](concept-configurations.md).


Extension Aliases <a name="extension-aliases"></a>
-----------------

An alias is automatically defined for each [extension](structure-extensions.md) that is installed via Composer.
Each alias is named after the root namespace of the extension as declared in its `composer.json` file, and each alias
represents the root directory of the package. For example, if you install the `yiisoft/yii2-jui` extension,
you will automatically have the alias `@yii/jui` defined during the [bootstrapping](runtime-bootstrapping.md) stage, equivalent to:

```php
Yii::setAlias('@yii/jui', 'VendorPath/yiisoft/yii2-jui');
```
