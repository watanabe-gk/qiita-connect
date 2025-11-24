---
title: <button>タグがsubmitされるときの解決
tags:
  - HTML
  - JavaScript
  - jQuery
private: false
updated_at: '2024-11-14T15:25:30+09:00'
id: c670b4ea3e3afba2c1d2
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

モーダル制御などで、閉じるボタンに
```
<button></button>
```
を設定していて、JavaScriptで制御しようとしたらsubmitされてしまうため、思うように動かず。
# 解決策
```
<button type="button"></button>
```
formタグ内のbuttonタグは、type属性を設定しないと`type="submit"`になるらしい。
思わぬ落とし穴。。
# 参考文献

https://developer.mozilla.org/ja/docs/Web/HTML/Element/button
