---
layout: post
title:  Flutter WebをAmazon Linux2023でホスト Jenkinsでデプロイ
date:   2023-06-04 21:30:00 +0900
categories: webland
tags: aws flutter jenkins
---

以下の記事でFlutter WebアプリをEC2に手動アップロードしてデプロイしました。

[手動デプロイ]({% link _posts/2023-05-27-flutter-web-on-ec2-linux.markdown %})

今回はEC2にJenkinsをインストール、ソースコードをGitHubにおいて、 半自動でデプロイできるようにしました。

* ソースコードをコミット
* jenkinsでボタンを押すと最新のソースコードでデプロイ

## インフラ環境

手動でデプロイしたときと同じ構成ですが、テンプレートには変更点があります。

* apache.yaml
  * インスタンスサイズをt2.micro(無料利用枠あり)からt2.mediumに変更しています。理由はflutterアプリのビルドが1時間経っても終わらなかったので。。。
  * UserDataにJenkinsをインストール/起動するスクリプトを追加しています。
  * UserDataにgitをインストールするスクリプトを追加しています。JnekinsのGitプラグインをインストールしても、Git本体はインストールされないです。
  * UserDataにflutterをインストールするスクリプトを追加しています。
* security_group_apache.yaml
  * インバウンドルールにJenkinsにhttpsアクセスするときのポート8443を許可します。

テンプレートを使っていただければ、必要なソフトがインストールされたサーバが起動します。

## Jenkins 初期設定

### httpsを有効化

httpのままではJenkinsとの通信が暗号化なしでインターネットを通るため、httpsを有効にして通信を暗号化します。
パスワードや秘密鍵を入力しますので。

以下の記事を参考にさせていただきました。

