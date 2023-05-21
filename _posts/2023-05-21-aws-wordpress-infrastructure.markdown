---
layout: post
title:  AWS WordPress構築
date:   2023-05-21 17:00:00 +0900
categories: AWS
tags: cloudformation wordpress
---

以下の公式テンプレートを参考に、WordPressをホストする環境をCloudFormationで作成しました。

[WordPress Template](https://cloudformation-templates-ap-northeast-1.s3.ap-northeast-1.amazonaws.com/WordPress_Single_Instance.template)

## 公式テンプレートとの違い

### 構成

以下のロードバランサを使ったハンズオンの構成を取ってます。
ここに、EC2にWordPressをインストールしています。

![wordpress composition](/assets/images/image-2023-05-21-wordpress-composition.png)

[Load Balancer Hands-On](https://catalog.us-east-1.prod.workshops.aws/workshops/47782ec0-8e8c-41e8-b873-9da91e822b36/ja-JP/hands-on/phase4)

### ネストしたスタック

スタックを分離して、CloudFormationのテンプレートを分割しています。

* VPC
* Subnet
  * Public Subnet Main
  * Private Subnet Main
  * Public Subnet Sub
  * Private Subnet Sub
* Security Group
  * ALB
  * Apache
  * MySQL
* MySQL(RDS)
* Apache(EC2)
* ApplicationLoadBalancer(ELB)

## 完成したテンプレート

[作成した WordPress Template](https://github.com/ohmusso/ohmusso.github.io/tree/main/assets/src/aws/cloud_formation/wordpress)

### 使用上の注意

* テンプレートは自身のS3に置く必要があります。またwordpress.yamlの各スタックへのurlを修正するる必要があります。
* パラメータについて
  * KeyNameは事前に作成してください。
  * パラメータで入力したデータベースのパスワードはコンソールに表示されます。AWS環境を他社と共有する場合は気を付けてください。
    ![wordpress composition](/assets/images/image-2023-05-21-wordpress-parameter.png)
* スタックを削除しても、以下のものは残りますので、手動で削除する必要があります。
  * S3に格納したテンプレート
  * MySQL(RDS)のスナップショット
* 完成した環境はHTTPSに対応していないです。練習用です。

## 今後

それぞれのスタックについて補足があれば書いていこうと思います。
