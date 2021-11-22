---
layout: post
title: BigQuery を使って分析する際の tips (part2)
categories: ['Machine Learning']
---


### TL;DR
- part2 はスカラー関数・集計関数・分析関数、サブクエリ、型変換について書く
- BigQuery は便利な機能が色々備わってるので、それらの基本的な振る舞いを頭に入れておくと便利
- 本文と全然関係ないけど自分のブログはコードブロックの表示とかイマイチなので改善せねばか...
<br>

<script type="text/javascript" src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>

part1 に続いて part2 として、分析する際によく使うことになる道具について理解しておくとよいことをいくつかピックアップしてまとめる。

ちなみに今回の tips シリーズではクエリのパフォーマンスは気にしない。
自分が現状仕事で書いてるものはほぼクエリのパフォーマンスを気にしなくてよいのと、そもそも BigQuery が強力なので細かいことを気にせずに書いてしまって殆どの場合問題ない、というのが理由。
実行前のデータスキャン量だけ見ておいて、数百 [GB] 以上のクエリをガンガン実行しそうとなったらコストを気にし始める、というくらいしておけば自分の環境では十分という状況である。


## tips4: スカラー関数・集計関数・分析関数
分析でよく使う関数といえば集計関数や分析関数だが、これらの振る舞いを理解しておくために合わせてスカラー関数も見ておく。

これらの関数の振る舞いとして、入力行と出力行の対応が以下のようになっているというイメージを念頭に入れておくとよい。

- スカラー関数 (入力行):(出力行) = 1:1
- 集計関数 (入力行):(出力行) = N:1
- 分析関数 (入力行):(出力行) = N:N


### スカラー関数
スカラー関数は入力行と出力行が 1:1 になるもので、例えば以下のクエリは `word` column の各行に対して最初の文字から 2 文字目までを取得するクエリになっている。

```sql
SELECT
  word,
  SUBSTR(word, 0, 2) AS substr_word
FROM
  `bigquery-public-data.samples.shakespeare`
ORDER BY
  word DESC
```

<center>
<table style="border-collapse:collapse;border-spacing:0" class="tg"><thead><tr><th style="background-color:#F7F7F7;border-color:inherit;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">行</span></th><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">word</span></th><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">substr_word</span></th></tr></thead><tbody><tr><td style="background-color:#FFF;border-color:inherit;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit;color:rgba(0, 0, 0, 0.99)">1</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">zwaggered</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">zw</span></td></tr><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit;color:rgba(0, 0, 0, 0.99)">2</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">zounds</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">zo</span></td></tr><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit;color:rgba(0, 0, 0, 0.99)">3</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">zone</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">zo</span></td></tr><tr><td style="border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal">...</td><td style="border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal">...</td><td style="border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal">...</td></tr></tbody></table>
</center>

これは特別何かを意識する必要はないものではあるが、リファレンスなどを呼んでてスカラー関数と出てきたときはこのような振る舞いをするものだと認識しておけばよい。

スカラー関数を殊更に意識するところはそんなにないが、スカラー関数にはたいてい `SAFE.` 接頭辞が利用可能で、これをつけるとエラーを発生させずに `NULL` を返すものに変えることができる。
上の例でいうと、`SUBSTR(word, 0, -2)` とすると `Third argument in SUBSTR() cannot be negative` というエラーが発生するが、`SAFE.SUBSTR(word, 0, -2)` とすれば各行で `NULL` が返る（この例ではあまり意味が感じられないが、引数が何かの計算よって得られるもので、それが負になりうるケースがあるという場合には使えるかもしれない）。

脱線になるが、この `SAFE.` という接頭辞は演算子には使えない。
分析上よく遭遇するエラーとして `division by zero` があるが、これを防ごうとして `1 SAFE./ 0` などとすることはできない。
そのため、演算子と等価でありエラーが出るときには `NULL` を返す `SAFE_xxx` スカラー関数が準備されており、除算の場合は `SAFE_DIVIDE` を使うことになる（なお、整数除算の場合は演算子がなくスカラー関数 `DIV` があるのみなので、こちらの場合は `SAFE.DIV` とすればよい）。

さらに脱線になるが、この `SAFE.` というのは関数の返り値を評価するときに使われるもので、引数の式を評価するときには使われないものであると頭に入れておくと、`SAFE.` が使えないスカラー関数がいるということも理解しやすい。
例えば `LOWER` スカラー関数は `STRING` を引数にして、`STRING` であればエラーは出ないので `SAFE.LOWER` はサポートされていない。
`Lower(1)` のようなものは引数の式を評価して型がマッチしないということで実行前にエラーが分かるためで、型の静的解析でエラーか分かるものは `SAFE.` とかいらなくて実行時に初めてエラーか分かるものには `SAFE.` が使えるみたいなイメージ。

