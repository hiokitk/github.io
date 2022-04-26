サーバーレスのウェブアプリケーションを構築
=====================================================
本章ではAWSのハンズオン時の作業メモを残す。

`実施したハンズオン <https://aws.amazon.com/jp/getting-started/hands-on/build-serverless-web-app-lambda-apigateway-s3-dynamodb-cognito/>`_
では、以下のとおりAWSLambda、APIGateway、DynamoDB、Cognito、Amplifyを使用して、Webアプリケーションを構築した。
HTMLで表示された地図の位置を選択してリクエストを送信すると、指定した位置に馬のロゴが走ってくるというもの。

.. image:: https://d1.awsstatic.com/diagrams/Serverless_Architecture.5434f715486a0bdd5786cd1c084cd96efa82438f.png

アーキテクチャ概要
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* Amplifyは、デプロイと、HTML/CSS/JavaScript、画像などの静的リソースのホストを提供する。

* JavaScriptは、API GatewayとLambdaを使用して構築されたパブリックバックエンド API からデータを送受信します。

* Cognitoは、バックエンド API を保護するためのユーザー管理機能と認証機能を提供する。

* DynamoDBは、APIのLambda 関数によってデータを格納できる永続レイヤーを提供する。


ハンズオンの実施内容
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   #. 静的ウェブホスティング
      * Amplifyでは、Hosting機能を用いて、リポジトリを接続してウェブアプリケーションをビルド、デプロイできる。
      * GitHubやAWS CodeCommitと連携できるが、今回は、AWS CodeCommitで作成したリポジトリと接続をした。

   #. ユーザ管理
      * ユーザーのアカウントを管理するため、Amazon Cognitoの「ユーザープール」を用いる。
      * 作成したユーザプールの「プールID」と、認証対象として追加したアプリケーションクライアントの「アプリケーションクライアントID」をメモ。
      * JavaScript上にAmazon Cognitoの接続情報として、「プールID」と「アプリケーションクライアントID」と「リージョン」を反映し、リポジトリにプッシュする。

   #. サーバレスバックエンド
      * DynamoDBのテーブルを作成し、ARNをメモ。
      * Lambda用のロールを作成する。また、作成したロールに、DynamoDBのテーブルへのPutItemができるポリシーを付与する。
      * Lambda関数を作成し、Node.jsをランタイムとする。作成したロールを付与し、既存のコードを指定されたコードに置き換える形で修正する。
      * Lambda関数のテストイベントを作成する。次のAPIGatewayで作成するREST APIを想定するHTTPリクエストをイベント内容に置き換える形で修正する。

   #. RESTful API
      * APIGateway作成し、Lambda関数をRESTful API として公開する。
      * 作成したAPIGatewayのオーソライザを設定する。Cognitoを選択し、既に作成済みのユーザプール名を入力する。
      * ユーザーがサインインした際に、Cognito が返すIDまたはアクセストークンを渡すように、[トークンソース] にAuthorizationと入力する。
      * 新しいリソース、メソッドを作成し、APIGateway CORSおよびLambdaプロキシ統合を有効にし、作成したLambda関数を呼び出す。また、「メソッドをリクエスト」から、Authorizationを選択し、Cognitoユーザプールにチェックをし、APIGatewayリクエスト時にCognitoへ認証されるよう設定する。
      * 新しいステージを作成し、APIGatewayをデプロイする。

API Gateway CORS
^^^^^^^^^^^^^^^^
CORS (Cross-Origin Resource Sharing)とは、ブラウザがオリジン以外のサーバからデータを取得すること。
この設定がなされていないAPIは、ドメインが違うWebページ上のJavascriptから呼び出すこ>とができません。
^
