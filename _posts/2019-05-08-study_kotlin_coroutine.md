---
layout: post
title: Kotlin Coroutine の勉強経過記録
categories: ['programming']
---


### TL;DR
- Kotlin の Coroutine を勉強したときの記録
- 勉強した内容をまとめる目的じゃなくて、何をどういう順番でやって何が分かって何が分からなかったかを残しておく目的
<br>

Kotlin で Android アプリを作ってみたりしているが、まあ色々と分からないものが出てくる。
この宇宙には分からないことが多すぎて人生に飽きなくて済むことは確定しているので、その点に関しては安心でして人生を送ることができる。

とりあえず色々教えてもらって Coroutine を使ったコードを書いてはみたが、中身は理解してないのでその辺を勉強してみようという話である。
やった内容をいい感じにまとめるというのではなく、経過をメモとして残しておくだけのもの。

最終的にちゃんと理解したというものでもなくて、とりあえずやってみたところまで。
なので他の人が見て何か参考になるところはあまりない（と思う）、ということを最初に断っておこう。


### 勉強に使った資料
最終的に参考にした資料を最初に挙げておく。

- [公式リファレンス](https://kotlinlang.org/docs/reference/coroutines/coroutines-guide.html)  
  実際にどう書けばよくてどう動くか、ということに関してはやはり公式リファレンスを見るのが一番
- [KotlinConf 2017 - Introduction to Coroutines by Roman Elizarov](https://www.youtube.com/watch?v=_hfBv0a09Jc)  
  そもそも論として非同期プログラミングとか全然知らないので、そこから話してくれるのは助かる
- [KotlinConf 2017 - Deep Dive into Coroutines on JVM by Roman Elizarov](https://www.youtube.com/watch?v=YrrUCSi72E8)  
  Coroutine が Kotlin においてどう実装されているかの概略。
- [Suspend 関数の話](https://www.amazon.co.jp/Suspend%E9%96%A2%E6%95%B0%E3%81%AE%E3%81%AF%E3%81%AA%E3%81%97-%E6%9C%89%E9%87%8E%E5%92%8C%E7%9C%9F-ebook/dp/B07MDXF3YZ)  
  Coroutine の実現において中心的役割を担う suspending function の実装詳細。100円安すぎる！
- [Scheme:使いたい人のための継続入門](https://practical-scheme.net/wiliki/wiliki.cgi?Scheme%3A%E4%BD%BF%E3%81%84%E3%81%9F%E3%81%84%E4%BA%BA%E3%81%AE%E3%81%9F%E3%82%81%E3%81%AE%E7%B6%99%E7%B6%9A%E5%85%A5%E9%96%80)  
  Scheme は分からないが継続渡しを勉強するときに参考にした。
- [kotlinx.coroutines reference documentation](https://kotlin.github.io/kotlinx.coroutines/)  
  主に core を眺めながらシグネチャとかを見たりした。


### とりあえず紹介してもらった動画から見てみる
まず Deep Dive into ... を見てみた。

まず CPS (Continuation passing style) が分からない。
分からないが、callback と同じだという話なのでそういうもんだと思って進める。
とりあえず Coroutine は `suspend` とつけてあとは普通の関数っぽく書けば良いというお手軽な感じになっていることを知る。

詳細を把握はしてないが、継続を使って実装されていて、ステートマシンでそれぞれの suspending function に付与されたラベル情報を保持しながら該当の部分にジャンプするようになっているんだなということを知る。
この辺は継続をちょっと前に勉強したのでイメージはつく（実際にどうラベルがつけられているかなどは分からないが）。
ステートマシンとして実装することで、callbacks だと処理が重なる場合にラムダ式が連なって厳しくなるところも普通の関数っぽく書けるという話。
ほい。

その次あたりからついていくのがしんどくなってくる。
Java8 ではいろんなライブラリがいろんな future を作って大変だとか、前のトークでの例を持ってきたりとか、suspending function では普通（他の言語）では future object を返すところをそういう object を使うのではなく統合ライブラリから `.await()` を呼ぶようになっているとか、そういう話になる。
んで更に `suspendCoroutine` のところでこれは coroutine builder とは逆で引数には通常の関数を使って云々とか、これは Scheme の call with current continuation にインスパイアされてますとか出てきて、ちょっと何言ってるか分からないとなって、このまま見てても理解できないことを察して撤退した。

もう少し基本的な内容を押さえておかないとこの動画を見ても分からんなと思ったので、公式リファレンス辺りから攻めることにする。


### リファレンスを眺めて Introduction ... を見てみる
公式リファレンスの Coroutine に関するいくつかのトピックを読み、そして前段のトークだったという Introduction ... を見てみた。

まず公式リファレンスは `Coroutine basics` から `Coroutine context and dispatchers` までを読んでみた。
公式リファレンスを読んでどんな感じのものなのかがだいぶ掴めてきた。
Coroutine は軽量なスレッド的なものであること、実行の際には `CoroutineScope` というスコープ内で `launch` を使って実行すること、non-blocking だけど子の処理が終わるまで待つ、`launch` の中身を抜き出して外で定義するときに `suspend fun` とすること、などなど。

ていうかね、ぶっちゃけ future/promise とかどんなもので何が違うとか分かってないレベルなんですよ。
あとで実行される結果を保持するオブジェクトで、色んな言語で色んな使われ方してるんですかね、という程度の理解。

まあ自分の知識不足はとりあえず置いといて先に進める。
`launch` は `Job` interface を返して `async` は `Deferred` interface を返すということを学ぶ。
Kotlin では素直に書くと sequential に実行されるコードになり、`async(...)` で作って `.await()` を使うと coccurent に実行されるコードになるんですな。
そして `dispatcher` で Coroutine 実行時のスレッドをコントロールする。

公式リファンレスではサンプルコードがあってそれを実行しながら読み進められるのでこれでだいぶ感覚が掴めてきた。
ここまでの準備で Introduction ... の動画にチャレンジしてみる。

スレッドはメモリ結構喰う（~1[MB/thred])ので 1,000 とか 10,000 になると大変なのでこれを救済しようという話から始まる。
callback を使うのはラムダ式が重なる辛さがある → future を使うのはましになるが combinator がライブラリごとに異なるのが辛い → そこで Coroutine だ、とか説明してもらえるのは初心者には嬉しい。
suspending function を使えば通常の関数のように書けること、`launch` やコンテキストを `dispatcher` で指定するという話はリファレンスで読んでおいたのもあるのですんなり入ってきた。

そのあとは C# との対比で async/await の説明になるが、ここで C# では普通に書くと concurrent で `await` と陽につけると sequential になるが、Kotlin では逆で、普通に書くと sequential で `async` と陽につけると concurrent になるという話が出てくる。
どっちをデフォルトとするかは哲学が出るところかと思うが、Kotlin では例えば、何かリクエストしてその結果を待ってその次の処理をして、みたいなのが多いだろうということでこうなっているとのこと。
ふむふむ。

concurrent が必要なときは `async {...}` という Coroutine builder で `Deferred<T>` を返すようにして、これが他の言語の future に対応する、ということでこの辺まで聞いてくるとまあまあ整理されてきた感がある。
Javaとの互換性とかは置いといて（すみません）、最後に Kotlin では Coroutine は standard library の外にある kotlinx.coroutine で実装されているぞ、という話で終了。

公式リファレンスをある程度見たおかげで、基本的な使い方とか意味はイメージがつくようになってきた。
ここらでもっかい Deep Dive into ... に挑戦するか、と思ったがその前に継続渡しをちょっと見ておくことにする。


### 継続渡し
Scheme の記事を読んでみる。
とりあえず流し読みしてみたらコードはパースできないし内容も深くて難しいので、はっきり言って全然分からない。
頑張って気合いを入れて読んでみる。

継続渡し形式の階乗の定義を n = 0 の場合から丁寧に見ていくことで理解した。
一旦理解すれば `(fact/cps 10 (lambda (a) (* 2 a)))` の lambda 式の部分が `cont` として渡されて decrement するときに引き続き渡されてい構造が得心できた。 
前置記法ではあるが、スタックでの取り扱いを PostScript でやったおかげで理解しやすかった部分もある。

これが分かったあとは複数の継続手続きを渡す次の除算もすんなり理解できる。
fail の継続手続きから始めて、`read` した結果が非ゼロなら `pass` の方の継続が渡されてその引数は `(/ num den)` となっている。

```
;; 継続渡し形式の除算
(define (divide/cps num den pass fail)
  (if (= den 0)
      (fail num den pass fail)
      (pass (/ num den))))

;; 非零の分母要求する除算
(divide/cps 10 0
            (lambda (a) (format #t "answer is ~a~%" a))
            (lambda (n d p f)
              (format #t "> ")
              (flush)
              (divide/cps n (read) p f)))
```

継続渡しの雰囲気は掴めてきたが、本題の call with current continuation (call/cc) を理解するために call with 系関数の説明が入る。
ちょっとはしょるが、重要になる部分は以下の「ファイル名を受け取ってそれを開くという準備を整えた上で `proc` を call する」というものだ。

```
;; とんでもなく杜撰なcall-with-input-fileの実装(コアのみ)
(define (call-with-input-file file-name proc)
  (proc (open-input-file file-name)))
```

これからの類推で `call-with-procedure` は以下のようにラムダ式を使って書けるだろうと考えられる。
`lambda-expr` を準備して `proc` を call するというものになっている。

```
;; call-with-procedureの実装
(define (call-with-procedure lambda-expr proc)
  (proc lambda-expr))
```

例としては `(call-with-procedure (lambda (a) (* 2 a)) (lambda (p) (p (fact 10))))` のように使えて、これは `(* 2 (fact 10))` という結果を返す。
うん、少しずつ読めるようになってきた。

ここに更に継続を絡めてみる、`call-with-continuation-procedure` だ。
先ほどの `lambda-expr` を `cont` に変えただけなので、それはそうという感じだ。

```
;; call-with-continuation-procedureの定義
(define (call-with-continuation-procedure cont proc)
  (proc cont))
```

本題の `call/cc` に辿りつくには `call-with-continuation-procedure` を起点にする。
例によって `(* 2 (fact 10))` を題材にすると次のように書ける。

```
(call-with-continuation-procedure (lambda (a) (* 2 a))
                                  (lambda (cont) (cont (fact 10))))

;; fact/cpsをcall-with-continuation-procedureを使って定義する
(define (fact/cps n cont)
  (call-with-continuation-procedure cont (lambda (c) (c (fact n)))))
```

ここで継続渡し `(lambda (a) (* 2 a))` に注目しよう。
ここは一般には継続渡しが連なる構造になってもいいわけだが、そういう可能性は考えずに現在の `(lambda (a) (* 2 a))` だけを相手にすると決めてしまうとする。
そうすればここでの `lambda` などは省略するように取り決めても任意性は生じないはずで、 `* 2` を外出しして次のように書けると期待できそう（[]は省略するぞの気持ち）。

```
(* 2 (call-with-continuation-procedure [(lambda (a) (a))]
                                       (lambda (cont) (cont (fact 10)))))
```

この省略しますよというときに新たに `call/cc` という記号を使うことにすれば、ゴールに辿り着く。

```
(* 2 (call/cc (lambda (cont) (cont (fact 10)))))
```

ということでなんとなく分かった風に書いてみたが、これは間違っている。
ゴールである `call/cc` でこう書けるのは合っているのだが、その前段の `call-with-continuation-procedure` で何となく `* 2` を外出ししたり省略したところが間違っている。
`call/cc` が実現したいのは現時点での継続を作ってそれを proc に渡して call するということなので、その続きの計算も作った継続を介することで実施していきたい。
ということで続きの REPL を実行するような `PRINT-AND-NEXT-REPL` というものを便宜的に持ち込むと、次のように書ける。

ちなみにここは何言ってるか分からないと思う（言ってる自分がよく分かっていないので）。

```
;; call/ccの継続渡し形式への書き換えを試みてみる(PRINT-AND-NEXT-REPL導入版)
(* 2 ((lambda (cont) (cont (fact 10)))
      (lambda (a) (PRINT-AND-NEXT-REPL (* 2 a)))))
```

`lambda` を計算していくと最終的に `(* 2 (PRINT-AND-NEXT-REPL (* 2 (fact 10))))` が得られる。
最初の `* 2` は継続を作るときに材料として使われはしたが別にそれによって消えるわけではないので残っていて、しかし `PRINT-AND-NEXT-REPL` によって次の REPL に飛ぶので実行されることはない。

なんだこれは。
分からん。
分からんが、分からんなりにもう少し考えてみるために他の例を考える。

まずはこれ。
`lambda` で `cont` を使わないもので、これは `call/cc` が作った継続は使われずに処理が流れていく。

```
(* 2 (call/cc (lambda (cont) (+ 2 3))))

(変換後)
(* 2 ((lambda (cont) (+ 2 3))
      (lambda (a) (PRINT-AND-NEXT-REPL (* 2 a)))))

(lambda 計算後)
(* 2 (+ 2 3))
```

次にこれ。
これは `* 2 10` が計算されたら `PRINT-AND-NEXT-REPL` なので、残りの `+ 2 3` などは実行されない。

```
(* 2 (call/cc (lambda (cont) (+ 2 3 (cont 10)))))

(変換後)
(* 2 ((lambda (cont) (+ 2 3 (cont 10)))
      (lambda (a) (PRINT-AND-NEXT-REPL (* 2 a)))))

(lambda 計算後)
(* 2 (+ 2 3 (PRINT-AND-NEXT-REPL (* 2 10)))))

```

継続なので理解が難しい部分もあるし実際まだ理解したというレベルでもないが、計算を実行するための情報であることを考えればこういうものなのかなと感じるくらいにはなってきた。

これを使えば計算をスキップして大域脱出みたいなこともできそうだという感触が得られてきたので、一旦この辺で終えておく。
勉強したことがない対象だったのでだいぶ長々と続いてしまった。
ここらでもっかい Deep Dive into ... を見てみよう。


### Deep Dive Into ... (再訪)
前回挫折したあたりの `suspendCoroutine` が Scheme の `call/cc` inspired だというあたりを考え直してみる。
まずこれは suspending function なのでどこかの CoroutineScope の中で呼ばれることで　Coroutine の実行を中断する。
そしてその時点での継続を作るので、Kotlin ではこれはステートマシンに状態としてラベルなどを保存して、どこかで `resume` が呼ばれることで再開されるということになっているんだろう。
合ってるか分かってはいないが、少なくともある程度想像できるようにはなってきた。

と思って次の Continuation Interceptor になるとまたよく分からん。
CoroutineContext が key と element という形で Coroutine の情報を保持していて、interceptor でそれをガッと取って別のスレッドで使うみたいな感じだろうか。
dispatcher がこれを使っていると。
なんていうか、そんな感じかのイメージくらいまでしか到達できず実際のところどうなっているのかは分からない。

Java の future との統合が楽だとか Job cancellation の話で Java の `Thread.stop()` が deprecated とかいうのはまあそんなもんかと聞き流す。
cooperatie cancellation は Coroutine がキャンセルしたいときにちゃんとキャンセルできるようにという考えで、実際に `kotlinx.coroutines` の suspending function は全てキャンセル可能になっている。
そのために `CancellableContinuation` とか `suspendCancellableCoroutine` とかを使っているとのことだが、こういうのもまあそうなんかなぁくらいで実際のところどうなってるかはよく分からん。
例えば `Job.cancel()` が実際のところどういう風に実現されているかとか分からない。
ソースコードとか読もうと思ってもどう追えばいいのか分かってない。

ということで全体的にしんどい。
時間を掛けてもなかなか進んでいかない。

これくらいで諦めておいて、Suspend関数の話を読んで識者に色々聞くということにしよう。


### Suspend関数の話
読んでる途中なので後で書く。


## まとめ
Kotlin の Coroutine を勉強した際の経過をメモしておいた。
全然読み解けなぁい！

---
---
<br>
