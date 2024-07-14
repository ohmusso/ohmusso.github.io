---
layout: post
title:  Powershell Excel テーブルコピー
date:   2024-03-20 21:00:00 +0900
categories: Powershell
tags: powershell excel
mermaid: true
---

* [Powershell Tips]({% link _posts/2024-03-20-powershell-tips.markdown %})

あるシートの表を別シートにコピーするスクリプトです。

* ポイント
  * コピーするときに数式のセルは値にしてコピーします。
  * 書式もコピーしますが、条件付き書式はコピーできないです。
  * 表の一番左端の列を表の終端チェックで使用しています。
  * クリップボードを使わずにコピペするスクリプトです。

以下の記事を参考にしました。記事のようにクリップボードを使わずにコピペするスクリプトです。

<https://excel-ubara.com/excelvba4/EXCEL254.html>

### スクリプト本体

<https://github.com/ohmusso/powexcel/blob/main/excel.psm1>

Copy-Table

### テストスクリプト

<https://github.com/ohmusso/powexcel/blob/main/test/test_excel_copy.ps1>

test_copy_from.xlsxに以下のような表が入力されています。

![テストテーブル](/assets/images/image-2024-03-20-powershell-excel-test-table.png)

これをtest_copy_to.xlsxにコピーします。

test_copy_to.xlsxはあらかじめ開いておくことを前提としたテストスクリプトにしています。

![テスト実行時](/assets/images/image-2024-03-20-powershell-excel-test.gif)
