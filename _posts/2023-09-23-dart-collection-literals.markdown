---
layout: post
title:  Dart リストで重複する値を一つにまとめる 
date:   2023-09-23 12:00:00 +0900
categories: How-To
tags: dart flutter
---

## 重複する値があるリスト

``` dart
List<int> list = [1, 2, 2, 3];
```

## 重複する値をまとめる(期待値)

``` dart
list == [1, 2, 3];
```

## 方法 マップで定義してからリストに変換

``` dart
Set<int> map = {1, 2, 2, 3};　// この時点で{1, 2, 3}になる
List<int> list = map.toList(); // [1, 2, 3]
```

## ネットで見つけた方法

[ネットの記事](https://note.com/hatchoutschool/n/nd265a2ffcae2)

``` dart
list.toSet().toList();
```

これも同じことをやってます。List⇒Map⇒List。

## linterから改善を推奨される場合

以下の例だとlinterから改善を勧められました。

``` dart
[1, 2, 2, 3].toSet().toList();
```

[info: prefer_collection_literals](https://dart.dev/tools/linter-rules/prefer_collection_literals)

修正方法は以下の通り。

``` dart
{1, 2, 2, 3}.toList();
```

実際のユースケースではリストをくっつけて重複する値をまとめたい場合があると思います。

``` dart
list = [ ...listA, ...listB].toSet().toList();
```

しかし、上記の改善をlinterから勧められるので、以下の書き方になります。このようなケースでは最初からマップで扱いなさいということでしょうか。

``` dart
list = {...mapA, ...mapB}.toList();
```

## まとめ

* Listのリテラルは[]に入れる
* Mapのリテラルは{}に入れる
* 重複する値をまとめるListはMapにできないかを検討する

## 参考

* [Dart公式によるコレクションの説明](https://dart.dev/language/collections)
* [外国の方によるコレクションの解説](https://medium.com/dartlang/exploring-collections-in-dart-f66b6a02d0b1)
