サーバーレスのウェブアプリケーションを構築
=====================================================
本章ではAWSのハンズオン時の作業メモを残す。

`実施したハンズオン <https://aws.amazon.com/jp/getting-started/hands-on/build-serverless-web-app-lambda-apigateway-s3-dynamodb-cognito/>`_
では、以下のとおりAWSLambda、APIGateway、DynamoDB、Cognito、Amplifyを使用して、Webアプリケーションを構築した。

.. image:: https://d1.awsstatic.com/diagrams/Serverless_Architecture.5434f715486a0bdd5786cd1c084cd96efa82438f.png

アーキテクチャ概要
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* Amplifyは、継続的デプロイと、HTML/CSS/JavaScript、画像などを含む静的リソースのホストを提供する。

* JavaScriptは、Lambda と API Gateway を使用して構築されたパブリックバックエンド API からデータを送受信します。

* Cognitoは、バックエンド API を保護するためのユーザー管理機能と認証機能を提供する。

* DynamoDBは、APIのLambda 関数によってデータを格納できる永続レイヤーを提供する。


ハンズオンの実施内容
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   #. 静的ウェブホスティング
   #. ユーザ管理
   #. サーバレスバックエンド
   #. RESTful API
   #. リソースの終了

