Yii 2 日本語ドキュメント
===========================

Yii 2.0 のドキュメントを日本語翻訳するプロジェクトです。

- メンバーはだれでも master に push/merge できます。
- だれでも、コミット差分にコメントを残し、作業者に通知することができます。
- ひとまとまりの作業をやるときは、新規 issues で開始を宣言します。

## 含むもの

- [ユーザーガイド](docs/guide-ja)
- [開発者ガイド](docs/internals-ja)
- [メッセージファイル](framework/messages/ja/yii.php)

## 本家への申請について

いつでもだれでも、このリポジトリから翻訳コミットを抽出し、yiisoft/yii2 へのプルリクエストを作ることができます。

原則、Yii の翻訳はこのリポジトリにいったん登録し、ここの master が最新であることを確認してから、本家に送るようにします。

もし、本家の翻訳ファイルが直接変更された場合は、1回のコミットで「いついつの本家からの取り込み」がわかるようにします。(英語のファイルでも同様です。英文の diff を見ればどこが変更されたかわかるからです)

### プルリクエスト作成手順

Yii2本家のリポジトリを自分のアカウントにフォークし、開発者ガイドにしたがって upstream を追加、最新の master からブランチを切ります。

```
git clone git@github.com:yourname/yii2.git
cd yii2
git remote add upstream git@github.com:yiisoft/yii2.git
git checkout master
git pull upstream master
git checkout -b translation-ja-xxxx
```

yiijan/yii2-i18n-ja リポジトリを追加し、プルリクエストに含みたいコミットを順に cherry-pick して、GitHubにブランチを送ります。

```
git remote add yii2-i18n-ja git@github.com:yiijan/yii2-i18n-ja.git
git fetch yii2-i18n-ja
git cherry-pick ebfa645a
git cherry-pick 1520e365c
        :
        :
git push origin translation-ja-xxxx
```

自分が送信したいものとは異なるファイルを変更しているコミットは、この cherry-pick から省くことができます。他の人が作業中でも、それを yiijan/yii2-i18n-ja 内に保留したままで、完了しているファイルだけを本家に送ることが可能です。

自分のアカウントに作成されたブランチから、本家にプルリクエストを送ります。
