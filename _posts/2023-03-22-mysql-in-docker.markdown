---
layout: post
title:  Docker Get started での mysql
date:   2023-03-22 22:00:00 +0900
categories: how-to
tags: mysql docker
---

DockerのGet startedを進めていると、mysqlのコンテナを建ててmysqlのクエリを実行する箇所があります。

<https://docs.docker.jp/get-started/07_multi_container.html#id5>

``` mysql
select * from todo_items;
```

そのまま実行すると"ERROR 1046 (3D000): No database selected"が出力されます。

事前にデータベースを選択する必要がありました。sqlを触らない方は何も考えずに実行しちゃうはず!!!

クエリを実行する前に、以下のUSE文を使います。

``` mysql
use todos
```

これで上記のクエリが実行されます。

以下にコンソールの出力キャプチャを載せておきます。
"\c"はタイプミスがあった時にキャンセルするコマンドです。

![console log](/assets/images/ss-2023-03-22%20224108.png)
