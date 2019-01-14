---
layout: post
title: BERT with SentencePiece を日本語 Wikipedia で学習してモデルを公開しました
categorjies: ['Machine Learning']
---


### TL;DR
- 日本語 Wikipedia で学習した BERT モデルを公開しました **[yoheikikuta/bert-japanese](https://github.com/yoheikikuta/bert-japanese)**
- livedoor ニュースコーパスで finetuning して良い性能を発揮することも確認
- まあまあコスト掛かっているので、役に立った場合は **[BOOTH の商品ページ](https://yohei-kikuta.booth.pm/items/1174843)** でサポートしてくれると嬉しい
<br>

<script type="text/javascript" src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>

BERT の登場以降、自然言語処理の応用タスクを気軽に解く機運が高まってきたように思う。
自分はもともと画像分析の方に重きを置いていたが、最近は割と自然言語処理の応用タスクに興味があって色々やっていた。
BERT が決定版とも思わないし今後もどんどん改良はされていくとは思うが、ともかく機械学習モデルを利用する側にとってはかなり気軽に自然言語処理の応用タスクを解けるようになってきている。良いことだ。

これは誰か日本語の tokenizer を使った pretrained モデルを公開してくれるだろうと期待していたが、自分の知る限り公開されてない。
ということで自分がやって pretrained モデル含めて公開した。
repository は以下。
- **[yoheikikuta/bert-japanese](https://github.com/yoheikikuta/bert-japanese)**


## 内容
repository の README に書いてあるので特に言うことがない...  
google drive で学習済みの SentencePiece モデルと BERT モデルを提供しています。
- **[google drive](https://drive.google.com/drive/folders/1Zsm9DD40lrUVu6iAnIuTH2ODIkh-WM-O?usp=sharing)**

他人に試してもらっていないので現状では色々バグがあるかもしれません。
ダウンロードできないとかスクリプトが動かないとか何か発見したら issue か Twitter 辺りで言ってもらえると助かります。

コードもモデルも全部公開しているので、好きに使ってデータ追加して学習するなり改変するなりしてください。
それで何か面白いことがわかったら教えてください。


## BOOTH で投げ銭用のページを準備しました
この repository を完成させるのにまあまあ頑張りました。
自分がやりたくて勝手にやっただけの話ではありますが、「役に立ったのでお金払ってもいい」という篤志家の方々は以下のリンクから BOOTH の商品をご購入いただけるとすこぶる嬉しいです。
購入しても特に情報のない pdf がダウンロードできるのみです。
- **[BOOTH の商品ページ](https://yohei-kikuta.booth.pm/items/1174843)**

下限は 500 円になっていますが、BOOTH は上乗せができるのでもっと払いたくて仕方がないあなたも安心！


## Special Thanks
完成させるのに [@karino2](https://twitter.com/karino2012)氏 と [@himkt](https://twitter.com/himkt)氏 にも助力いただきました。
ありがとうございました。


## まとめ
BERT with SentencePiece を日本語 Wikipedia コーパスで学習したモデルを公開した。
みんなどんどん使って日本語自然言語処理のタスクを解いていこう。

---
---
<br>

