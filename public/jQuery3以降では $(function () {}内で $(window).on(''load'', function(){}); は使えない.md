---
title: 'jQuery3以降では $(function () {}内で $(window).on(''load'', function(){}); は使えない'
tags:
  - PHP
  - JavaScript
  - jQuery
  - Laravel
  - Ajax
private: false
updated_at: '2024-12-05T15:33:39+09:00'
id: d38e9c10125697b0c98c
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに
jQueryの`$(window).on ('load')`が上手く動かない。
具体的には、リストデータを遅延読み込みする際に画面読み込み時だけ上手く動かず、条件検索や件数表示を変えたときの処理はきちんと動く。

### 改善前のソースコード
```
$(function () {
    $(window).on('load', function(){
    	// ここでAjax通信を行う。
    });
}
```
これがそもそもダメだった。
`$(function () {} `で`$(window).on('load', function(){});`を囲うことは **jQuery3以降では完全にアウト**とのこと。
# 解決策
```
$(window).on('load', function(){
    // ここでAjax通信を行う。
});
```
`$(function () {} `の外に出すだけ。

ただ、開発によっては思わぬ事故を防ぐために関数定義のたびに`$(function () {}`で囲うのをちょいちょい見ます。
少なくとも、`$(window).on('load', function(){});`に限っては`$(function () {} `の中で定義するのはやめておいた方が良さそう。

# 参考文献
https://wemo.tech/109
