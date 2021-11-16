---
layout: post
title: BigQuery を使って分析する際の tips (part1)
categories: ['Machine Learning']
---


### TL;DR
- BigQuery で分析する際の tips をまとめてみる。長くなりそうなのでいくつかに分割して書く
- part1 はエディタとして何を使うかとか実行結果の連携などについて書く
- BigQuery console/DataGrip を使いつつ、結果を GitHub issues/Google Sheets/Bdash Server で共有するという感じで使っている
<br>

<script type="text/javascript" src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>

仕事で BigQuery を使って分析することが多いので、いくつかの回に分けて BigQuery を使って分析する際の tips をまとめていくことにする。今回は part1 としてエディタとして何を使うかとか実行結果の連携などについて書く。
個人的な探索的・アドホック分析用途の話に限定して、組織的にどういうデータ分析基盤を使うかとかそういう話はしない（会社だと ETL の L として dbt [https://www.getdbt.com/](https://www.getdbt.com/) を使っていて、これについても色々と書きたいことはあるけどそれはまた別の機会に）。


## tips1: クエリを書くためのエディタ
基本的には BigQuery console [https://console.cloud.google.com/bigquery](https://console.cloud.google.com/bigquery) を使ってきたし、今でも使っている。

しかしながら、これは BigQuery console を使っている人全員が感じていることだと思うが、イマイチと言わざるを得ない。
ブラウザで提供してるので補完が遅いのはやむなしと思う（この遅さによって書いてる時にちょくちょくスムースにいかなくてストレスが溜まることもあるが）けど、フォーマッターはとりあえずこれで納得するかというレベルにならない（CASE 式を見るたびに足が震えてしまう）し、ちょっと前に新 UI が出た時も「これは本当に開発者が触って使い心地チェックしてるのだろうか？」という感じだった（同僚の中には使いづらいのでエディタタブを無効にしているという人もいる）。
BigQuery は間違いなく素晴らしいプロダクトではあるが、その「プロダクト」には UI の観点があまり含まれていないようで、こういうところは残念である。

もちろんいいところもたくさんあって、ブラウザがあれば使えるというお手軽さは最高だし、実行前の検査でクエリスキャン量を表示したりや文法エラーに留まらずより広範囲のエラーを教えてくれたり（例えば異なる region にある dataset のテーブルを join は不可能だが、それをクエリ実行前に教えてくれる）、Google Sheets や GCS への export などがシームレスに実行できる点、など重宝している。
個人的にはフォーマッターがマシになってくれてかつそのフォーマッターが公開されてくれれば、BigQuery console 一本でやっていくという選択肢を取ってもいいかなと思っているが、待ち続けてもそういう日がやって来る気配はない。

そんなこんなで JetBrains 社の DataGrip [https://www.jetbrains.com/datagrip/](https://www.jetbrains.com/datagrip/) も併用している。
これはデータベース IDE で JetBrains 社の製品！という感じの rich な機能を提供してくれる。

書き味はざっくりこんな感じ。
FROM 句書く時に謎の空白行を入れてるけど、これはテーブル名の変換候補の表示で会社で使っている project/dataset/table が見えないようにするためです。
<center>
<a href="https://imgur.com/cTFW5yq"><img src="https://i.imgur.com/cTFW5yq.gif" title="source: imgur.com" /></a>
</center>

JetBranins 社製品だけあって、補完の滑らかさはかなり心地良い。
その他にも文を複数書いてもよしなに各文を解釈してくれるので `CMD + Enter` とカーソル選択のみで特定の文を選択的に実行できるとか、`F1` で Quick Documentation してテーブルの概要を把握できるとか、`Export Data` でクエリ結果を markdown table にして `Copy to Clickboard` して GitHub issues に貼れるとか、便利な機能が多い。
フォーマッター `Editor > Code Style > SQL > General` で実際にどのようにフォーマットされるのかの例が以下のように視覚的に分かりやすい形で柔軟に色々と設定することができる（フォーマッターとかあれこれ細かく設定したくないのでこれ使っとけばだいたいオッケーというものをチームで使うようにしたいのだが...）。
<center>
<a href="https://imgur.com/W9juwKo"><img src="https://i.imgur.com/W9juwKo.png" title="source: imgur.com" /></a>
</center>

DataGrip は色々なデータソースに対応していて、BigQuery 対応は比較的最近（2020.2）始まったので、まだまだ不完全な部分もある。
やはり Array型 とか STRUCT型 周りが弱い感じがする。
補完が弱いのはまあそんなに気にならないが、手元で色々分析する時に例えば id だけ入れ替えれば使いまわせるような scripting statements [https://cloud.google.com/bigquery/docs/reference/standard-sql/scripting](https://cloud.google.com/bigquery/docs/reference/standard-sql/scripting) を使ったりするので、これがサポートされてないのはちょっと不便だ。
それと JetBrains 社製品ということでそこそこの金額がするのも気になるところ。
会社で働いていてがっつり使うという場合は問題ないが、そんなに頻度高くなく必要になったらちょっと使いたいくらいの場合はなかなか購入まで踏み切りづらいかもしれない。

local でのクエリの保存とちょっとしたグラフを見るという用途で Bdash [https://github.com/bdash-app/bdash](https://github.com/bdash-app/bdash) も使っている。
local での保存という意味では別に Bdash を使わなくてもいいのだが、後述するように Bdash Server [https://github.com/bdash-app/bdash-server](https://github.com/bdash-app/bdash-server) で他の人にクエリを共有するというのにも便利なので Bdash を使っている。
作者が社内にいるのでなんかサポートされてない機能があったら feature request を出せるのも便利（contribute しろよという意見は一旦無視する）。
<center>
<a href="https://imgur.com/KZU9uoR"><img src="https://i.imgur.com/KZU9uoR.png" title="source: imgur.com" /></a>
</center>


## tips2: Colaboratory での利用
クエリだけでなく、Python で例えば `pandas.DataFrame` に結果を読み込んで interactive に色々分析したいということもある。
ちょっとした分析が目的のときには local に環境準備をするのは面倒だし、他の人に再現可能な形で共有するのにも適してない。
そういうときにはやはり Colaboratory [https://colab.research.google.com/notebooks/welcome.ipynb](https://colab.research.google.com/notebooks/welcome.ipynb) を使うのは便利である。

BigQuery にクエリを投げて結果を利用する方法はいくつかあるが、一番お手軽なのは magic commands を使うことだろう。
自分は以下のコードを snippets に登録して使えるようにしている。

```python
from google.colab import auth
auth.authenticate_user()

%%bigquery --project [適当な GCP project] df

select * from ``
```

これで interactive に authentification をして、クエリ実行結果を `df` という `pandas.DataFrame` に格納することができる。
Colaboratory は URL を共有してコピーして使ってもらえば他の人も（自分と同じような環境構築をしてもらうとかの手間がなく）簡単に利用できる。
ちなみに自分がいま分析してる範囲だとそんなに Python を使ってあれこれ分析せずに SQL だけで十分なことも多いので、そんなに使ってはいない。

ちょっと話は逸れるが、分析やモデリング周りを一手に担う統合プラットフォームとして Vertex AI [https://cloud.google.com/vertex-ai](https://cloud.google.com/vertex-ai) が実際どんな感じかはやや気になっている。
ここまで色々揃っていると、このプラットフォームに沿うように自分たちがうまく振る舞わないといけないと思うが、そうしたときにどれくらいのメリットを享受できるのかというのはガッツリ運用している人がいればぜひ聞いてみたいところだ。


## tips3: 実行結果の連携方法
分析結果をどう連携するかについても日々実践していることを書いてみる。

一番頻度高く使うのは、クエリの実行結果を作業ログとしての GitHub issues に markdown table として貼るというもの。
これは連携というよりは自分の日々の作業ログであったり他の人が作業ログを見た時に自分の思考の流れが追えるようにしてるという側面が強い（ちなみに以前データ分析系タスクの作業ログ共有に Jasper を使っているという話 [https://yoheikikuta.github.io/working_log_using_jasper/](https://yoheikikuta.github.io/working_log_using_jasper/) も書いたりしている）。
markdown table としてコピーする方法は BigQuery なら以下のように tweet した方法を使っていて、DataGrip の場合は `Export Data` で `Copy to Clickboard` を使っている（当然後者の方が使いやすい）。
<center>
<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">BQ console のクエリ結果をコピーして GitHub issues とかにマークダウンとしてペーストできるのは便利だけど、左だとヘッダーがちゃんととれなくて右だと大丈夫（これは Safari で Chrome だと見た目の違いは分からない）<br><br>tips は左上からではなく右下の値のところから範囲選択することです（真顔） <a href="https://t.co/Th3h99C5Qt">pic.twitter.com/Th3h99C5Qt</a></p>&mdash; Yohei KIKUTA (@yohei_kikuta) <a href="https://twitter.com/yohei_kikuta/status/1457273523706077188?ref_src=twsrc%5Etfw">November 7, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</center>

次によく使うのは Google Sheets に結果を export して連携するというもの。
Google Sheets はソフトウェアエンジニア以外でも使える人が多いので、結果を共有したり interactive にちょっと触ってもらうのには適している。
BigQuery console だと `結果の保存 > Google スプレッドシート` でスッと export できるし、DataGrip ならば `Export Data` で TSV にして `Copy to Clickboard` でコピーしてから Google Sheets にペーストしている。
ちょっとダルいのは、特に前者の方法だと個人の Google Drive に保存されてそのままでは他の人が使えないので、共有できるよう保存場所を移動するなりしないといけないという点。

Google Sheets という意味では Connected Sheets [https://cloud.google.com/bigquery/docs/connected-sheets](https://cloud.google.com/bigquery/docs/connected-sheets) を使うのも便利。
単純にクエリの結果を貼って共有するのとは違い、こちらはちょっとしたクエリを書いてその実行結果を Google Sheets 上で扱えるというのが利点で、さらに定期実行を設定できるので新しいデータを参照してもらうことができる。
自分はちょっとした分析結果を使ってビジネス的な意思決定のインプットにしてもらう、とかいうときに日時で新しい結果を参照できるような Connected Sheets を共有したりする。
もちろんこの Connected Sheets 上で複雑なクエリを書いたり、あれもこれもとやりすぎると管理や運用が大変になってくるので、よく使うし重要というものはちゃんとデータマートを準備したり BI ツールで閲覧できるようにしたりなどと交通整理する必要はある。

Data Portal とか BI tool での連携というのはアドホックというよりもう少しかっちり運用という感じになるので、今回は対象外としておく。

連携というほどにはまだ会社全体で使い倒してるレベルにはないが、Bdash Server が導入されてるのでクエリと結果を共有（というか他の人も見れたら便利かもなというものを見れるとこに置いておくだけという感じ）するときに使っている。
Bdash からワンクリックで以下のように共有できるので、local での自分用のクエリ保存と簡単なグラフ確認を Bdash でして、それを他の人も共有できるように Bdash Server にも送っておくという使い方をしている。
<center>
<a href="https://imgur.com/fu4nsGJ"><img src="https://i.imgur.com/fu4nsGJ.png" title="source: imgur.com" width="600" /></a>
</center>

## まとめ
BigQuery で分析する際の part1 としてエディタとして何を使うかとか実行結果の連携などについて書いた。

---
---
<br>

