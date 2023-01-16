# External Media and ARI

[External Media and ARI](https://wiki.asterisk.org/wiki/display/AST/External+Media+and+ARI)

## イントロダクション

- 外部メディアサーバーとの相互の疎通を可能にする方法
- `/channels/externalMedia` ARIリソースを使って自前のプロキシサーバーを通してメディアのハンドリングができる
- （e.g. クラウド音声認識へ転送）

## シンプルな音声認識アプリケーション

- 自前のARIアプリケーションは、メディアの宛先やフォーマットなどの基本的なパラメータを指定し外部メディアチャネルを作る
- 既存のブリッジにそのチャネルを追加する
- チャネルドライバーがメディアをブリッジから指定された宛先へ転送する
- 自前のアプリケーションは、そのメディア受け取り、音声認識プロバイダーに要件を満たすように渡す
- その結果を自前のアプリケーションは自由に使う（e.g. リアルタイム字幕）
- 外部メディアチャネルはブリッジにメディアを注入することもできる

## 実装

外部メディアチャネルを作成するには`/channels/externalMedia`へPOSTリクエストをする。標準的なARIチャネルオブジェクトが返される。

必須のパラメータは、app、external_host、format

チャネルを使い終わったら作成時のレスポンスであるチャネルオブジェクトをDELETEして終了させる

## メモ

### チャネルとは

[Introduction to ARI and Channels](https://wiki.asterisk.org/wiki/display/AST/Introduction+to+ARI+and+Channels)

- 端末とAsterisk本体の間の通信路で、端末を行き来するすべての情報を含む
- この情報とは、シグナリング（デバイスを呼び出し中の状態にする、通話を切るなど）やメディア（端末から送受信される実際のオーディオやビデオ）などの情報のこと
- Asterisk上でチャネルが作成されるとAsteriskはチャネルにUniqueIDとNameを割り当てる
- チャネルのNameはチャネルタイプと識別子（ほとんど場合PJSIPチャネル）
- ARIがチャネルを操作するハンドルとしてUniqueIDを使う
- 外部エンドポイント（端末）とAsterisk間の外部チャネルとAsterisk内のチャネルであるローカルチャネルがある
- ローカルチャンネルは常にペアで作成される点で特別
- ローカルチャネルペアの間にその間でメディアを送受信する特別なバーチャルエンドポイントが配置される
- 各ローカルチャネルの一端は常にこの仮想バーチャルエンドポイントに接続されていてもう一端を操作する

#### Stasis Applicationにおけるチャネル

- Asterisk上にチャネルが作成されるとダイヤルプランの実行が開始される
- すべてのチャネルがcontext/extension/priorityの組で定義された位置でダイヤルプランに入力される
- ダイヤルプランのそれぞれの組の位置はチャネルが実行すべきAsteriskアプリケーションを定義する
- アプリケーションが完了すると組のpriorityが1増やされてダイヤルプランの次の位置が実行される
- ダイヤルプランが実行すべきことがなくなるか、チャネルに切断（hangup）を指示するか、デバイスが切断するまで繰り返す
- チャネルはStasisダイヤルプランアプリケーションを通してARIに渡される
- Stasisダイヤルプランアプリケーションはダイヤルプランからチャンネルの制御を受け取り、チャネルの制御の準備ができたWebSocketで接続しているARIクラアントに渡される
- この時StasisStartイベントが発火し、チャネルがStasisアプリケーションから離れるよう指示されるか、デバイスが切断するとチャネルはStasisアプリケーションから離れStasisEndイベントが発火する。StasisEndイベントの発火でARIから元のダイヤルプランへチャネルの制御が戻る
- Asteriskにおけるリソースはデフォルトでは自身に関するイベントを接続しているARIアプリケーションには送信しないので次のいずれかの方法で取得する必要がある
  - リソースがStasisダイヤルアプリケーションに入力されたチャネルの場合、暗黙的にサブスクライブされ、チャネルがStatsisダイヤルアプリケーションから離れるときこのサブスクリプションは暗黙的に破棄される
  - チャネルがStasisダイヤルアプリケーション側にある間、そのチャネルはブリッジのような他のリソースとやり取りをすることがあり、チャネルが別のリソースとやり取りを行う間はそのリソースに対してもサブスクライブが行われる。やり取りが終わるとこのサブスクリプションは暗黙的に破棄される
  - ARIアプリケーションはいつでもAsterisk上のリソースをサブスクライブすることができ、そのリソースが存在する間はARIアプリケーションはサブスクリプソンを所有する

#### チャネルとのやりとりの例

次のことをするARIアプリケーションを書きます

- 接続すると既存の全チャネルの名前をプリントする。もし既存チャネルがない場合そのことを教えてくれる
- チャネルがそのStasisアプリケーションに入力されると、そのチャネルに関する仕様情報をすべてプリントする
- チャネルがそのStasisアプリケーションから離れると、そのことをプリントする

##### ダイヤルプランの定義

チャネルをStasisに渡すだけの単純な内線

```
[default]
 
exten => 1000,1,NoOp()
 same =>      n,Answer()
 same =>      n,Stasis(channel-dump)
 same =>      n,Hangup()
```

##### Pythonの例

略

### ブリッジとは

### （チャネルドライバー）とは

### メモの参照

- [オープンソースアプリケーションのアーキテクチャ - Asterisk](https://inzkyk.xyz/aosa/asterisk/)
- [Introduction to ARI and Bridges](https://wiki.asterisk.org/wiki/display/AST/Introduction+to+ARI+and+Bridges)
- [Introduction to ARI and Media Manipulation](https://wiki.asterisk.org/wiki/display/AST/Introduction+to+ARI+and+Media+Manipulation)
- [Asterisk Configuration for ARI](https://wiki.asterisk.org/wiki/display/AST/Asterisk+Configuration+for+ARI)

