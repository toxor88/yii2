フォームを作成する
==================

Yii においてフォームを使用するための主たる方法は [[yii\widgets\ActiveForm]] によるものです。
フォームがモデルに基づくものである場合はこの方法を優先すべきです。
これに加えて、[[yii\helpers\Html]] にはいくつかの有用なメソッドがあり、どんなフォームでもボタンやヘルプテキストを追加するのには、通常はそれらが使われます。

フォームは、クライアント側で表示されるものですが、たいていの場合、サーバ側でフォームの入力を検証するために使われる、対応する [モデル](structure-models.md) を持ちます
(入力の検証の詳細については、[入力を検証する](input-validation.md) の節を参照してください)。
モデルに基づくフォームを作成する場合、最初のステップは、モデルそのものを定義することです。
モデルは、データベースの何らかのデータを表現するために [アクティブレコード](db-active-record.md) クラスから派生させるか、あるいは、任意の入力、例えばログインフォームの入力を捕捉するために ([[yii\base\Model]] から派生させた) 汎用的な Model クラスとすることが出来ます。
以下の例においては、ログインフォームのために汎用的なモデルを使う方法を示します。

```php
<?php

class LoginForm extends \yii\base\Model
{
    public $username;
    public $password;

    public function rules()
    {
        return [
            // 検証規則をここで定義
        ];
    }

```

コントローラにおいて、このモデルのインスタンスをビューに渡し、ビューでは [[yii\widgets\ActiveForm|ActiveForm]] ウィジェットがフォームを表示するのに使われます。

```php
<?php
use yii\helpers\Html;
use yii\widgets\ActiveForm;

$form = ActiveForm::begin([
    'id' => 'login-form',
    'options' => ['class' => 'form-horizontal'],
]) ?>
    <?= $form->field($model, 'username') ?>
    <?= $form->field($model, 'password')->passwordInput() ?>

    <div class="form-group">
        <div class="col-lg-offset-1 col-lg-11">
            <?= Html::submitButton('ログイン', ['class' => 'btn btn-primary']) ?>
        </div>
    </div>
<?php ActiveForm::end() ?>
```

上記のコードでは、[[yii\widgets\ActiveForm::begin()|ActiveForm::begin()]] がフォームのインスタンスを作成するだけでなく、フォームの開始をマークしています。
[[yii\widgets\ActiveForm::begin()|ActiveForm::begin()]] と [[yii\widgets\ActiveForm::end()|ActiveForm::end()]] の間に置かれた全てのコンテントが `<form>` タグによって囲まれます。
どのウィジェットでも同じですが、ウィジェットをどのように構成すべきかに関するオプションを指定するために、`begin` メソッドに配列を渡すことが出来ます。
この例では、追加の CSS クラスと要素を特定するための ID が渡されて、開始 `<form>` タグに適用されています。
利用できるオプションはすべて [[yii\widgets\ActiveForm]] の API ドキュメントに記されていますので参照してください。

フォームの中では、フォームの要素を作成し、それと一緒に要素のラベルと、そして、適用できる JavaScript の検証メソッドがあれば、それも追加するために、ActiveForm ウィジェットの [[yii\widgets\ActiveForm::field()|ActiveForm::field()]] メソッドが呼ばれています。
このメソッドは、[[yii\widgets\ActiveField]] のインスタンスを返します。
このメソッドの呼び出し結果を直接にエコーすると、結果は通常の (text の) インプットになります。
出力結果をカスタマイズするためには、このメソッドの呼び出しに追加の [[yii\widgets\ActiveField|ActiveField]] のメソッドをチェーンします。

```php
// パスワードのインプット
<?= $form->field($model, 'password')->passwordInput() ?>
// ヒントとカスタマイズしたラベルを追加
<?= $form->field($model, 'username')->textInput()->hint('お名前を入力してください')->label('お名前') ?>
// HTML5 のメールインプット要素を作成
<?= $form->field($model, 'email')->input('email') ?>
```

これで、フォームのフィールドによって定義された [[yii\widgets\ActiveField::$template|テンプレート]] に従って、`<label>`、`<input>` など、全てのタグが生成されます。
インプットフィールドの名前は、モデルの [[yii\base\Model::formName()|フォーム名]] と属性から自動的に決定されます。
例えば、上記の例における `username` 属性のインプットフィールドの名前は `LoginForm[username]` となります。
この命名規則の結果として、ログインフォームの全ての属性が配列として、サーバ側においては `$_POST['LoginForm']` に格納されて利用できることになります。

モデルの属性を指定するために、もっと洗練された方法を使うことも出来ます。
例えば、複数のファイルをアップロードしたり、複数の項目を選択したりする場合に、属性の名前に `[]` を付けて、属性が配列の値を取ることが出来ることを指定することが出来ます。

```php
// 複数のファイルのアップロードを許可する
echo $form->field($model, 'uploadFile[]')->fileInput(['multiple'=>'multiple']);

// 複数の項目をチェックすることを許可する
echo $form->field($model, 'items[]')->checkboxList(['a' => 'Item A', 'b' => 'Item B', 'c' => 'Item C']);
```

フォームに HTML タグを追加するためには、素の HTML を使うか、または、上記の例の [[yii\helpers\Html::submitButton()|Html::submitButton()]] のように、[[yii\helpers\Html|Html]] ヘルパクラスのメソッドを使うことが出来ます。

> Tip|ヒント: あなたのアプリケーションで Twitter Bootstrap CSS を使っている場合は、[[yii\widgets\ActiveForm]] の代りに [[yii\bootstrap\ActiveForm]] を使うのが良いでしょう。
> これは ActiveForm クラスのエクステンションであり、bootstrap CSS フレームワークで使用するための追加のスタイルをサポートしています。

> tip|ヒント: 必須フィールドをアスタリスク付きのスタイルにするために、次の CSS を使うことが出来ます。
>
>```css
>div.required label:after {
>    content: " *";
>    color: red;
>}
>```

次の節 [入力を検証する](input-validation.md) は、送信されたフォームデータのサーバ側でのバリデーションと、ajax バリデーションおよびクライアント側バリデーションを扱います。

フォームのもっと複雑な使用方法については、以下の節を読んで下さい。

- [表形式インプットのデータ収集](input-tabular-input.md) - 同じ種類の複数のモデルのデータを収集する。
- [複数のモデルを持つ複雑なフォーム](input-multiple-models.md) - 同じフォームの中で複数の異なるモデルを扱う。
- [ファイルをアップロードする](input-file-upload) - フォームを使ってファイルをアップロードする方法。
