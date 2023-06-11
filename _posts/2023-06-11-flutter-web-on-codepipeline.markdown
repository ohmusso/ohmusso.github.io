---
layout: post
title:  Flutter Web CodePipelineでデプロイ
date:   2023-06-11 12:00:00 +0900
categories: How-To
tags: aws flutter codepipeline
---

前回はFlutter WebアプリをJenkinsでデプロイできるようにしました。

[半自動デプロイ]({% link _posts/2023-05-28-flutter-web-on-ec2-linux.markdown %})

今回はCodePipelineを使用してS3にデプロイします。

## 構成

![構成図](/assets//images/image-2023-06-11-structure.png)

## GitHubにリポジトリを作成

Flutterのデフォルトプロジェクトをコミットしたリポジトリを作成してください。

参考までに、私が作成したリポジトリを記載しておきます。

<https://github.com/ohmusso/flutter_aws_host>

## S3 バケットの作成

静的ウェブサイトをホストできるS3バケットを作成します。

ここにビルドしたFlutter Webを入れます。

手順の記載は省略します。検索すると記事がたくさん出てきますので。

インデックスドキュメントだけ"index.html"を設定してください。

### S3静的ウェブサイトの参考記事

以下の方の記事とか分かりやすいです。

<https://in-housese.hatenablog.com/entry/2021/05/06/181746>

## CodePipelineの作成

AWSコンソールからCodePipelineのページを開きます。

パイプラインの作成ボタンがあるので、クリックします。

### Step1

パイプラインの名前を入力すること以外はデフォルトのままです。

![Step1](/assets/images/image-2023-06-05-codepipeline-step1.png)

### Step2

ソースをどこから取ってくるかを設定します。

ここではGitHubにします。

#### ソースプロバイダ

GitHub(バージョン1)は非推奨となっていましたので、バージョン2を選択します。

#### 接続

Githubに接続をクリックして新しい接続を作成します。

別のウインドが立ち上がります。

![Githubに接続](/assets/images/image-2023-06-05-codepipeline-step2-github.png)

##### 接続名

名前を付けます。タグの設定は任意です。

GitHubに接続するをクリックすると、GitHubのページが立ち上がります。

![Githubに接続](/assets/images/image-2023-06-05-codepipeline-step2-github-connect.png)

##### アクセス許可

AWSサービスのアクセスを許可します。

緑のボタンをクリックします。少し待つとAWSの画面に戻ります。

※GitHubのアカウントを持っていない、ブラウザの認証情報が残っていない場合は以下の画面が表示されないかもしれません。

![Githubに接続](/assets/images/image-2023-06-05-codepipeline-step2-github-permission.png)

##### 新しいアプリをインストールする

AWSアカウントにGitHubアプリをインストールします。こんなことできるのですね。

新しいアプリをインストールするをクリックすると、またGitHubの画面が立ち上がります。

![GitHubアプリをインストール](/assets/images/image-2023-06-05-codepipeline-step2-github-app.png)

リポジトリを選択します。ビルドするリポジトリだけを選択しました。アカウント内の全てのリポジトリを選択することも可能です。

ただし、アカウント内のリポジトリしか選べないようです。
共同開発するときは共同開発用のGitHubアカウントが必要かもしれませんね。

Installを選択すると、パスワード入力画面に移動します。パスワードを入力すると、AWSの画面に戻ります。

![リポジトリ選択](/assets/images/image-2023-06-05-codepipeline-step2-github-repo.png)

以下のようにGitHubアプリのところに数字(GitHubのアカウント名)が入ります。

これで自分のアカウントと紐づいたGitHubアプリがインストールされました。

右下の接続ボタンをクリックすると、Step2の画面に戻ります。

![GitHub接続設定が完了](/assets/images/image-2023-06-05-codepipeline-step2-github-install.png)

### Step2 完成

”接続する準備が完了しました”が表示されます。

* リポジトリ名とブランチ名を設定します。

* 検出オプションにチェックを入れます。
リポジトリへのPushによりパイプラインが動きます。

* 出力アーティファクト形式は完全クローンにします。
こちらを選択すると、ビルド実行時にリポジトリを自動でクローンしてくれます。
ただし、選択すると表示されるインフォメーションのとおり、追加のアクセス許可が必要です。後で解説します。

この画面の設定は以上です。次にをクリックします。

![Step2完成](/assets/images/image-2023-06-05-codepipeline-step2-comp.png)

### Step3

ビルドステージを作成します。

#### プロバイダーを構築する

AWS CodeBuildとJenkinsを選択できます。

AWS CodeBuildを選択します。

#### リージョン

東京を選択します。

先ほど作ったGitHubとの接続と同じリージョンにする必要があるみたいです。

#### CodeBuildプロジェクトの作成

プロジェクトを作成するを選択して新しくCodeBuildプロジェクトを作ります。別のウインドが立ち上がります。

![プロジェクトの作成](/assets/images/image-2023-06-05-codepipeline-step3-start.png)

##### プロジェクト名

以下のように入力します。

![プロジェクト名](/assets/images/image-2023-06-05-codepipeline-step3-build-setting.png)

##### 環境

以下のように入力します。

* Linuxのところは最新バージョンを選びました。

* サービスロールは新しく作成します。後でgitからcloneするための権限を追加します。

* 追加設定はデフォルトのままです。

![環境設定](/assets/images/image-2023-06-05-codepipeline-step3-build-env.png)

##### Buildspec

ビルドの本体です。ここにビルドやテストのコマンドを記述します。

ビルドコマンドの挿入を選択してエディタに切り替えを選択すると、入力画面が表示されます。

![Buildspec](/assets/images/image-2023-06-05-codepipeline-step3-build-buildspec.png)

以下をエディタにコピペします。

FlutterのSDKをダウンロードして、Flutter Webのビルドを行います。

ソースコードは自動で作業ディレクトリにクローンされるので、ソースコードを取得するコマンドは不要です。

buildspecの公式リファレンスは以下になります。

<https://docs.aws.amazon.com/ja_jp/codebuild/latest/userguide/build-spec-ref.html>

``` yaml
version: 0.2
env:
  shell: bash
phases:
  install:
    commands:
      - wget https://storage.googleapis.com/flutter_infra_release/releases/stable/linux/flutter_linux_3.10.2-stable.tar.xz
      - tar xf flutter_linux_3.10.2-stable.tar.xz
      - ls flutter
      - ls
  build:
    commands:
      - export PATH="$PATH:`pwd`/flutter/bin"
      - flutter --version
      - flutter build web
artifacts:
  files:
   - '**/*'
  base-directory: build/web

```

##### バッチ設定

バッチ設定は不要です。

##### ログ

ビルドログが見たいので以下のように設定します。

![ログの設定](/assets/images/image-2023-06-05-codepipeline-step3-build-log.png)

##### CodeBuildプロジェクト作成完了

これで完了です。CodePipelineに進むを選択すると、元の画面に戻ります。

#### ビルドステージの作成完了

これでビルドステージの作成は完了です。

環境変数とビルドタイプはデフォルトのままです。

"次に"をクリックしてください。

![ビルドステージの作成完了](/assets/images/image-2023-06-05-codepipeline-step3-comp.png)

### Step4

デプロイステージを作成します。

S3を選択して先ほど作成したバケットを選択してください。

"デプロイする前にファイルを抽出"をチェックすると、CodeBuildが出力するzipファイルを展開してS3にデプロイしてくれます。

![ビルドステージの作成完了](/assets/images/image-2023-06-05-codepipeline-step4.png)

Step4はこれだけです。"次に"をクリックします。

### レビュー

パイプラインの設定はすべて完了です。

"パイプラインを作成する"をクリックすると、パイプラインが作成されます。

作成が完了すると、パイプラインも実行されます。

![パイプライン](/assets/images/image-2023-06-05-codepipeline-firstrun.png)

## パイプラインのエラー

パイプラインが実行されますが、以下のようにエラーが出ると思います。

CodeBuldで表示をクリックすると、ビルドログを確認できます。

![パイプラインエラー](/assets/images/image-2023-06-05-codepipeline-error.png)

authorization failedとなり、ソースコードのダウンロードに失敗しています。

このエラーはCodeBuildのサービスロールに権限を追加すると解決します。

![パイプラインエラー](/assets/images/image-2023-06-05-codepipeline-errorlog.png)

### CodeBuildのサービスロールに権限を追加

公式ドキュメントに手順が記載されています。これに従って権限を追加します。

<https://docs.aws.amazon.com/ja_jp/codepipeline/latest/userguide/troubleshooting.html#codebuild-role-connections>

#### ソースのARNを確認

パイプライン画面のSourceにある(i)をクリックします。

"ConnectionArn"が表示されているので、これをメモします。

![ソースARN](/assets/images/image-2023-06-05-codepipeline-source-arn.png)

#### CodeBuildサービスロールに移動

CodeBuildのビルドの詳細画面からサービスロールに移動します。

![ビルドの詳細](/assets/images/image-2023-06-05-codepipeline-builddetail.png)

下へスクロールしていくと、サービスロールが記載されています。

クリックするとIAMの画面に移動します。

![サービスロール](/assets/images/image-2023-06-05-codepipeline-servicerole.png)

#### サービスロールにポリシーを追加

git cloneするためのポリシーを追加します。

"インラインポリシーを作成"をクリックします。

![ポリシーの追加](/assets/images/image-2023-06-05-codepipeline-inlinepolicy.png)

#### インラインポリシーを記述

jsonでインラインポリシーを記述します。

"Resource"には先ほど確認したソースのARNを記述します。

完了したら、右下の"ポリシーの確認"をクリックします。

![jsonの記述](/assets/images/image-2023-06-05-codepipeline-inlinepolicy-json.png)

#### ポリシー名を入力

ポリシーの名前を入力して右下の"ポリシーの作成"をクリックすると、ポリシーが作成されてサービスロールにアタッチされます。

サービスロールの変更は以上です。

### ビルドを再試行

パイプラインに戻り、ビルドの再試行を行います。

![ビルドを再試行](/assets/images/image-2023-06-05-codepipeline-rebuild.png)

Deployまで成功したら完了です。

### ウェブサイトにアクセス

AWSコンソールから作成したS3のバケットに移動します。
バケットにビルド結果が格納されています。

プロパティをクリックします。

![S3バケット](/assets/images/image-2023-06-05-codepipeline-bucket.png)

下にスクロールすると、"静的ウェブサイトホスティング"があります。

バケットウェブサイトエンドポイントをクリックすると、Flutterで作ったウェブサイトが表示されます。

![完成したWebアプリ](/assets/images/image-2023-05-27-flutter-couter-app.png)

### 変更をpush

色を変える変更を行ってpushすると、パイプラインが実行されます。

![色の変更をpush](/assets/images/image-2023-06-05-codepipeline-changecolor.png)

ウェブサイトにアクセスすると、色が変わります。正しくデプロイされてますね。

![色を変更したWebアプリ](/assets/images/image-2023-06-05-codepipeline-changeapp.png)

#### 403 Forbiddenが表示される場合

エンドポイントにアクセスして、以下のような画面が表示される場合はバケットポリシーが記述されていないと思われます。

![S3バケット](/assets/images/image-2023-06-05-codepipeline-forbidden.png)

以下の画面からバケットポリシーを記述します。

![S3バケット](/assets/images/image-2023-06-05-codepipeline-bucketpolicy.png)

エディタが立ち上がるので、以下のjsonをコピペします。

"codebuild-flutter-web"バケットに対する読み取りを許可します。

``` json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::codebuild-flutter-web/*"
        }
    ]
}
```

## 以上

S3にデプロイすると、CloudFrontを使用した拡張(コンテンツキャッシュ、HTTPS化、セキュリティ強化)が行えます。

次はAWS Amplifyを使ってデプロイしてみます。
