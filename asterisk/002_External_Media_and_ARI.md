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

- 端末とAsterisk本体の間の通信路で、端末を行き来するすべての情報を含む
- この情報とは、シグナリング（デバイスを呼び出し中の状態にする、通話を切るなど）やメディア（端末から送受信される実際のオーディオやビデオ）などの情報のこと
- Asterisk上でチャネルが作成されるとAsteriskはチャネルにUniqueIDとNameを割り当てる
- チャネルのNameはチャネルタイプと識別子（ほとんど場合PJSIPチャネル）
- ARIがチャネルを操作するハンドルとしてUniqueIDを使う
- 外部エンドポイント（端末）とAsterisk間の外部チャネルとAsterisk内のチャネルであるローカルチャネルがある
- 

### ブリッジとは

### （チャネルドライバー）とは

### メモの参照

- [オープンソースアプリケーションのアーキテクチャ - Asterisk](https://inzkyk.xyz/aosa/asterisk/)
- [Introduction to ARI and Channels](https://wiki.asterisk.org/wiki/display/AST/Introduction+to+ARI+and+Channels)
- [Introduction to ARI and Bridges](https://wiki.asterisk.org/wiki/display/AST/Introduction+to+ARI+and+Bridges)
- [Introduction to ARI and Media Manipulation](https://wiki.asterisk.org/wiki/display/AST/Introduction+to+ARI+and+Media+Manipulation)
- [Asterisk Configuration for ARI](https://wiki.asterisk.org/wiki/display/AST/Asterisk+Configuration+for+ARI)

