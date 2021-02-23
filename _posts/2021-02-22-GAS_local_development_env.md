---
layout: post
title: clasp と TypeScript で GAS の開発環境を整えた備忘録
categories: ['programming']
---


### TL;DR
- ちょっとした仕事で GAS を使うときがある
- ローカルで clasp と TypeScript を使って開発する環境を整えた（が、いまの自分の用途ではここまでしなくてもよかった）
- 完全に忘れそうなので備忘録としてブログに残しておく
<br>

仕事で Google Workspace を使っていると、そのサービスを使ったちょっとしたタスクをするときに Google App Script (GAS) を使う時がある。  
ちょっとしたコードにトリガーを設定して定期実行するというのも便利だったりする。

スクリプトエディタで書いてたりもしたが、Git/GitHub での管理がしづらかったり、ローカルでの開発環境を提供する clasp [https://github.com/google/clasp](https://github.com/google/clasp) が TypeScript をサポートしているという話を知ったり（だいぶ遅い）して、せっかくなのでローカルでの開発環境を整えてみるかと思い立ってやってみた。

一瞬で全て忘れそうなので備忘録としてブログに残しておく。


### 諸々のインストール
自分が使ってる Node と npm のバージョンは以下。

```
$ node -v
v15.9.0
```

```
$ npm -v
7.5.3
```

npm で必要なパッケージをインストールする。
TypeScript とか TSLint とかも入ってるけどそこはまあよしなに。

```
$ npm install -g typescript tslint @google/clasp @types/google-apps-script
```

自分が使っているのは以下のバージョンになっている。

```
$ npm list -g
/Users/yoheikikuta/.npm-global/lib
├── @google/clasp@2.3.0
├── @types/google-apps-script@1.0.25
├── clasp@1.0.0
├── tslint@6.1.3
└── typescript@4.1.5
...
```

`clasp login` で Google OAuth 2.0 を使った認証が実施され、Authorization が成功すると `~/.clasprc.json` というファイルにログイン情報が保存される。


### 開発のためのセットアップ
適当にディレクトリ（ここでは `sample` とする）を掘って `npm init` などをする（yarn がよかったらよしなに読み替える）。

```
$ mkdir sample
$ cd sample
$ npm init -y
$ tslint --init
```

このディレクトリで新しい script project を作成する。
`standalone` の script project を作成する。  
各種コマンドに関しては [https://github.com/google/clasp](https://github.com/google/clasp) を見ればいいんだけど、ここでは script project のタイトルと root directory を設定している。
後者を設定しておくとこの root directory に置いてあるものだけが push したときにスクリプトエディタに送られて便利なので設定しておくのがよい（これを設定しない場合は不必要なファイルを push しないように `./.claspignore` を作る必要がある）。

```
$ clasp create --title "sample-title" --rootDir ./src 
? Create which script? standalone
```

GAS API を使ったことがなければ初回はエラーが出るので、[https://script.google.com/home/usersettings](https://script.google.com/home/usersettings) で `Google Apps Script API` をオンにする必要がある。

セットアップが整ったことを確認するために、簡単な GAS を作って push して実行してみる。
`./src/sample.ts` を以下のように準備する。

```ts 
function test(): void {
    console.log("hellow world.");
}
```

このファイルを以下のコマンドで Google Drive に push する。`./src/` 以下のファイルが push される。  
このときに `.ts` ファイルが `.gs` ファイルにトランスパイルされて、GAS として実行できるようになる。

```
$ clasp push
└─ src/appsscript.json
└─ src/sample.ts
```

push されたファイルは GAS として Google Drive に保存される。
GAS をスクリプトエディタ（script.google.com）で開くには以下を実行する。

```
$ clasp open
```

これで以下の図のようにスクリプトエディタが開く。
`// Compiled using ts2gas 3.6.4 (TypeScript 4.1.5)` とか書いてあってトランスパイルされたことも確認できる。
型アノテーションも落ちている。  
あとはこれを `実行` すればよいという寸法である。

<div align="center">
<img src="https://i.imgur.com/u4zGPTM.png" width="600">
</div>

Google Drive に GAS として保存されたコードをローカルに持ってくるには `clasp pull` をすればよい。
これをすると `.gs` ファイルから `.js` ファイルにトランスパイルされてローカルの `./src/` 以下に保存される。
pull するとローカルに同じ内容の `.ts` ファイルと `.js` ファイルが存在するようになってしまうので、再び push すると 400 エラーで `A file with this name already exists in the current project: sample` となるので、push するには `.js`　ファイルを削除する必要がある。
ローカルで開発する場合は pull は殆ど使わない、ということになりそう。  
（`./clasp.json` に `"fileExtension": "ts"` を追加しておくと `.ts` ファイルとして pull してくれるが、当然型アノテーションとかは破棄されてるので嬉しいことは特にない。）

これだけでもやりたいことはやれなくはないが、コードを動かすために push してから毎回ブラウザを開いて実行ボタンを押下しないといけないのでしんどい。
App Script のランタイムは Google の基盤にあるので push して実行しないといけないのはいいのだが、せめてローカルでコマンドを叩いて実行してその結果をローカルで確認する、くらいはしたい。

それをするためには色々と面倒な準備をしないといけないので、以降ではそのやり方を記録していく。


### ローカルで `clasp run` が実行できるようにする
これがダルい。
かなりダルいのでもっといい感じにできるようになって欲しいが、とりあえず自分がやった手順を残しておく。

何もしない状態で `clasp run` を実行するとどうなるかをまず見ておく。
`clasp run` を実行するとどの関数を実行するか選択できるので、今回であれば `test` を選ぶ。

```
$ clasp run
Running in dev mode.
? Select a functionName test
Could not read API credentials. Are you logged in locally?
```

こんな感じで API credentials が読めないと怒られる。
これを解決するために具体的な手順として以下を実施することになる。

- GCP プロジェクトと連携
- GAS を実行可能 API として公開
- App Script API を有効化
- 認証情報の作成とそれを用いたログイン

GCP プロジェクトと連携するにはスクリプトエディタで `プロジェクトの設定` を選択し、GCP の `プロジェクトの変更` から連携させたい GCP プロジェクトのプロジェクト番号を設定すればよい。

実行可能 API として公開するには、スクリプトエディタの `デプロイ > 新しいデプロイ` において `実行可能API` としてアクセスできるユーザをお望みのものに設定した上でデプロイする。
この段階で `clasp pull` すると `./src/appsscript.json` に executionAPI が追加されることが確認できる。

```json
{
  "timeZone": "America/New_York",
  "dependencies": {},
  "exceptionLogging": "STACKDRIVER",
  "runtimeVersion": "V8",
  "executionApi": {
    "access": "MYSELF"
  }
}
```

GCP のプロジェクトで `APIとサービス > ライブラリ` で `Apps Script API` を検索して有効にすることもやっておく（もし過去に有効にしたことがなければ）。

最後に認証情報を作成してそれを使ってログインをする。  
まず、`APIとサービス > 認証情報` の `認証情報を作成` で、`OAuth クライアント ID` を `デスクトップアプリ` アプリケーション用に作成する。
この認証情報を json としてダウンロード（以降ではこのファイル名を `creds.json` とする）して適当な場所に置く（ここでは `sample` ディレクトリ直下に置いたものとする）。  
この認証情報を使いローカルでログインし直してブラウザで認証を実施する。
このとき `? What is your GCP projectId?` と聞かれるので GCP のプロジェクト ID を入力する。

```
$ clasp login --creds creds.json 
```

これが成功すると、`sample` ディレクトリ以下にこのプロジェクトで使用するアクセストークンなどが記載された `.clasprc.json` というファイルが作成される。

ここまで来れば `clasp run` が実行できるようになるのでやってみる（以下のように実行したい関数名を指定して実行することができる）。

```
$ clasp run test
Running in dev mode.
No response.
```

ということで、動く！  
`clasp run` では返り値が表示されることになるので `console.log()` とかの結果は表示されない（これに関しては次の節で書く）。
適当な string とかを return すればちゃんと結果が表示されることが確認できる。


### ログの確認
ログは GCP の Cloud Logging で管理されている。
clasp のコマンドでローカルでも確認できるようになっている。

```
$ clasp logs
```

ただし、これは連携している GCP プロジェクトに関するログがそのまま表示されてしまうので、そのプロジェクトが色々な用途で使われているものだとお望みの GAS のログを見つけるのが難しい。
GCP のログエクスプローラだとフィルタリングのためのクエリが発行できるのでそれを clasp でも使えるとよいのだけど、20210223 時点ではサポートされていない。

諦めてログエクスプローラで以下のようなクエリ（その他必要な条件は適宜つけて）を発行して見るしかないかな。

```
resource.type="app_script_function"
resource.labels.invocation_type="apps script api"
```

### GAS のサービスを使う
ここまで長々とやってきたが、GAS を使うなら GAS のサービスを使いたいわけで、それをするには使いたいサービスに応じて適切な permission を設定する必要がある。
これは必要なものを `./src/appsscript.json` に記載しましょうという話なのだが、せっかくなので一つ例を。

先ほどまで使っていた `./src/test.ts` を以下のように書き換える。

```ts
function test(): string {
    var response = UrlFetchApp.fetch('https://httpbin.org/get');
    return response.getContentText();
}
```

これを実行しようとすると以下のようなエラーが出る。

```
$ clasp push && clasp run test
Exception: ScriptError Exception: You do not have permission to call UrlFetchApp.fetch. Required permissions: https://www.googleapis.com/auth/script.external_request [ { function: 'test', lineNumber: 3 } ]
```

この permission を `./src/appsscript.json` に加える。

```json
{
  "timeZone": "America/New_York",
  "dependencies": {},
  "exceptionLogging": "STACKDRIVER",
  "runtimeVersion": "V8",
  "executionApi": {
    "access": "MYSELF"
  },
  "oauthScopes": [
    "https://www.googleapis.com/auth/script.external_request"
  ]
}
```

これを追加したら改めてログインして再度実行する。 
`? Manifest file has been updated. Do you want to push and overwrite?` と聞かれるので `y` で答えると変更した `appscript.json` が push される。うまくいけば以下のようにちゃんと結果が帰ってくる。

```
$ clasp login --creds
...
$ clasp push && clasp run test
...
Running in dev mode.
{
  "args": {}, 
  "headers": {
    "Accept-Encoding": "gzip,deflate,br", 
    "Host": "httpbin.org", 
    ...
  }, 
  ...
  "url": "https://httpbin.org/get"
}
```

これくらいまでできるようになれば、何かしらのサービスの API を叩いてちょっとした処理をするのを GAS で定期実行するコードを書く、とかもローカルでできるようになってめでたしめでたし。


### その他
`clasp run foo` で関数 foo を実行するときに引数を渡すこともできるが、これはシングルクォーテーションで囲んで例えば string なら `clasp run foo --params '"paramString"'` みたいに書く必要があるので注意。
まあ README [https://github.com/google/clasp/tree/ee70a6ae743d3b28b27b2c5cc6ca3e70c64f465b#options-11](https://github.com/google/clasp/tree/ee70a6ae743d3b28b27b2c5cc6ca3e70c64f465b#options-11) とかコードをちゃんと読めってだけの話なんだけど。

npm パッケージを使いたい場合はどうするんだっけというのもあるが、これはローカルでビルドして push せいという話みたいなので自分にも必要性が発生したら調べたらよさそう。

それ以外にも設定ファイルの細々したこととかあるけど、大した話じゃないので割愛。


### テストとかデバッグとか
GAS を使いたいときは GAS のサービスを使いたいときが多いので、やはりローカルだと限界があって、結局はスクリプトエディタでデバッガを使いながら期待通りの動きをしているのか確認する、というのが試行錯誤しながら開発するときは効率的ということになりそう。

ある程度しっかり開発するなら、ローカルでモックを準備してテストもできるようにして、とか複数人で開発できるようにして、とかで今回のような環境をちゃんと作った上でやっていくのがよいと思うけど、自分の用途としてはそこまでガッツリしたものを GAS で作ることはなかなかなさそう。

ということで、自分の用途に限れば、黙ってスクリプトエディタで開発して出来たコードは gist なりなんなりで共有しておけば十分、という感じかもしれない。「そしたらなんのためにローカルで TypeScript で書いたの」だって？
せっかくだからちょっとやってみようと思ったからだよ、こまけぇこたぁいいんだよ。（AA略

オチがついたところでこのエントリは終了。


### 参考にした情報
文中で書いたものもあるがまとめて羅列しておく。

clasp: [https://github.com/google/clasp](https://github.com/google/clasp)  
Advanced Development Process with Apps Script: [https://developers.googleblog.com/2015/12/advanced-development-process-with-apps.html](https://developers.googleblog.com/2015/12/advanced-development-process-with-apps.html)  
GAS overview: [https://developers.google.com/apps-script/reference](https://developers.google.com/apps-script/reference)  
GAS manifest structure: [https://developers.google.com/apps-script/manifest](https://developers.google.com/apps-script/manifest)  
GAS Executing Functions using the Apps Script API: [https://developers.google.com/apps-script/api/how-tos/execute](https://developers.google.com/apps-script/api/how-tos/execute)  
GAS class UrlFetchApp [https://developers.google.com/apps-script/reference/url-fetch/url-fetch-app](https://developers.google.com/apps-script/reference/url-fetch/url-fetch-app)

## まとめ
clasp + TypeScript でローカルで GAS の開発ができる環境を整えたという話。  
いまの自分の用途だとここまでしなくてもよかったんだけど、せっかくなので記録を残しておいたというエントリ。

---
---
<br>
