---
layout: post
title: TensorFlow Hub のソースコードを読む
categories: ['programming']
---


### TL;DR
- default で準備されているモデルで embedding を得る場合、 Module class の call メソッドに input を渡して実行するようになっている
- モデルは Tensorflow Hub が提供するものをダウンロードしてきて "default" とかで所定の tensor を名前引きして結果を得るようになっている
- モデルの情報がちゃんと残ってないのが辛い...
<br>

最近 TensorFlow Hub の NLP モデルで embedding を作って色々試したりしている。
主な用途としては（複数）文を入力としてその embedding で類似度を計算したりとかそういう感じ。

使っていると Universal Sentence Encoder (USE) が（BERT とか ELMo と比べて）良い性能を発揮しているが、こやつの詳しい情報があまりない。
論文を読んでどういうモデルなのかは理解したつもりだが、実装がどうなっているかとか具体的にどの層から embedding を引っ張ってきているかなどが分からん。

TensorFlow Hub で使われている学習済みモデルを学習した時のコードがあればラクなのだが、ちょっと探してもどこにあるのか分からない。
論文にも学習済みモデルが TensorFlow Hub で利用できるよ、ということくらいしか書いてない。

ということで TensorFlow Hub の内部のコードがどうなっているかを把握することも含めて、ちょっとソースコードを読んで色々理解してみよう、というエントリ。

### 基本的な使い方
そもそもの TensorFlow Hub の基本的な使い方について。
既に準備されているモデルを使って embedding を手に入れるには、以下のように書くだけでよい。

```
embed = hub.Module("https://tfhub.dev/google/universal-sentence-encoder-large/3")
embeddings = embed([
    "The quick brown fox jumps over the lazy dog.",
    "I am a sentence for which I would like to get its embedding"])

print session.run(embeddings)
```

前処理とか tokenize とかも全てモデルの方で実施するようになっている。
そのためお手軽ではあるが、ソースコード読んでも結局その辺はまるで分からん、ということになっている。
色々なモデルを扱う場合はそれぞれ前処理も違うので仕方ないけど、もうちょっとどこかに情報を残しといてもらいたいところだ。

### Module class の call メソッドを追う
上記の snippet を見れば分かるように、`Module` class の call メソッドを読んでそれを `session.run()` しているという構造になっている。

ということでここをちょっと真面目に追ってみるとしよう。
コメントを除けばコードとしては以下のものだけだ。

```
  def __call__(self, inputs=None,  # pylint: disable=invalid-name
               _sentinel=None, signature=None, as_dict=None):
    if self._graph is not tf_v1.get_default_graph():
      raise RuntimeError(
          "Module must be applied in the graph it was instantiated for.")

    signature = self._impl.get_signature_name(signature)
    safe_signature = signature.replace(":", "_")
    name = "%s_apply_%s" % (self._name, safe_signature)

    dict_inputs = _convert_dict_inputs(
        inputs, self._spec.get_input_info_dict(signature=signature,
                                               tags=self._tags))

    dict_outputs = self._impl.create_apply_graph(
        signature=signature,
        input_tensors=dict_inputs,
        name=name)
    return _prepare_outputs(dict_outputs, as_dict=as_dict)
```

ここの `signature` とか `name` は TensorFlow Hub が提供するモデルをそのまま使う場合は `default` と `module` と思っておけばよい。
（実際はモデル毎に定まっていて、自分も USE と ELMo くらいしか触ってないので断言はできないが、まあそこはよしなに）

ここで実装上重要になる次のものたちを見ておく。
`ModuleSpec` class と `ModuleImpl` class は抽象クラスになっていてインターフェースを提供しており、これらを継承する `_ModuleSpec` class と `_ModuleImpl` class が具体的な実装を有している。
`_ModuleSpec` は学習済みモデルの情報を担うもので入出力の情報を定めたり、`_create_impl` メソッドで `_ModuleImpl` object を return するものになっている。
`_ModuleImpl` が TensorFlow graph の諸々を扱うもので、特に `create_apply_graph` メソッドが肝になっていて、これで入力 tensor の情報と実際に計算される graph が与えられるようになっている。

Universal Sentence Encoder を例にとって、先ほどの `Module` class の call メソッドを追う作業に戻ろう。
`dict_inputs` に入るのは model の入力で、以下のように単なるテキストになっている。

```
{'text': <tf.Tensor 'Const_1:0' shape=(入力のテキストのリストの長さ,) dtype=string>}
```

そして `dict_outpus` に入るのは model の出力で、以下のように Encoder のどこかの出力を l2 normalize したもので、特徴量次元が 512 次元となっていることが分かる。

```
{'default': <tf.Tensor 'module_apply_default_1/Encoder_en/hidden_layers/l2_normalize:0' shape=(?, 512) dtype=float32>}
```

最後の return はこの `default` が key の tensor を返すので、あとはこれを `session.run()` に入れて実際に計算をして結果の配列が手に入る、ということになっている。

ということで、そんなに難しいところはない、ということが分かる。
モデルをダウンロードするところとかはちょっと読むのが面倒だったが、ここでは本筋と離れるので詳細は省こう。
GCS も使えるけど、用意されてるモデルを使う場合は指定した URL から tarfile をダウンロードしてきて解凍して、となっている。

### モデルの情報が知りたい
いままで読んできて分かるように、学習済みのモデルにテキストを渡して `default` とかで定められた出力を得る、という構造なのでモデルの中の情報は TensorFlow Hub にはない。
学習した時のスクリプトとか残しておいて欲しいが、無い。どこに情報があるのかも分からない。いただけない。

Webページにドキュメントがあるが、ELMo はまだもう少し詳しく書いてあるが USE は全然中身が分かるように書いてない。
入力もモデルの中で前処理しているのでどんなことが行われているかが分からない。情報もない。
USE の場合はテキストを入力にするが、モデルが tokenize して最初の方のいくつか（おそらく 512 token かな）の token までしか使われてないが、そういうのも全然書いてない。

学習済みモデルはあるわけなのでそれを TensorBoard とかで見てるんだけど、結構複雑だしなかなか骨が折れる。

もうちょっとちゃんと情報を残して欲しいなぁ。
論文もそこまで質が高くなくて宣伝論文みたいな感じだし、ちょっと残念な感じでしたね。

自分が知らないだけでちゃんと情報があるのかもしれないので、その場合はぜひ教えて欲しい。


## まとめ
TensorFlow Hub のコードを読んでみた。
コードを骨子を理解できたのはいいが、モデルの詳しい情報が全然残ってないのは不満だ。

---
---
<br>