[JenkinsのSSL(https)化](https://sbc-web.hatenablog.jp/entry/2017/05/18/Jenkins%E3%81%AESSL%28https%29%E5%8C%96)

#### 自己証明書を作成

自己証明書をkeytoolコマンドで作成し、権限を変更してjenkinsが読み取れるようにします。

``` bash
# jenkinsのホームディレクトリに移動
cd /var/lib/jenkins
# keytoolを実行
sudo keytool -genkey -alias jenkins-ssl-cert -keyalg RSA -keystore keystore -validity 3650
sudo chown ec2-user:ec2-user keystore
```

以下はkeytoolを実行したときの画面です。
パスワードだけ入力して、他は空です。

![keytool実行](/assets/images/image-2023-06-04-jenkins-https-keystore.png)

#### Jenkins HTTPSの設定

Jenkinsのシステム設定を変更して、HTTPSを有効化します。

Linuxに慣れてないので、ここで1日くらい時間を取られました。

いろいろな記事を渡りましたが、最終的に以下の公式ページが参考になりました。

[Managing systemd services](https://www.jenkins.io/doc/book/system-administration/systemd-services/)

##### system edit jenkins

設定変更は以下のコマンドから行います。

``` bash
# systemdのオーバライドでHTTPS設定を行う
sudo systemctl edit jenkins
```

実行すると、"nano"というエディタの編集画面が表示されます。

以下の設定をコピペします。
PASSWORDはkeystoreで入力したパスワードです。

``` nano
[Service]
Environment="JENKINS_PORT=-1"
Environment="JENKINS_HTTPS_PORT=8443"
Environment="JENKINS_HTTPS_KEYSTORE=/var/lib/jenkins/keystore"
Environment="JENKINS_HTTPS_KEYSTORE_PASSWORD=XXXXXX"
```

以下の画像のように、赤線の間にコピペします。
英文で書かれているように、間以外に記入された内容は保存されません。

![keytool実行](/assets/images/image-2023-06-04-jenkins-https-edit-jenkins.png)

コピペしたら、以下のキー入力で保存して閉じます。

^O(Ctrl + o) ⇒ Enter ⇒ ^X(Ctrl + x)

設定内容は以下のファイルに保存されます。

``` bash
/etc/systemd/system/jenkins.service.d/override.conf
```

#### Jenkisnを再起動

再起動して設定を反映させます。

``` bash
sudo systemctl restart jenkins
```

補足ですが、override.confを直接編集したり、あらかじめ用意したファイルを置くことも可能です。
その場合はdaemon-reloadで明示的に読み込む必要があります。

``` bash
sudo systemctl daemon-reload
sudo systemctl restart jenkins
```

これで、httpsの設定は以上です。

### アクセス

ブラウザで以下のURLにアクセスします。

<https://XXX.XXX.XXX.XXX:8443>

XXX.XXX.XXX.XXXはJenkinsをインストールしたEC2イスタンスのPublic IpまたはDNSです。

自己証明書が信頼されていないので以下のような画面になりますが、"～に進む(安全ではありません)"をクリックしてください。

今後、AWSのロードバランサを使ってhttpsでアクセスさせてみようと思ってます。

![自己証明書を使ったhttps](/assets/images/image-2023-06-04-jenkins-https-access.png)

### Unlock Jenkins

初めてJenkinsにアクセスすると、以下ののページが表示されます。

記載に従って、Jenkinsのロックを解除します。

EC2のLinuxにログインして、以下のコマンドを入力すると、初期パスワードが取得できます。

``` bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

![jenkins start page](/assets/images/image-2023-06-04-jenkins-startpage.png)

### プラグインの設定

以下のように二つのプラグイン設定する画面が表示されます。

Jenkinsに詳しくないので、左のおすすめ設定を選びます。

プラグインのインストールに移り、インストールが始まります。

![プラグインの設定](/assets/images/image-2023-05-27-jenkins-plugin.png)

### ユーザ設定

ユーザ情報(名前、パスワード、メールアドレス)を入力します。

表示に従って入力するだけです。

### Instance Configuration

JenkinsのRoot URLを設定します。

httpsになっていると思いますので、そのままでOKです。

### Jenkins is ready

設定完了です。

Startボタンをクリックします。

## Githubで使うSSHの設定

githubからcloneするためにSSHの設定をします。

### Git Host Key Verification Configuration

GitHubの公開鍵をJenkinsに登録します。

SSHの接続で接続先がGitHubの本当のサーバであるかを確認するためであり、設定しないと接続できません。

まずはダッシュボードのJenkinsの管理からセキュリティ画面に移動します。

![Jenkins セキュリティ](/assets/images/image-2023-05-28-jenkins-security.png)

セキュリティ画面のスクロースしていくと、"Git Host Key Verification Configuration"があります。

今回はAccept first connectionを選択します。

#### Accept first connection

自動でGithubが公開する公開鍵を設定してくれますので、他に作業はありません。

ただし、これは初回アクセス時だけです。

Githubの公開鍵が変わるとKnown hosts fileを削除する必要があります。または他の設定項目による設定が必要です。
これはJenkinsではなく、OpenSSHの動作によるものです。

また、Amazon Linux2でAccept first connectionを設定すると後述のジョブ作成画面で以下のようなエラーが表示されます。

![Jenkins セキュリティ](/assets/images/image-2023-05-28-jenkins-git-error-no-option.png)

Amazon Linux2にインストールされているOpenSSHのバージョンでは初回アクセスを許可するオプションがないため、エラーになるようです。

[質問サイトでのやり取り](https://serverfault.com/questions/1105812/command-line-line-0-unsupported-option-accept-new-how--fix-ths)

OpenSSH 7.6以降なら使用できます。Amazon Linux2023なら問題ありません。

``` bash
# Amazon Linux2のSSHバージョン確認
[ec2-user@ip-10-0-0-186 jenkins]$ sshd -v
unknown option -- v
OpenSSH_7.4p1, OpenSSL 1.0.2k-fips  26 Jan 2017
```

SSHのバージョンアップで解決できますが、失敗するとSSHでログインできなくなるので、難易度は高いと思います。

#### 他のHost Key Verification Strategyについて

##### Known hosts file

デフォルトはこれになっています。

jenkinsユーザでbashを起動し、以下の方法で.ssh/known_hostsにGitHubの公開鍵を設定する必要があります。

* 手動で"/var/lib/jenkins/.ssh/known_hosts"を作成
* sshコマンドで初めてGitHubにアクセスすると、known_hostsファイルが作成される。

以下はGithubにアクセスして、known_hostsファイルを作成するコマンドを記載しています。

``` bash
# rootユーザとなる。ec2ユーザからjenkinsユーザでbashを起動できないため。
sudo su -
# jenkinsユーザとなり、bashを起動
su jenkins --shell=/bin/bash
# sshでGitHubサーバにアクセス。
# 初回アクセス時はGitHubサーバから受け取った公開鍵を受け入れるか聞かれるので、
# yesを選択するとknown_hostsファイルに登録される。
ssh -T git@github.com
# 作成されたかを確認
cat /var/lib/jenkins/.ssh/known_hosts
```

###### Manually provided keys

Jenkinsの画面上から手動で設定します。

Approved Host Keysには[GitHub公開鍵](https://docs.github.com/ja/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints)をコピペします。

![Jenkins ホストキー設定](/assets/images/image-2023-05-28-jenkins-hostkey.png)

![コピペする箇所](/assets/images/image-2023-05-28-jenkins-github-publickey.png)

### Credentials 認証情報の登録

SSHで使用する秘密鍵を登録します。

Jenkinsの管理からCredentials画面を表示します。

![Jenkins Credentials](/assets/images/image-2023-05-28-jenkins-credentials.png)

DomainsのGlobalを選択します。

![Credentials Global](/assets/images/image-2023-05-28-jenkins-credentials-Global.png)

Add Credentialsを選択します。

![Credentials Global](/assets/images/image-2023-05-28-jenkins-credentials-add.png)

種類"SSH ユーザ名と秘密鍵"を選択し、IDに好きな値を入力します。
他は空でOKです。ユーザ名は作成が完了すると"jenkins"が自動で設定されます。

![Credentials Global](/assets/images/image-2023-05-28-jenkins-credential-settting1.png)

秘密鍵は直接入力のみで、開発PCで使用している秘密鍵を入力します。
パスフレーズは空でOKです。

秘密鍵の詳細は以下の記事を参照してください。開発PCでGitHubにPushできているなら、秘密鍵が"~/ssh"に作成済みだと思います。
"~"はユーザフォルダです。Windowsだと"C:\Users\XXX"です。隠しフォルダなので、表示する設定が必要です。

[Jenkins SSHの認証情報の設定](https://qiita.com/nkjzm/items/865f71efbf494ed57006)

![Credentials Global](/assets/images/image-2023-05-28-jenkins-credential-settting2.png)

#### Jenkins用に新しく秘密鍵を作成する場合

鍵を分けたい場合は、以下の記事が参考になります。

[Jenkins SSHの認証情報の設定](https://qiita.com/nkjzm/items/865f71efbf494ed57006)

## Jenkins ジョブ作成

ここからビルドに向けての作業になります。

### 新規ジョブ作成

ダッシュボードから新規ジョブ作成を選択してください。

### フリースタイル・プロジェクトのビルド

ジョブ名を入力して、フリースタイル・プロジェクトのビルドを選択してOKを選択してください。

![新規ジョブ](/assets/images/image-2023-06-04-jenkins-newjob.png)

### Configure

#### General

設定が必須な項目はないです。

#### ソースコード管理

Gitを選択して以下のように設定してください。

![Git設定](/assets/images/image-2023-06-04-jenkins-git.png)

* リポジトリURL
  * ビルドしたいリポジトリを入力してください。
  * 画像は私が作成したflutterの初期プロジェクトをコミットしたリポジトリです。
* 認証情報
  * Credentialsで作成した認証情報を選択してください。
  * jenkinsという名前になっていますが、作成したCredentialのユーザ名が空だとこの名前になるようです。
* ブランチ指定
  * ビルドしたいブランチを入力してください。
  * デフォルトは"*/master"になっていますが、最近githubはmainがデフォルトの名前になっています。私が作成したリポジトリはmainだけなので、mainを入力しています。

#### ビルド・トリガ

設定必須な項目はありません。

Jenkins上でビルド開始ボタンを押してビルドします。後で説明します。

自動化においては重要なところで、また使ってみたいと思います。

#### ビルド環境

設定必須な項目はありません。

#### Build Steps

シェルの実行を選択します。

スクリプトを入力できるので、お好きなスクリプトを入力してください。
スクリプトがエラーなく実行されると成功になります。
何かのビルドが必須というわけではありません。

例えば、以下のスクリプトを入力してビルドを実行すると、gitからブランチをcloneしてhellow worldを出力して成功となります。
gitからコードを取ってくる処理はスクリプトに書く必要はありません。

``` bash
#!/bin/bash
echo hellow world
```

flutterアプリをビルドしてApacheのルートディレクトリに出力するスクリプトは以下のようになります。

``` bash
#!/bin/bash

echo "add path to flutter"
export PATH="$PATH:/usr/bin/flutter/bin"

echo "delete html directory on apache"
rm -rf /var/www/html

echo "flutter version"
flutter --version

echo "build flutter web app and output apache hosted directory"
flutter build web -o /var/www/html
```

#### ビルド後の処理

設定必須な項目はありません。

#### ジョブ作成完了

完成なので保存します。ジョブの作成は以上です。

### ビルド

#### 実行

ビルド実行をクリックすると、ビルドが開始されます。

![ビルド開始ボタン](/assets/images/image-2023-06-04-jenkins-build.png)

#### 成功

成功すると2回目のように緑のマークがつきます。

1回目はスクリプトの間違いに気づいたので中断しました。

失敗したら赤のマークがつきます。試してみてください。

![ビルド成功](/assets/images/image-2023-06-04-jenkins-build-success.png)

#### ビルド出力の確認

各ビルドをクリックすると、スクリプトのコンソール出力を確認できます。

![ビルド出力](/assets/images/image-2023-06-04-jenkins-build-output.png)

#### ワークスペース

ビルドするときの作業ディレクトリです。

以下のようにgitから取ってきたファイルが置かれます。

![ビルド出力](/assets/images/image-2023-06-04-jenkins-build-workspace.png)

Linuxの中の場所は以下になります。

``` bash
$ ls /var/lib/jenkins/workspace
'flutter web'  'flutter web@tmp'
```

#### ビルド完了

ビルドとデプロイが完了しました。

### Webアプリにアクセス

ロードバランサのDNS名からブラウザでアクセスして、flutterのアプリが表示されたら完了です。

![完成したWebアプリ](/assets/images/image-2023-05-27-flutter-couter-app.png)

## メモ

### Jenkinsのパスワードを忘れた場合

いったん認証を無効化します。

``` bash
sudo sed -i 's/<useSecurity>true/<useSecurity>false/' /var/lib/jenkins/config.xml
systemctl restart jenkins
```
