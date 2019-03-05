---
layout: post
title: リモートサーバのファイルをローカルの VSCode で開く
categories: ['programming']
---


### TL;DR
- ローカルの VSCode でリモートサーバのファイルを開きたい
- ssh トンネルを掘ってリモートサーバで rmate などを使う
- local で docker container を作って試した
<br>

VSCode を使うことが増えてきたので、ちょっとした tips をちょこちょこと残していきたい。
今回はリモートサーバのファイルをローカルの VSCode で開くというもの。

この手のは調べれば色々と出てくるのだが、自分がやったものを備忘録として残しておく。


### リモートサーバ側
[rmate](https://github.com/aurora/rmate) をインストールする。  
これはリモートサーバに置かれているファイルを扱うためのヘルパーツールである。
デフォルトでポート `52698` を使用していて、後から ssh トンネルを掘ってリモートサーバのこのポートで受けた接続をローカルのポートに流して VSCode と繋げようという算段である。

```
$ wget -O /usr/local/bin/rmate https://raw.github.com/aurora/rmate/master/rmate
$ chmod a+x /usr/local/bin/rmate
```


### ローカル側
まずは VSCode の拡張機能「Remote VSCode」をインストールする。
再起動後に `settings` の `Remove VSCode` において `Remote: Onstartup` の `Launch the server on start up` にチェックを入れておく。
そして `CMD+Shift+P` で `Remote: Start Server` を実行しておく。
これはデフォルトでポートが `52698` になっていて、リモートサーバ側からの接続を受けるようになる。

次いで terminal を開いて下記コマンドで ssh トンネルを掘る。

```
$ ssh -R 52698:localhost:52698 <virtual-machine-ip-address>
```

`-R` オプションはリモート側でポートを開いて接続を受けたものをローカルのポートに流すものである。
`-L` の逆向きの働きをする。

この ssh 接続でリモートサーバに接続したら、開きたいファイルを `rmate` を使用して開くことができる。

```
(remote) $ rmate test.py
```

これでローカルの VSCode で `test.py` が開いて終了。


### docker container で実験
ここまで特にどうということはなかったが、せっかくなので docker container で試しておく。

リモートサーバの代わりをするような ssh 接続ができる docker container を立てよう。
色々調べてぐちゃぐちゃやったりしたが、結論は公式ページの情報に従うのが良い。  

ということで [公式ページ](https://docs.docker.com/engine/examples/running_ssh_service/) に従って Dockerfile を準備して docker image を build する。

```
docker build -t eg_sshd .
```

この docker image を使って container を起動する。
こいつはポート `22` を EXPOSE しているが、ホストのポートを何番で受け付けるかは指定しないと自動で決まるのでそれを `docker port` で調べておく。

```
$ docker run -d -P --name test_sshd eg_sshd
$ docker port test_sshd 22
0.0.0.0:32771
```

調べたポートを使って ssh 接続をする。

```
$ ssh -R 52968:localhost:52968 root@localhost -p 32771
root@localhost's password: 
```

ここまで来れば後はリモートサーバの場合と同様である。
`rmate` をインストールして同様の手順を踏めば良い。

ホスト上で動かしてる docker container なら directory をマウントしておけばこんなことしなくてもファイルはいじれるじゃん、ということは置いといて手順はこんな感じでしたとさ。


## まとめ
リモートサーバ上のファイルを ローカルの VSCode で開く手順をまとめておいた。

---
---
<br>
