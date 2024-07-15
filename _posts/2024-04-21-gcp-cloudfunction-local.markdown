---
layout: post
title:  Cloud Functions ローカル環境 httpリクエスト
date:   2024-04-21 13:00:00 +0900
categories: webland
tags: CloudFunctions
mermaid: false
---

Cloud Functionsのローカル環境を構築するチュートリアルにトライしてみました。

そのままトライするのではなく、チュートリアルのアプリはhttpリクエストに対してhellow worldを返すだけですが、アプリ内でhttpリクエストを行い、結果を返すアプリに変更してみます。

Cloud Functionsのプログラムはnodejsで作ります。

## リファレンス

<https://github.com/GoogleCloudPlatform/functions-framework-nodejs?tab=readme-ov-file#functions-framework-for-nodejs>

## nodejsのインストール

Cloud Functionsはいろいろな言語で開発できますが、ここではnodejsを使用します。

以下からインストーラをダウンロードして実行します。

<https://nodejs.org/en/download>

## dockerのインストール

Cloud Functionsは結局のところコンテナで実行されるアプリです。dockerを使用します。

以下の手順に沿ってインストールします。

<https://docs.docker.com/desktop/install/windows-install/#install-docker-desktop-on-windows>

日本版のサイトもあります。

<https://docs.docker.jp/desktop/install/windows-install.html#id4>

## packのインストール

dockerイメージをビルドするツールです。dockerだけでもimageのビルドが可能だと思いますが、packを使うことでdockerfileの記述が不要になります。

<https://buildpacks.io/docs/for-platform-operators/how-to/integrate-ci/pack/>

### chocolateのインストール

Windowsでpackをインストールする場合、上記の公式サイトにいくつか方法が記載されていますが、ここではchocolateを使用します。

chocolateはwindowsでのパッケージ管理ツールの一つのようです。以下のコマンドでインストールできます。

``` powershell
# 管理者権限で実行
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

### chocolateでpackをインストール

以下のコマンドでインストールできます。

``` powershell
# 管理者権限で実行
choco install pack --version=0.33.2
```

## チュートリアルアプリの作成

公式の手順通りです。

<https://github.com/GoogleCloudPlatform/functions-framework-nodejs?tab=readme-ov-file#quickstart-set-up-a-new-project>

## チュートリアルアプリのimageをビルドしてコンテナを実行

公式の手順通りです。

<https://github.com/GoogleCloudPlatform/functions-framework-nodejs?tab=readme-ov-file#quickstart-build-a-deployable-container>

<http://localhost:8080/>にアクセスしてhello worldが表示されたらOKです。

### 補足

imageの作成はpowershellだと以下のコマンドになります。

``` powershell
pack build `
 --builder gcr.io/buildpacks/builder:v1 `
 --env GOOGLE_FUNCTION_SIGNATURE_TYPE=http `
 --env GOOGLE_FUNCTION_TARGET=helloWorld `
 my-first-function
```

以下のようにイメージが作成されます。

![docker image](/assets/images/image-2024-04-21-gcp-cloudfunctions-local-image.png)

※image内の容量が足りないと"Timer: Saving index.docker.io/library/my-first-function:latest..."で止まります。

## チュートリアルアプリの修正

hello worldを返すだけでしたが、httpリクエストを送信して、クライアントにその結果を返す処理に修正します。

### index.jsを更新

Cloud Functionsでhttpリクエストを送信するサンプルコードが公式であります。これをベースにします。

まず、index.jsにコピペします。

<https://cloud.google.com/functions/docs/samples/functions-concepts-requests?hl=ja#functions_concepts_requests-nodejs>

次に以下の4点を修正します。

* index.js
  * urlを修正します。私は簡単なjsonを返してくれるサイト見つけて、そのurlにしました。
    * 例): <http://ip.jsontest.com>
  * httpリクエストに成功したら、bodyをテキストで返すように修正します。
* package.json
  * scripts startの関数名を"helloWorld"から"makeRequest"に変更します。
* docker imageのビルドコマンドのGOOGLE_FUNCTION_TARGETも"helloWorld"から"makeRequest"に変更します。

index.jsは以下のようになりました。

``` node
const fetch = require('node-fetch');
const functions = require('@google-cloud/functions-framework');

/**
 * HTTP Cloud Function that makes an HTTP request
 *
 * @param {Object} req Cloud Function request context.
 * @param {Object} res Cloud Function response context.
 */
functions.http('makeRequest', async (req, res) => {
  const url = 'http://ip.jsontest.com/'; // URL to send the request to
  const externalRes = await fetch(url);

  if(!externalRes.ok){
    res.send("error");
    return;
  }

  const body = await externalRes.text();
  res.send(body);
});
```

変更したimageのビルドコマンドです。

``` powershell
pack build `
 --builder gcr.io/buildpacks/builder:v1 `
 --env GOOGLE_FUNCTION_SIGNATURE_TYPE=http `
 --env GOOGLE_FUNCTION_TARGET=makeRequest `
 my-first-function
```

### packageを更新

サンプルコードでnode-fetchというモジュールを使用していますので、npmインストールします。node-fetchはv2系を指定します。requireでインポートできないためです。

``` script
npm install node-fetch@2
```

コードの修正は以上です。イメージをビルドしてコンテナを実行して<http://localhost:8080/>にアクセスすると、直接<http://ip.jsontest.com/>にアクセスしたときと同じ内容が表示されるはずです。

※コンテナを実行するコマンド

``` powershell
docker run --rm -p 8080:8080 my-first-function
```

## 以上
