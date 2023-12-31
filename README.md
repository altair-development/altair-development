## Introduction
ここに搭載されているのは現在プライベートで開発中のタスク管理ソフト「Altair」のリソース一式になります。

学習目的で開発を行っているため、ただ動くものを作るのではなく現在の開発のトレンドやユーザービリティ・保守性をとことん追求することを意識して開発を行ってきました。

そのため使用言語やサーバー構成の変更を何度か繰り返しており、以下のように変遷してきました。

1. CakePHPで初期バージョンを作成する。サーバー構成はAPサーバー（CakePHP）/DBサーバー（MySQL）/Redisサーバー。
2. チーム内チャット機能を実装するためにサーバーサイド言語をNode.js（FW:Express.js）に修正する。サーバー構成はAPサーバー（Node.js）/DBサーバー（MongoDB）/Redisサーバー。
3. 独自JSクラス+jQueryで記述されたjavascriptコードが肥大化、保守性が非常に悪いためフロントエンド側にNext.jsを導入する。これによりNode.jsからビュー（HTML/javascript）が削除され新たにNext.jsサーバーが追加される。サーバー構成はAPIサーバー（Node.js）/WEBサーバー（Next.js）/DBサーバー（MongoDB）/Redisサーバー/WebSocketサーバー（Socket.IO）。

## What's this
Altairはゲーマーに特化したタスク管理ソフトです。

昨今のオンラインゲームでは志を一にするプレイヤー同士でクランと呼ばれるユニットを結成して共に同じゲームをプレイすることがトレンドになっています。

Altairはタスク管理を通じてクランをマネージメントするために作成されました。

[開発途中の画面を確認する](https://altair-dev.net)

したがってユーザーはアカウント作成したあとまず管理対象となるクランを作成し、そのクランに対してプロジェクトを立ち上げてタスクを登録していきます。  
最終的に次のような構造でタスクが管理されることになります。

```
クラン
  - プロジェクト
      - タスク
```
> note
> 
> 2023年7月時点で稼働できているのは下記3画面のみとなります。  
> 
> - サインイン画面
> - サインアップ画面
> - マイページ画面
> 
> サインイン画面では下記ユーザーを既に作成済みですのでemail・パスワードを入力のうえサインインしてください。
> 
> 　Email : testuser01@yahoo.co.jp  
> 　Passward : testuser01
>   
> 上記以外のユーザーを使用する場合、サインアップ画面にて任意のemail・パスワードを入力のうえ送信ボタンを押してください。  
> サインイン・サインアップを実施するとマイページ画面に遷移します。
> 
> **※マイページではクラン名の登録とプロジェクトの登録のみ実施可能な状態です。  
> ※サインイン画面とサインアップ画面は認証機能の確認のために実装したためスタイルは適用されていません。  
> ※PC端末でのみ動作確認済み（モバイル未対応）**

以降で各モジュールごとの概要を記載していきます。  
※各モジュールのインストールと実行方法などは各リポジトリのreadme.mdをご確認ください。

## api
MongoDBサーバーに接続しアカウントやタスク情報の更新を行うAPIサーバーモジュール。フロントエンドからの各種データ更新・読み取りリクエストに対し必要なデータのみ返却するREST API構成。

- MongoDBサーバー/Redisサーバー接続
- Google OAuth認証
- APIキーを使用してWEBサーバーのクライアント認証
- Node.js(Fw:Express.js)

[apiへ移動する](https://github.com/altair-development/api)

## websock
WebSocket通信専用サーバーモジュール。ユーザーはログイン完了後に当該サーバーに直接アクセスし、自身の所属するクランチャンネルにログインしクランに関するデータの更新をサブスクライブする。

- Redisサーバー接続
- Redis Pub/Sub API
- APIサーバーとセッションを共有
- npm socket.ioライブラリ

[websockへ移動する](https://github.com/altair-development/websock)

## websock-monitor
WebSocketサーバーに配置されるWebSocket通信用クライアントモジュール。  
WebSocketサーバーモジュールはHTTPリクエストを処理しないため、代わりにロードバランサーに対するヘルスチェックエンドポイントを実装しlocalhost上のWebSocketサーバーと通信が確立していればステータスコード200を返す。

- npm socket.ioライブラリ

[websock-monitorへ移動する](https://github.com/altair-development/websock-monitor)

## spa
Next.jsで構築したWebサーバーモジュール。

- HTMLレンダリング：CSR方式
- Storybook/Chromatic/Jest/ESLint/Sass/VSCode/Redux

HTMLのレンダリングにSSRを採用しないのにNext.jsの利用を決めた理由としては下記があげられる。

- SSRではスクリプトがサーバーとブラウザの両方で実行されるため処理フローがかなり複雑であると感じた
- awsインスタンス上でレンダリング方式ごと（SSR/SSG/CSR）にCore Web Vitalsの測定を行った結果、ブラウザでレンダリングを行うCSRでも十分に高速にレンダリング可能であると判断したから
- プライベートインスタンス上のAPIサーバーと唯一つながるWEBサーバーを導入したかったから
- 勉強のため

[spaへ移動する](https://github.com/altair-development/spa)

## deploy
Amazon EKSとKubernetesを使用したデプロイメントリソース。  
起動用bashスクリプトによりデプロイを自動化している。

主に下記のawsリソースの展開を行っている。

- APIサーバーインスタンス
- WEBサーバーインスタンス
- WebSocketサーバーインスタンス
- MongoDBサーバーインスタンス
- Redisサーバーインスタンス
- CloudWatch Container Insights
- Application Load Balancer

下記にaws構成図を示す ※一部抜粋

<img width="718" alt="aws構成図" src="https://github.com/altair-development/altair-development/assets/140937480/da63c974-0297-4d47-a206-9d753d41b0be">

[deployへ移動する](https://github.com/altair-development/deploy)

## docker
ローカルにDockerコンテナを展開し開発環境を構築するためのDockerリソース。

docker-compose.ymlを実行することで下記のコンテナが展開される。

- MongoDBレプリカセット
- Redisセンチネル

[dockerへ移動する](https://github.com/altair-development/docker)