ついつい `SAFE.` の話をしてしまったが、ともかくスカラー関数は (入力行):(出力行) = 1:1 という関係で素直に使えばよい。


### 集計関数
集計関数に関する公式リファレンスは [https://cloud.google.com/bigquery/docs/reference/standard-sql/aggregate_functions](https://cloud.google.com/bigquery/docs/reference/standard-sql/aggregate_functions) である。

集計関数は入力行と出力行が N:1 になるもので、例えば以下のクエリは `word_count` の `DISTINCT COUNT` を計算するクエリである。

```sql
SELECT
  COUNT(DISTINCT word_count)
FROM
  `bigquery-public-data.samples.shakespeare`
```

<center>
<table style="border-collapse:collapse;border-spacing:0" class="tg"><thead><tr><th style="background-color:#F7F7F7;border-color:inherit;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">行</span></th><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">distinct_count_word_count</span></th></tr></thead><tbody><tr><td style="background-color:#FFF;border-color:inherit;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit;color:rgba(0, 0, 0, 0.99)">1</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">483</span></td></tr></tbody></table>
</center>

複数行（ここでは全ての行）に対して集計した結果を1行で返しているというもので、特に難しいところはない。

`GROUP BY` で集計するグループを設定するというのも普段から自然にやっている人が多いだろう、例えば以下のような感じ（集計関数の引数に入れてる `LIMIT` は単に結果が長くなりすぎるので切っているだけ）。

```sql
SELECT
  word_count,
  COUNT(DISTINCT word) AS distinct_word_count,
  COUNT(DISTINCT corpus) AS distinct_corpus_count,
  STRING_AGG(word, "&" LIMIT 3) AS string_agg_word,
  ARRAY_AGG(word LIMIT 2) AS array_agg_word
FROM
  `bigquery-public-data.samples.shakespeare`
GROUP BY
  word_count
ORDER BY 
  word_count
```

<center>
<table style="border-collapse:collapse;border-spacing:0" class="tg"><thead><tr><th style="background-color:#F7F7F7;border-color:inherit;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">行</span></th><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">word_count</span></th><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">distinct_count_word</span></th><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">distinct_count_corpus</span></th><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">string_agg_word</span></th><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">array_agg_word</span></th></tr></thead><tbody><tr><td style="background-color:#FFF;border-color:inherit;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit;color:rgba(0, 0, 0, 0.99)">1</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">1</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">30198</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">42</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">LVII&amp;augurs&amp;dimm'd</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">LVII</span></td></tr><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"></td><td style="background-color:#F5F5F5;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"></td><td style="background-color:#F5F5F5;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"></td><td style="background-color:#F5F5F5;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"></td><td style="background-color:#F5F5F5;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">augurs</span></td></tr><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit;color:rgba(0, 0, 0, 0.99)">2</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">2</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">8946</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">42</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">cheque'd&amp;affords&amp;meet</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">cheque'd</span></td></tr><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"></td><td style="background-color:#F5F5F5;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"></td><td style="background-color:#F5F5F5;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"></td><td style="background-color:#F5F5F5;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"></td><td style="background-color:#F5F5F5;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">affords</span></td></tr><tr><td style="border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal">...</td><td style="border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal">...</td><td style="border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal">...</td><td style="border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal">...</td><td style="border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal">...</td><td style="border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal">...</td></tr></tbody></table>
</center>

集計関数単体だと tips というほどの情報もないのだが、この (入力行):(出力行) = N:1 という関係を意識しつつ分析関数と併せて理解しておくと見通しがよくなることが多い。


### 分析関数
分析関数に関する公式リファレンスは [https://cloud.google.com/bigquery/docs/reference/standard-sql/analytic-function-concepts](https://cloud.google.com/bigquery/docs/reference/standard-sql/analytic-function-concepts) である。

分析関数は入力行と出力行が N:N になるもので、例えば以下のクエリは `word_count` が同じものを一つのグループにして、その中で `word` の長さが長い順に `rank` をつけるというクエリである。

```sql
SELECT
  word,
  word_count,
  RANK() OVER (PARTITION BY word_count ORDER BY LENGTH(word) DESC) AS rank_word_count_length
FROM
  `bigquery-public-data.samples.shakespeare`
ORDER BY
  word DESC
```

<center>
<table style="border-collapse:collapse;border-spacing:0" class="tg"><thead><tr><th style="background-color:#F7F7F7;border-color:inherit;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">行</span></th><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">word</span></th><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">word_count</span></th><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">rank_word_count_length</span></th></tr></thead><tbody><tr><td style="background-color:#FFF;border-color:inherit;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit;color:rgba(0, 0, 0, 0.99)">1</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">zwaggered</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">1</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">8448</span></td></tr><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit;color:rgba(0, 0, 0, 0.99)">2</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">zounds</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">1</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">46667</span></td></tr><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit;color:rgba(0, 0, 0, 0.99)">3</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">zone</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">1</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">80132</span></td></tr><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal">...</td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal">...</td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal">...</td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">...</span></td></tr></tbody></table>
</center>


最も重要なのは `OVER` 句で、ここで行のグループがどの単位なるのかを指定している。
今回は `PARTITION BY word_count` で `word_count` が同じもの毎にグループを作っている。いわゆる `WINDOW` というやつですね。
この `WINDOW` はこのように指定することもできるが、以下のように named window を使って書くこともできるが、あまり使ってるのを見たことがないし自分も使わない（可読性がそんなによくないので）。
コードブロックのコードシンタックスも効いてない。

```sql
SELECT
  word,
  word_count,
  RANK() OVER (word_count_window) AS rank_word_count_length
FROM
  `bigquery-public-data.samples.shakespeare`
WINDOW
  word_count_window AS (PARTITION BY word_count ORDER BY LENGTH(word) DESC)
ORDER BY
  word DESC
```

もう一つポイントになるのは結果を返すのはグループにした各行それぞれということ。
今回の例でいうと、例えば `word_count` が同じものが 100 行あったとしたら、各 100 行に対して結果を返している。
ランクをつけるってそういうことなんだから当たり前だろ、というのはごもっともだが、これが分析関数と呼ばれるものだとちゃんと意識しておくのは役に立つ。

例えば、各 `word` 毎に、その `word` の長さを全ての単語の中で最も長い単語で割った時の割合も一緒に結果に出すようなクエリを書きたいとしよう。
これを集計関数・分析関数の区別なくとりあえずこんな感じか！？と書いてエラーに遭遇するという経験をした人がいるかもしれない。

```sql
-- これはエラーになる: SELECT list expression references column word which is neither grouped nor aggregated at xxx
SELECT
  word,
  word_count,
  LENGTH(word) / MAX(LENGTH(word)) AS ratio_word_length
FROM
  `bigquery-public-data.samples.shakespeare`
ORDER BY
  ratio_word_length DESC, word
```

この `MAX` は集計関数で、入力行と出力行が N:1 になるものなので、各行に結果を返すわけではないので `word` や `word_cout` が返す行数がマッチしてないのでエラーになる。
`SELECT list expression references column word which is neither grouped nor aggregated at xxx` というエラーで、集計されてないので集計して行数がマッチするようにしなさいというものである。
このエラーだけ見て集計するようなクエリを書かないといけないか〜と考えるのではなく、これは集計関数の N : 1 の関係なので問題が起こるのであって、N : N の分析関数に変換できればいいんだなと考えれば、余分なクエリを書かずに対処ができる。
多くの集計関数は `OVER` 句を付与することで分析関数になるということを押さえておくと、以下のように書くことで目的が達成できる。

```sql
SELECT
  word,
  word_count,
  LENGTH(word) / MAX(LENGTH(word)) OVER () AS ratio_word_length
FROM
  `bigquery-public-data.samples.shakespeare`
ORDER BY
  ratio_word_length DESC, word
```

<center>
<table style="border-collapse:collapse;border-spacing:0" class="tg"><thead><tr><th style="background-color:#F7F7F7;border-color:inherit;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">行</span></th><th style="background-color:#F7F7F7;border-color:inherit;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">word</span></th><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">word_count</span></th><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">ratio_word_length</span></th></tr></thead><tbody><tr><td style="background-color:#FFF;border-color:inherit;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit;color:rgba(0, 0, 0, 0.99)">1</span></td><td style="background-color:#FFF;border-color:inherit;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">honorificabilitudinitatibus</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">1</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">1.0</span></td></tr><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit;color:rgba(0, 0, 0, 0.99)">2</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">Anthropophaginian</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">1</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">0.6296296296296297</span></td></tr><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit;color:rgba(0, 0, 0, 0.99)">3</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">indistinguishable</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">1</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">0.6296296296296297</span></td></tr><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal">...</td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">...</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">...</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">...</span></td></tr></tbody></table>
</center>

分析関数は分析クエリを書くときに多用するので、この辺りの振る舞いを押さえておくと、冗長で読みにくいクエリでなくてすっきりしたクエリを書けることが多い。

ちなみに `WINDOW` の柔軟な指定方法に関してはここでは触れなかったが、自分はあの文法は全然覚えられないので、必要が生じたらリファレンスを読みながら書くようにしている。


## tips5: サブクエリ
一口にサブクエリと言っても色々あるが、ここではテーブルサブクエリと式サブクエリについて書く。

テーブルサブクエリはテーブルを指定する位置で `()` で括ったクエリを書くことで利用できる。
これを使って先ほどと同じケースを書いてみると以下のようになる（分析関数を知っているとこれは全然よい書き方ではないと分かる）。

```sql
SELECT
  word,
  word_count,
  LENGTH(word) / max_length_word AS ratio_word_length
FROM
  `bigquery-public-data.samples.shakespeare`,
  (SELECT MAX(LENGTH(word)) AS max_length_word FROM `bigquery-public-data.samples.shakespeare`)
ORDER BY
  ratio_word_length DESC, word
```

テーブルサブクエリは基本的には `WITH` 句で書く方が可読性が高いので `WITH` 句を使うのがいいが、例えば今回の例を `WITH` 句を使うと以下のように冗長な感じにもなるので、ちょっとしたクエリを書いて `JOIN` したりするときはテーブルサブクエリを使ったりすることがある（繰り返しだが、今回の例だと分析関数を使っておくのが最も良い選択肢とは思うということは前提として）。

```sql
WITH
  table_max_length_word AS (
  SELECT
    MAX(LENGTH(word)) AS max_length_word
  FROM
    `bigquery-public-data.samples.shakespeare`)

SELECT
  word,
  word_count,
  LENGTH(word) / max_length_word AS ratio_word_length
FROM
  `bigquery-public-data.samples.shakespeare`,
  table_max_length_word
ORDER BY
  ratio_word_length DESC, word
```

式サブクエリは式が使えるところで利用できるクエリで、こちらはいくつかの典型的なものがある。
例えばスカラーサブクエリは式を指定する位置で `()` で括ったクエリを書くことで利用でき、例によってこれまでと同じものを表現すると以下のようになる。

```sql
SELECT
  word,
  word_count,
  LENGTH(word) / (SELECT MAX(LENGTH(word)) FROM `bigquery-public-data.samples.shakespeare`) AS ratio_word_length
FROM
  `bigquery-public-data.samples.shakespeare`
ORDER BY
  ratio_word_length DESC, word
```

スカラーサブクエリは単一行の結果を返さないとエラーになることは注意が必要で、特に相関スカラーサブクエリを書く時に条件を適切に設定しないといけない。
例えば以下のようにクエリ（こんな書き方をする意味などないクエリだが、例として）を書くとエラーになり、これは `word` がユニークではないので `WHERE` 句の条件で単一行に絞れてないのでエラーとなる。
これを適切に動かすようにするには、`WHERE` 句の条件で単一行になるように、例えば元のテーブルを `DISTINCT word` などして `word` がユニークになるようにする、などとする必要がある。

```sql
-- これはエラーになる: Scalar subquery produced more than one element
SELECT
  word,
  word_count,
  (SELECT LENGTH(word) FROM `bigquery-public-data.samples.shakespeare` WHERE t.word = word)
FROM
  `bigquery-public-data.samples.shakespeare` AS t
```

スカラーサブクエリは他のテーブルの集計結果を持ってきて使用するとかには便利なのでちょっとした分析では使ったりするが、やりすぎると読みづらいので用法用量をお守りくださいという感じ。
自分用の分析ではちょくちょく使うが、他人に共有するクエリの場合はあまり使わないように心がけている、くらいの温度感で使っている。

式サブクエリの他のパターンとしては `Array` サブクエリがあり、名前の通り以下のような感じで `Array` を作るサブクエリを指す。

```sql
SELECT
  ARRAY(SELECT word FROM `bigquery-public-data.samples.shakespeare` LIMIT 5) AS array_test
```

<center>
<table style="border-collapse:collapse;border-spacing:0" class="tg"><thead><tr><th style="background-color:#F7F7F7;border-color:inherit;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">行</span></th><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">array_test</span></th></tr></thead><tbody><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit;color:rgba(0, 0, 0, 0.99)">1</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">LVII</span></td></tr><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">augurs</span></td></tr><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">dimm'd</span></td></tr><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">plagues</span></td></tr><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">treason</span></td></tr></tbody></table>
</center>

これは返しているのは 1 行で、その 1 行が `ARRAY` 型で 5 個の要素を持っていることに注意。
`ARRAY` は `STRUCT` と一緒に使われることも多いし、リテラルで書くときは `SELECT [1,2,3]` とか `SELECT (1,2,3)` のように同じような形で書いたりするので混同しがちだが、`STRUCT()` の方は以下のようにスカラー関数として機能しており、返してるのは 5 行である。

```sql
SELECT
  STRUCT(word, word_count) AS struct_test
FROM
  `bigquery-public-data.samples.shakespeare`
LIMIT
  5
```

<center>
<table style="border-collapse:collapse;border-spacing:0" class="tg"><thead><tr><th style="background-color:#F7F7F7;border-color:inherit;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">行</span></th><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">struct_test.word</span></th><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">struct_test.word_count</span></th></tr></thead><tbody><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit;color:rgba(0, 0, 0, 0.99)">1</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">LVII</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">1</span></td></tr><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit;color:rgba(0, 0, 0, 0.99)">2</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">augurs</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">1</span></td></tr><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit;color:rgba(0, 0, 0, 0.99)">3</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">dimm'd</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">1</span></td></tr><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit;color:rgba(0, 0, 0, 0.99)">4</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">plagues</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">1</span></td></tr><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit;color:rgba(0, 0, 0, 0.99)">5</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">treason</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">1</span></td></tr></tbody></table>
</center>

つまり以下のように `ARRAY<STRUCT>` 型を作ると、`ARRAY` 型として 1 行を返していて、その中の要素として 5 個の `STRUCT` が入ってるんだなと分かる。

```sql
SELECT
  ARRAY(SELECT STRUCT(word, word_count) FROM `bigquery-public-data.samples.shakespeare` LIMIT 5) AS array_struct_test
```

<center>
<table style="border-collapse:collapse;border-spacing:0" class="tg"><thead><tr><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">行</span></th><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">array_struct_test.word</span></th><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">array_struct_test.word_count</span></th></tr></thead><tbody><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit;color:rgba(0, 0, 0, 0.99)">1</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">LVII</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">1</span></td></tr><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">augurs</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">1</span></td></tr><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">dimm'd</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">1</span></td></tr><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">plagues</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">1</span></td></tr><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">treason</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">1</span></td></tr></tbody></table>
</center>

かなり基本的な話だが、こういう簡単なところで違いをちゃんと認識しておくと `ARRAY` と `STRUCT` を区別できて変なクエリを書いてエラーになることが少なくなる。

続いて `IN` サブクエリで、以下のように `WHERE` 句で条件指定するときにサブクエリを使うところで出番が多い。

```sql
SELECT
  *
FROM
  `bigquery-public-data.samples.shakespeare`
WHERE
  word IN (SELECT DISTINCT title FROM `bigquery-public-data.samples.wikipedia`)
ORDER BY
  LENGTH(word) DESC
```

<center>
<table style="border-collapse:collapse;border-spacing:0" class="tg"><thead><tr><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">行</span></th><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">word</span></th><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">word_count</span></th><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">corpus</span></th><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">corpus_date</span></th></tr></thead><tbody><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit;color:rgba(0, 0, 0, 0.99)">1</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">Northamptonshire</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">1</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">kingjohn</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">1596</span></td></tr><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit;color:rgba(0, 0, 0, 0.99)">2</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">Gloucestershire</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">3</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">kingrichardii</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">1595</span></td></tr><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit;color:rgba(0, 0, 0, 0.99)">3</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">Gloucestershire</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">2</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">1kinghenryiv</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">1597</span></td></tr><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal">...</td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">...</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">...</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">...</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">....</span></td></tr></tbody></table>
</center>

これもちょっとした条件をささっと書くのに便利だったりする。
`IN` の指定は `IN (1,2,3)` のように直接要素を入れることができたり、`IN UNNEST([1,2,3])` のように `ARRAY` を `UNNEST` したもので書くことができたり、色んなバリエーションがある。
あまり考えずに使っている分にはまあ便利な書き方かなという感じだが、真面目にここでの `()` はどういう意味なのか理解しようとしたり文法的に意味のあるものとして解釈しようとしてもうまくいかないので、こういうパターンもあるのね、というくらいに思っておくのが精神衛生上いいとは思う。


## tips6: 型変換
型変換について簡単に書く。
公式リファレンスは [https://cloud.google.com/bigquery/docs/reference/standard-sql/conversion_rules](https://cloud.google.com/bigquery/docs/reference/standard-sql/conversion_rules) である。

明示的な変換である `cast` と暗黙的な変換である `coerce` (強制型変換) がある。
前者は特に意識することもなく型変換を明示的にしたい場合に使えばいいが、後者はお手軽にクエリを書くときに（場合によっては知らず知らずのうちに）よくお世話になるもので、特に `WHERE date_column = "2021-01-01"` みたいな形で使うことが多い。

以下のように、`STRING` を特別な形式で書くと様々な時間に関する型へと強制型変換してくれる。

```sql
SELECT
  TIME(00, 00, 00) = "00:00:00" AS test_time,
  TIMESTAMP("2021-01-01 00:00:00") = "2021-01-01" AS test_timestamp,
  DATE("2021-01-01") = "2021-01-01" AS test_date,
  DATETIME("2021-01-01T00:00:0") = "2021-01-01" AS test_datetime
```

<center>
<table style="border-collapse:collapse;border-spacing:0" class="tg"><thead><tr><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">行</span></th><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">test_time</span></th><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">test_timestamp</span></th><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">test_date</span></th><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">test_datetime</span></th></tr></thead><tbody><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit;color:rgba(0, 0, 0, 0.99)">1</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">true</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">true</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">true</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">true</span></td></tr></tbody></table>
</center>

これはめちゃくちゃよく使うのでちゃんと意識しておくのがよい。

型変換に関しては supertype も頭に入れておくのがよい。
`UNION ALL` するときとか `CASE` 式を使うときに型が異なっていても supertype が同じなら処理できる。
例えば以下は `INT64` の supertype として `FLOAT64` があるため、`INT64` のデータが `FLOAT64` に強制型変換されて `FLOAT64` として扱われるようになる。

```sql
SELECT 1 AS test_supertype UNION ALL SELECT 2.0
```

<center>
<table style="border-collapse:collapse;border-spacing:0" class="tg"><thead><tr><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">行</span></th><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">test_supertype</span></th></tr></thead><tbody><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit;color:rgba(0, 0, 0, 0.99)">1</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">1.0</span></td></tr><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit;color:rgba(0, 0, 0, 0.99)">2</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:right;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">2.0</span></td></tr></tbody></table>
</center>


20211121 現在、`TIME`、`TIMESTAMP`、`DATE`、`DATETIME` はそれ自身のみが supertype だと公式リファレンスには書いてあるが、実は `DATE` の supertype として `DATETIME` があることが以下のクエリが実行可能で結果が `DATETIME` になることから分かる。

```sql
SELECT DATE("2021-01-01") AS test_supertype UNION ALL SELECT DATETIME("2021-01-01T00:00:0")
```

<center>
<table style="border-collapse:collapse;border-spacing:0" class="tg"><thead><tr><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">行</span></th><th style="background-color:#F7F7F7;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:middle;word-break:normal"><span style="font-weight:400;font-style:inherit;color:rgba(0, 0, 0, 0.99);background-color:#F7F7F7">test_supertype</span></th></tr></thead><tbody><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit;color:rgba(0, 0, 0, 0.99)">1</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">2021-01-01T00:00:00</span></td></tr><tr><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit;color:rgba(0, 0, 0, 0.99)">2</span></td><td style="background-color:#FFF;border-color:black;border-style:solid;border-width:1px;color:rgba(0, 0, 0, 0.99);font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;text-align:left;vertical-align:top;word-break:normal"><span style="font-weight:inherit;font-style:inherit">2021-01-01T00:00:00</span></td></tr></tbody></table>
</center>

BigQuery は型をよしなに型変換して扱ってくれるので便利な一方、型変換の基本的な振る舞いとか supertype がある程度頭に入ってないと気付かぬ間に期待と異なる型を取り扱っていたり、以前似たようなクエリ書いたときはエラーにならなかったのに今回はエラーになって困ったり、などが発生し得る。

今回取り上げたことだけでも頭に入っていれば分析用のクエリを書く時ににそこそこ役に立つのではないかなと思う。


## まとめ
BigQuery で分析する際の part2 としてスカラー関数・集計関数・分析関数、サブクエリ、型変換について書いた。
自分のブログはコードブロックの表示が綺麗ではないのでそろそろ改善したい気持ちになってきた...

---
---
<br>

