---
layout: post
title:  Flutter Columnの中でListViewを使用
date:   2023-07-17 21:00:00 +0900
categories: How-To
tags: flutter
---

## Columnの中でListViewを使用するとエラーが出る

Flutterを勉強していると遭遇するエラーだと思います。

解決方法についてはいろいろな記事が出ていると思います。

<https://qiita.com/tabe_unity/items/4c0fa9b167f4d0a7d7c2>

私の記事では、shrinkWrapとFlexibleを使用する場合について補足したいと思います。

## shrinkWrapを使っているとエラーになるケース

実際のユースケースを考えます。リストで表示させる内容はAPIで取ってくる場合を考えます。

このとき、FutureBuilderなどを使用して

* 待っている間はスピナーを表示
* データが取れたらListViewを表示

とします。

ここでListViewにshrinkWrap==trueを設定しても、データ取得時にエラーが発生します。

``` dart
@override
Widget build(BuildContext context) {
return FutureBuilder(
    future: _getApi(), // 2秒後にデータを取ってくる
    builder: (context, snapshot) {
    if (snapshot.hasData) {
        return ListView(
        shrinkWrap: true,
        children: snapshot.data!.map((e) => Card(
            child: SizedBox(
            height: 150,
            child: Text('data: $e')),
        )).toList(),
        );
    } else {
        return const CircularProgressIndicator();
    }
    }
);
```

![構成図](/assets//images/image-2023-07-18-flutter-error.gif)

## 修正方法

この場合はFlexibleを使用すると、エラーが解決します。

``` dart
@override
Widget build(BuildContext context) {
return FutureBuilder(
    future: _getApi(), 
    builder: (context, snapshot) {
    if (snapshot.hasData) {
        return Flexible(
        child: ListView(
            // shrinkWrap: true,
            children: snapshot.data!.map((e) => Card(
            child: SizedBox(
                height: 150,
                child: Text('data: $e')),
            )).toList(),
        ),
        );
    } else {
        return const CircularProgressIndicator();
    }
    }
);
```

![構成図](/assets//images/image-2023-07-18-flutter-success.gif)

## 使い分けの判断

Widgetのサイズが最初から変わらない場合はshrinkWrapでも問題ないです。

Widgetのサイズが途中で変わる場合はFlexibleを使う必要があります。

![構成図](/assets//images/image-2023-07-18-flutter-widgets.png)

## 以上
