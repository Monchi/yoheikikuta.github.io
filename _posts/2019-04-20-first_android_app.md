---
layout: post
title: Kotlin で初の Android アプリを作った
categories: ['programming']
---


### TL;DR
- 初の自作 Android アプリである [FingerBoardCheck](https://github.com/yoheikikuta/FingerBoardCheck) を作った
- ギタータブ譜読み用のトレーニングアプリで、めちゃくちゃ簡単なものだけど、とにかく作った
- 色々学びながらどんどん自分の作りたいものを作っていこう
<br>

Kotlin で Android アプリを作ろう、ということで初めて自分で一から作ってみた。

細かいことはとりあえず置いておいて Kotlin で Android アプリを作ってみようと考え、とりあえず公式ページを眺めつつ前準備として以下の二つをこなした。

- [Kotlin Koans](https://kotlinlang.org/docs/tutorials/koans.html)  
  Kotlin の基本的な文法等を知ろうと思ってやった。Java のこれを云々、みたいなのが結構あってまあまあしんどかった（Java 知らないので）。
- [Build Your First Android App in Kotlin](https://codelabs.developers.google.com/codelabs/build-your-first-android-app-kotlin/index.html#0)  
  これは getting started の内容としてなかなか良かった。Android Studio の使い方を含めてごく簡単な処理を書いてみるというものなので、自分でアプリを作るための基礎中の基礎を学べた。

あと Udacity の [Kotlin Bootcamp for Programmers](https://www.udacity.com/course/kotlin-bootcamp-for-programmers--ud9011) もやったが、これはカバーしている内容が自分にマッチしなかったり変にエンターテイメント的だったりしてイマイチだった。

この段階でさてどうするかなと考えていたら、karino2氏にその実力を持ってして作ったらどうか、と言われたのがタブ譜読みアプリだったというわけだ。
いつもありがとうございます。
karino2氏は [Kotlin のプログラム教室](https://karino2.github.io/kotlin-lesson/) もやっているとのことなので、興味ある人はやってみるのもよいだろう。
これは [issue](https://github.com/karino2/kotlin-lesson/issues/) にいくつかのトピックが示されていてその中で取り組んでみたいものに取り組む、という形になっている。

### 作ったアプリに関して
内容に関しては単純なので [README](https://github.com/yoheikikuta/FingerBoardCheck) を見れば大体尽きている。

6弦12フレットのギターのタブ譜になっていて、`START` を押すたびにランダムに弦とフレットが指定されてそれらが何の音に該当するかを当てるというものだ。
自分がギターを弾かないので需要や使い勝手などは判断しかねる部分もあるが、触っているうちに少しは覚えてきた（正確には 0 フレットの音だけ自然に覚えてあとは順番に数えてるだけなのだが）。

このアプリは技術的にはとても簡単で、それこそアプリを作れる人なら数時間もあれば作れるものだろう。
自分はそもそも Android Studio でどうプロジェクトを構築してアプリを作っていくか、というところから知らない人間だったので基本的なところから学んでいった。
Kotlin っぽい書き方や Layout の基本などを教えてもらいながら、とりあえず使えるところまでは到達したので一旦終了。

このアプリを作るときに、何も分からんのでググって出てきた内容を使ったりしていたが、それがイマイチだったりして後で修正するということになった。
変に人のを流用するより公式レファレンスを読んでそれを基に作った方が結局は早い、ということなのでちゃんと意識してまずそこから当たっていくことにしよう。

### 今後何を作っていくか
他の誰かというより自分にとって役立つものを作っていこうと考えている。
とりあえず次に考えているのは arXiv の論文チェック用のアプリで、日々の新着のタイトル・アブストラクト・著者あたりをチェックして、ちゃんと読もうと思ったものだけをローカルなり Google drive なりに保存するようなもの。
以前 [arXiv の論文を drive に保存する Chrome extension](https://github.com/yoheikikuta/arxiv-pdf-downloader) を作ったことがあるので、それに近いものの Android アプリ版という感じだ。

機械学習を使ったアプリとかもそのうち作るかなぁ。
以前触ったときは結構しんどかったけど、MLKit とかももっとちゃんと使えるようになってたらいいな（まだ調べてない）。
まあその辺はもう少し経ったら調査しつつやっていくとしよう。

## まとめ
Kotlin で記念すべき初の Android アプリを作成した。
今後も色々学びつつ新しいアプリを作っていこう。

---
---
<br>
