# Asterisk REST Interface

[Asterisk REST Interface (ARI)](https://wiki.asterisk.org/wiki/pages/viewpage.action?pageId=29395573)

- Asteriskのおける通話の制御はextensions.confという静的な設定ファイルで設定
- extensions.confでダイヤルプランを記述する
- ダイヤルプランはCで実装されていて必要な機能がない時自分でCの実装を行う必要があった
- その後Asterisk Gateway Interface(AGI)とAsterisk Manager Interface(AMI)というAPIが追加された
- AGIはAsteriskダイアルプランとダイヤルプランのチャネルを操作したい外部プログラム間のインターフェースを提供。同期
- AMIはダイヤルプランのチャネルが実行される場所を制御するインターフェースを提供する。チャネルの実行は提供せず、状態や実行場所に関する制御を提供。非同期。
- AGIは同期である欠点があり、AMIもAGIも利用できないプリミティブなAsteriskオブジェクトが多くあり、何よりREST等の現代的な技術が導入されていなかった
- ARIは非同期でAsteriskのプリミティブなオブジェクトを直感的なRESTインターフェースで提供する
- ユーザーが制御しているオブジェクトの状態はWebSocket上でJSONイベントを使って伝達される

extensionに内線の意味がある

## ARIの基礎

3つの構成要素

- Asteriskのリソースを制御するためにクライアントが使用するRESTfulインターフェース
- クライアントへAsteriskのリソースに関するイベントをJSONで伝達するWebSocket
- Asteriskからクライアントへチャンネルの制御を引き渡すStasisダイアルプランアプリケーション

### What is REST?

- ソフトウェアのアーキテクチャスタイル
- ステートレスなクライアント・サーバーモデル
- ARIは厳密なREST APIではない

### What is WebSocket?

- クライアント・サーバー間の双方向通信を可能にするプロトコル
- ARIでは非同期のイベントをAsteriskからクライアントへ受け渡す場合に使用する

### What is Stasis?

- Asteriskのダイヤルプランアプリケーション
- Asteriskがチャンネルを制御するための従来の方法であるダイヤルプランからAPIへチャンネルの制御を引き渡すためにAsteriskが使用する仕組み
- StasisにないチャンネルはARIでは操作できない

## Asterisk ARI examples & ARI Libraries

[Asterisk ARI examples](https://github.com/asterisk/ari-examples)

[ARI Libraries](https://wiki.asterisk.org/wiki/display/AST/ARI+Libraries)

## Recommended Practices

- Webページから直接ARIにアクセスしない
- シンプルな抽象化レイヤーを使用する

# Getting Started with ARI

[Getting Started with ARI](https://wiki.asterisk.org/wiki/display/AST/Getting+Started+with+ARI)

- [static wiki documentation](https://wiki.asterisk.org/wiki/display/AST/Asterisk+18+ARI)
  - 今回の作業対象がAsterisk 18なのでAsterisk 18のリンク
- Asteriskは非同期のイベントをJSONメッセージとしてアプリケーションに送信するのにWebSocket(`/ari/event`)を使用する
- ダイヤルプランとアプリケーションを接続するために`Stasis()`を使用する。外部アプリケーションの名前を指定してStasis()にチャネルを渡す。オプショナルの引数をアプリケーションに渡すことができる

## Example: ARI Hello World!

### 前提

- Asterisk 12以降がインストール・起動されてる
- AsteriskにSIPのソフトフォンORハードフォンがchan_sipかchan_pjshipで登録されている

### 準備

- WebSocketのツールwscatのインストール
- curlのインストール

### Asteriskの設定

1. http.confのAsterisk HTTPサービスを有効にする
2. ari.confのARIユーザー設定する
3. Stasisアプリケーション用のダイヤルプラン内戦番号を作成する

#### freepbxの場合

1は最初から設定されている？

2はGUIの設定>Asterisk REST Interface Users>ユーザーの追加に対応

3.

---

/etc/asteriskの中に上記confファイルがある。GUIからの編集が前提であり、直接は編集しない。

http.confは以下2ファイルに紐付けられて(include)設定されている

- http_additional.conf
- http_custom.conf

ari.confは以下4ファイルに紐付けられて(include)設定されている

- ari_general_additional.conf
- ari_general_custom.conf
- ari_additional.conf
- ari_additional_custom.conf

- Send a channel into Stasis
- And playback "Hello World" to the channel


