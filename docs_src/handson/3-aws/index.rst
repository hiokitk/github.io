AWS
=====================================================
本章ではAWSのハンズオン時の作業メモを残す。

`実施したハンズオン <https://pages.awscloud.com/event_JAPAN_Hands-on-for-Beginners-Serverless-2019_LP.html?trk=aws_introduction_page>`_
では、以下のとおりAWSLambda/APIGateway/AWS-Translate/DynamoDBを組み合わせた翻訳APIを構築した。

.. image:: https://d2908q01vomqb2.cloudfront.net/b3f0c7f6bb763af1be91d9e74eabfeb199dc1f1f/2019/11/07/aws-hands-on-for-beginners-1-2-1024x576.png

今後は以下を着手予定。
https://aws.amazon.com/jp/getting-started/hands-on/build-serverless-web-app-lambda-apigateway-s3-dynamodb-cognito/

サーバレスとは
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* サーバーレスとは、サーバーの構築や保守などの面倒な管理をすることなく、サーバー上でプログラムを実行できる仕組み。

* サーバー“レス”とはいうのは、サーバーの管理が一切不要（サーバー管理レス）になるので、「サーバーレス」と呼ばれている。

* 実際にサーバは存在するが、構築、保守、負荷対策などは、利用者側で意識する必要がなく、開発者はコードを書くことにより集中できる。

* 料金体系は、リクエストベースとなり、実行回数と実行時間（確保したメモリ量）で決まる。



ハンズオンの実施内容
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   #. AWS Lambdaの作成（Python3.7）
   #. AWS Lambda から他のAWSサービス（AWS-Translate）の呼び出し
   #. APIGatewayから呼び出す。
   #. DynamoDBとの連携
  
1.AWS Lambdaの作成（Python3.7）
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* ランタイムは以下から選べる
* メモリ、タイムアウト値は以下の範囲で設定可能
  メモリ：128MB 〜 3,008MB （64MBごと）
  タイムアウト値：最大で900秒
* 実行ロールにポリシーを付与することで様々な操作を許可可能
* APの実行は、テストイベントを作成し、入力データを定義して実行する。

2.AWS Lambda から他のAWSサービス（AWS-Translate）の呼び出し
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* SDK（PythonSDK boto3）を参照し、実装を修正する。
  Boto3とはAWSをPythonから操作するためのライブラリの名称。
  https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/index.html

* 実装を修正するにあたり、まず、必要なライブラリをインポートする。

.. sourcecode:: python3
   :linenos:

   import boto3
   client = boto3.client('translate')

* Request Syntaxを参考に、ライブラリの実行処理を実装する
  Parametersをみて、必須パラメータ（REQUIRED）以外は、不要であれば削除でＯＫ


* Response Syntaxを参考に、リクエスト結果の取得処理を実装する
  Json形式でリクエストは返却されるため、「response.get('フィールド名')」の形式で取得する。


* Lamdaのロールにポリシーをアタッチする。
  別のサービスを呼出す場合（今回だとAWS-Translate）、そのサービスのポリシーが必要とある。

3.APIGatewayから呼び出す。
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* AWSAPIGatewayとはサーバーをプロビジョニング/管理することなく、APIを作成・管理できるマネージドサービス
* REST APIとWebSocketに対応しており、リクエストベースの料金体系となる。（実行回数＋キャッシュメモリ量＋データ転送量）
* 以下の流れでREST APIを実装した。

  3-1. リソースとメソッドタイプの定義
       リソース（URL）×メソッド（GET,POST,DELETE）の定義をする。　※リクエストごとの定義は、2)～4)で設定する。

  3-2. メソッドリクエストの設定
       認証の設定、クエリパラメータ、HTTPヘッダの設定をする。

  3-3. 統合タイプの設定
       バックエンドのサービスを以下から選択する。
       
       [Lambda/HTTP/Mock/AWSサービス/VPCリンク]

  3-4. リクエスト/レスポンス変換の定義
       バックエンドへのInput、バックエンドからのOutputを変換できる
       プロキシ統合を利用することで、変換せずにパススルーできる
       最終的なレスポンスを定義する。（レスポンスコード、HTTPレスポンスヘッダなど）

  3-5. デプロイ
       ステージを作成し、そのステージにのみデプロイできる。

Lambdaプロキシ統合とは
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* API GatewayがクライアントリクエストをバックエンドLambdaの入力eventパラメータにマッピングする機能。
* bodyに設定した値をResponse Bodyとして返してくれる。
* 本設定を有効にする場合、ResponseSyntaxが変わるので、実装を修正する。

https://qiita.com/yuuwatanabe/items/a3bd65e709f20574b6db

