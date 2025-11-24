---
title: 【話題のブラウザ】Arcをエンジニアが実際に使ってみてやめた話
tags:
  - Mac
  - Windows
  - Chrome
  - ARC
  - ブラウザ
private: false
updated_at: '2025-02-07T10:21:03+09:00'
id: 2508d7b1da1c87e58157
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに
今回は話題のブラウザ**Arc**を実際に使ってみた感想を**エンジニア視点**で話そうと思います。
結論から言うと、2週間足らずでChromeに戻しました。
そこまでに至ったメリットデメリット・経緯を記します。

# メリット(使いやすかった点)
>[世界で話題のブラウザ「Arc」が便利すぎたので魅力を解説する](https://qiita.com/ruitomo/items/cc444c6e4393568ee5b2)

ほぼ上記記事の内容がすべてだと思います。簡単にまとめると
* 他ブラウザからの移植性が高く移行しやすい
* サイドバーやショートカットなどUIの工夫が素晴らしい

# デメリット(使いづらかった点)
* 日本語入力での検索が致命的
日本語で検索する際にいちいち半角スペースを入力しないと入力値の補完がでないのは本当に致命的だと思いました。

スペース前 入力値の補完が出てこない。(このままEnter押すとYoutubeに飛びます)
<img width="400" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3537420/bafc890d-cfde-7347-987f-cebba6aecd1b.png">
スペース後 やっと入力値の補完が出てくる。(この状態でEnter押すとやっと検索したい内容に飛びます)
<img width="400" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3537420/6d92ba17-aecc-3d2d-e12e-4f678ef1c014.png">

* シンプルに重い&開発者ツール利用時が特にストレス大きい
エンジニアだと開発者ツールを利用することも多いと思いますが、とにかく重い&UXが悪い印象でした。サイズ変更の際のレスポンシブが微妙&開発者ツールを開いている画面と開いていない画面の行き来もかなりかくつきました。

サイズ変更前
<img width="400" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3537420/2e05b194-8ca1-5b77-2c8f-52cfab180f76.png">
サイズ変更後
<img width="400" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3537420/117e262f-cf17-c3cb-acbe-b158d25d8a87.png">

* AWSのS3でアップロードする際にドラック&ドロップできない
少々コアな内容になりますが、AWSのS3でアップロードする際にドラック&ドロップできなかったです。
Chromeではできるのに、こういったバグ?のような使い勝手の悪い事が他にもあるのではないかと継続意欲を失ってしまいました。
<img width="400" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3537420/16931e77-69a4-bf4d-d421-9780ee547574.png">

# まとめ
UIはかなり好みだったのですが、エンジニアとして開発をする際には効率が悪くなってしまう点が多々あり結局2週間足らずでChromeに戻しました。
逆に、この辺りが改善されたら是非また使ってみようと思います。
