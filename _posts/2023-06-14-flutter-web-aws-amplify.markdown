---
layout: post
title:  Flutter Web AWS Amplifyでデプロイ
date:   2023-06-13 23:00:00 +0900
categories: webland
tags: aws flutter amplify
---

前回はFlutter WebアプリをJenkinsでCodePipelineでデプロイできるようにしました。

[CodePipeline]({% link _posts/2023-06-11-flutter-web-on-codepipeline.markdown %})

今回はAmplifyを使用してデプロイします。

## GitHubにリポジトリを作成

Flutterのデフォルトプロジェクトをコミットしたリポジトリを作成してください。

参考までに、私が作成したリポジトリを記載しておきます。

<https://github.com/ohmusso/flutter_aws_host>

## 開始

AWS マネジメントコンソールからAWS Amplifyにアクセスします。

下にスクロースすると、Amplifyホスティングがあるので、"使用を開始する"をクリックします。

![コンソール画面](/assets//images/image-2023-06-14-amplify-consol.png)

ホスティングの開始方法ではGitHubを選択して、"続行"をクリックします。

![開始画面](/assets//images/image-2023-06-14-amplify-start.png)

## GitHub リポジトリへのアクセスを許可

クリックすると、パスワード入力画面に移動します。

パスワードを入力すると、AWSの画面に戻ります。

![アクセス許可](/assets//images/image-2023-06-14-amplify-git.png)

## ブランチを選択

リポジトリとブランチを選択します。

次へをクリックします。

![ブランチ選択](/assets//images/image-2023-06-14-amplify-branch.png)

## ビルドの設定

アプリケーション名はデフォルトでリポジトリ名が入力されます。

構築とテストの設定はCodeBuildのbuildspecのようにymlファイルを記述します。

編集ボタンをクリックして以下のコードを入力します。

ymlファイルの詳細は以下の公式ドキュメントを参照してください。

<https://docs.aws.amazon.com/ja_jp/amplify/latest/userguide/build-settings.html>

詳細設定はデフォルトのままです。次へをクリックします。

![ビルドの設定](/assets//images/image-2023-06-14-amplify-buildsetting.png)

```yml
version: 1
frontend:
  phases:
    # IMPORTANT - Please verify your build commands
    build:
      commands:
        # need XZ Utilz to extract flutter from tar.xz
        - yum install xz -y
        # download flutter
        - wget https://storage.googleapis.com/flutter_infra_release/releases/stable/linux/flutter_linux_3.10.2-stable.tar.xz
        # extract flutter
        - tar xf flutter_linux_3.10.2-stable.tar.xz
        # set path to flutter
        - export PATH="$PATH:`pwd`/flutter/bin"
        # build
        - flutter --version
        - flutter build web
        - ls
  artifacts:
    # IMPORTANT - Please verify your build output directory
    files:
      - '**/*'
    baseDirectory: build/web
```

## 確認

入力は以上です。入力内容に問題なければ"保存してデプロイ"をクリックします。

![確認画面](/assets//images/image-2023-06-14-amplify-confirm.png)

## 完了

アプリケーションが作成されます。

デプロイが完了するまで待ちます。

![確認画面](/assets//images/image-2023-06-14-amplify-building.png)

## デプロイ完了

デプロイが完了したら、赤線のリンクをクリックします。

![確認画面](/assets//images/image-2023-06-14-amplify-building.png)

## デプロイしたウェブアプリを表示

作成したアプリが表示されたらOKです。

![デプロイしたWebアプリ](/assets/images/image-2023-06-05-codepipeline-changeapp.png)

## 以上

CodePipelineよりも簡単にデプロイができました。

AmplifyはAmazon DynamoDBテーブルなどのバックエンドのリソースも一緒にデプロイできるようです。
