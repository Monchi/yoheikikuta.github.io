---
layout: post
title: Remote Development with VS Code を試してみた
categories: ['env development']
---


### TL;DR
- VSCode で container の中などでの開発をシームレスに実施できる拡張が発表された
- こういう機能は前から欲しかったので、ローカルで docker container 使って試した際のメモを残しておく
- まだ VSCode Insiders が必要な preview だけど、良い感じなので正式に取り込まれるのが楽しみ
<br>

最近は VSCode を使っていて、気に入ってもいる。

ただし docker で環境を用意することも多いので、container 内でプロセスを実行してそこでデバッグをしたいというときには不便な面もあった。
VSCode 入りの docker image をベースにして container で GUI アプリケーションを起動してそれを使えるようにする、みたいなのも試したことあるけど面倒だし key mapping がおかしかったりして結局使わなかった。
リモートでコーディングするときは Vim 使ってデバッガも適宜必要なのを使う、というところで落ち着いていた。

VSCode がサポートしてくれれば嬉しいなぁと思って過ごしていたところでのこの知らせということで、早速試してみた。
とりあえずローカルで container を立ててその中で Python プロセスを実行してデバッガを使ってみたので、備忘録としてブログに残しておく。
なかなか良さそうな感じだ。

まあ備忘録とか言わずに [公式のアナウンスページ](https://code.visualstudio.com/blogs/2019/05/02/remote-development) を見りゃいいという話はあるが、それはそれ。


### 前準備
ホストのマシンは macOS Moajave Version 10.14.4 で試している。

preview なので、実行には VSCode Insiders が必要である（[VSCode Insiders のダウンロードページ](https://code.visualstudio.com/insiders/)）。
試すには以下の preview の拡張をインストールする必要あり。
Container を使うだけなら Remote Container だけでもいいのだが、後々 SSH とかも試すと思うのでまあ全部入りをインストールしておく。

- [VisualStudio Marketplace: Remote Development](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack)

これをインストールすると VSCode の左下に緑色のアイコンが出現して各種機能を試せるようになる。
あとは [getting-started](https://code.visualstudio.com/docs/remote/containers#_getting-started) を見ながら試せばどんな感じか掴める。
自分は `vscode-remote-try-python` を試してみた。

これは Microsoft が準備しているレポジトリに沿って試すという感じなので、もっとシンプルなものを自分で試しておこう、ということでやってみた。


### 一から自分で試してみる
container で Python の hello world を実行して、それを VSCode のデバッガでデバッグするということをしてみる。
これができるようになれば container 内での作業に関してはまあだいたいオッケーだろう。

まずは VSCode で適当なフォルダもしくはワークスペースを開く。
何でもいいので `test` とかいうフォルダを作ってそれを開く。
これをしとかないと以下に書いている必要な Dockerfile とかをデフォルトで準備してくれるコマンドが出現しないので注意。

開いたら左下の緑のアイコンをクリックして `Remote-Containers: Create Container Configuration File...` を選択する。
これを選択すると色々なテンプレートが表示される。
色々使うようになったら自分用のテンプレートを準備するのだと思うが、最初は `Python 3` を選んだ。
これを選ぶと `.devcontainer` というフォルダができて Dockerfile やら設定を記した json ファイルなどが自動で作成される。
軽く眺めると、ローカルのワークスペースの path だったり requirements.txt.temp（これは temp ということで空なので必要なものは自分で準備）だったりが生えていて、それらを元に必要な docker container を作ってくれるようになっている。
これは自分用のを作るのも難しくなさそうやね。

この `.devcontainer` があるワークスペースを VSCode がよしなに解釈して自動的にに container を作って、その中の環境で VSCode を使った作業ができるようにしてくれる。
新しく作った場合は reload するかというポップアップが出るのでそれで reload すればよい。
ということで reload すると docker image をビルドするのでいくらか時間が掛かり、首尾よく進むと以下のような画面になる（ここでは例として `test.py` というファイルを作って編集している）。

<div align="center">
<img src="https://i.imgur.com/7lYgVi6.png" width="800">
</div>

ターミナルは最初は `Dev Containers` で VSCode Server が動いていることが分かるが、新しいターミナルを開くとこのように container 内で root ユーザとして bash プロセスが起動するようになっている。

なので、この terminal で `python test.py` などを実行すれば、このスクリプトが実行されて標準出力に `Hello From Container!` と表示されることが見て取れる。

VSCode でコーディングしてみれば、intellisense が機能していることも確認できる。お〜いいっすね。

最後に VSCode のデバッガを使ってみる。
通常の VSCode のように使えるので、適当にブレークポイントを貼って `F5` を押せばよい。
debug configuration が色々出てくるので、今回は `Python File` を選ぶ。
すると以下のように確かにデバッガが機能していることが分かる。
当たり前だけどステップ実行で VARIABLE が変わることも確認できる。
やったぜ。

<div align="center">
<img src="https://i.imgur.com/joJnnDv.png" width="800">
</div>


## まとめ
Remote Development with VS Code を試してみたけど、自分が欲しかったものでいいね！
今回はローカルの container で試しただけだけど、SSH とかも提供されているのでそっちも近いうちに試してみようかね。

---
---
<br>
