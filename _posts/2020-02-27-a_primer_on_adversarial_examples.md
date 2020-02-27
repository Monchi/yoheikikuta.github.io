---
layout: post
title: 技術書典用に書いていた本「A Primer on Adversarial Examples」を公開した
categories: ['Machine Learning']
---


### TL;DR
- PDF ファイルは [こちら](https://drive.google.com/file/d/159RzggX_4BwR9u7XMGrdqPuwaMiD80ic/view)
- GitHub repository は [こちら](https://github.com/yoheikikuta/a-primer-on-adversarial-examples)
- 無料で公開してますが、役に立ったという人はぜひ [BOOTH の商品](https://yohei-kikuta.booth.pm/items/1867263) を買っていただけると！（内容は同じです）
<br>

技術書典 8 で販売しようと思っていた adversarial examples の本を公開します。

技術書典、初参加の予定だったので楽しみだったんですが、中止となってしまったのは残念です（致し方ないですが）。
運営の方々は大変な決定だったと思います、改めてこの場で労いと感謝の意を表明しておきます。

オンラインで開催されるという話ですが、元々は 2 月末に出そうと思って準備していて一通りは完成したので、ここで公開しておきます。
（オンラインで開催されるようならそちらにも出したいとは思っています）

本とコードをセットで書きました。
自分は有料じゃないと見れない情報とか好きじゃないので、無料で公開しておいて、お金払ってもいいという方には BOOTH で同商品を買っていただくという形にします。

- 本の PDF ファイル: [Google Drive へのリンク](https://drive.google.com/file/d/159RzggX_4BwR9u7XMGrdqPuwaMiD80ic/view)
- GitHub repository: [repository へのリンク](https://github.com/yoheikikuta/a-primer-on-adversarial-examples)
- BOOTH の商品ページ: [BOOTH 商品へのリンク](https://yohei-kikuta.booth.pm/items/1867263)


## 本の内容
これは読んでくださいという感じですが、adversarial examples 事始め的な本になっています。

色々書きたいことはあったんですが、画像処理において代表的な攻撃手法や防御手法をちゃんと理解してコードも動かしてみる、という基本的な部分にフォーカスしました。

結構簡単ですが表面的なレベルよりは踏み込んでそれぞれを解説しているので、内容に興味があって少し画像分析をやった人であればまあまあちゃんと理解できる内容になっているのではないかと思います。

これぐらい理解できれば具体的な手法に関してはそこそこ知っていると言っていいと思うので、その先の発展的な内容に入っていくには

自然言語処理やもう少し発展的な話題についても書きたかったし構想はしていたんですが、ページ数も結構膨らんでしまったので一旦これで完成としておきます。
やる気が再燃したら追記していくかもしれません。

実験コードはもう少し小さくしようと思ってましたが、まあまあ色々と追加してコマンドライン引数が煩わしいレベルまで膨らんでしまいました。
この辺は YAML とかで管理する形に直そうかと迷いましたが、色んなパターンで実験を回しまくるということをするつもりがなかったので止めました。
同じ理由でデータとモデルのクラス以外はあまり切り分けずにガッと `main.py` にまとめてます。

issue や PR はウェルカムです。
コードの方は [こちら](https://github.com/yoheikikuta/a-primer-on-adversarial-examples) で本の内容の方は [こちら](https://github.com/yoheikikuta/a-primer-on-adversarial-examples-tex) まで。
特に本の方は誤字脱字や間違いもあるかと思うので、気付いたものがあれば是非お願いします。


## まとめ
adversarial examples 事始め的な本を書いた。
これで無職（フリーランス）でやっとこうと思ったことはやったので、そろそろ就職先を決めるぞ！

---
---
<br>
