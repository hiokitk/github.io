API GatewayとLambdaを使ったサーバレスSpringアプリケーション
===============================================================
本章では以下のチュートリアル時の作業メモを残す。（第1回～第3回）
https://news.mynavi.jp/techplus/series/AWS/

実施したチュートリアルでは、API GatewayとLambdaを連携し、
Lambda関数に登録したアプリケーション（Spring Cloud Function）の処理を呼び出すというもの。

SpringCloudFunctionとは
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* 関数（Function）を使ってロジックを実行する仕組みで以下の特徴を持つ。
  * SpringFrameworkの機能をフル活用できる、かつ、SpringBootをベースとした設定の簡易化により、最小限のコーディングで構築できる
  * Function型のロジック実行をベースとしたフレームワークとなっており、Function名でAPIとして公開できる（Contollerが不要）
  * アダプタが提供されているため、AWS Lambdaのみならず、「Microsoft Azure」や「Apache OpenWhisk」といった他のサーバレス環境でも実行可能(ポータビリティ性がある)

SpringCloudFunctionの実装メモ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* Spring Cloud Functionでは、基本的に、以下の4種類のクラスを実装しておく必要がある。

.. csv-table::

   "spring-cloud-function-web", "アプリケーションを実装するために必要"
   "spring-cloud-function-adapter-aws", "AWSのアダプタを利用するために必要"
   "aws-lambda-java-events", "AWS LambdaをJavaで動かすために必要"
   "aws-lambda-java-core", "AWS LambdaをJavaで動かすために必要"


.. sourcecode:: xml
   :caption: pom.xml
   :linenos:
   
   <dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-function-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-function-adapter-aws</artifactId>
    </dependency>
    <dependency>
        <groupId>com.amazonaws</groupId>
        <artifactId>aws-lambda-java-events</artifactId>
    </dependency>
    <dependency>
        <groupId>com.amazonaws</groupId>
        <artifactId>aws-lambda-java-core</artifactId>
    </dependency>
   </dependencies>


* Spring Cloud Functionでは、基本的に、以下の4種類のクラスを実装しておく必要がある。

  #. クラウド実行環境でイベントをハンドリングするクラス
       * APIからリクエストを受け取り、イベントとしてLambda関数を実行するクラス
       * ハンドラクラスから、SpringのDIコンテナ構築処理が呼ばれる

  #. Applicaitonの実行・起動・設定を行うクラス
       * アプリケーションをSpring Frameworkで動かすために必要となる起動クラス
       * SpringAppcaliton.run()メソッドを実行するmain()メソッドを持つ実行クラスに、＠SpringBootAppcalitonアノテーションを付与することで、SpringBootによりSpringFramework実行に必要なクラスが自動設定される。

  #. ファンクションコードを実装したクラス
       * ハンドラクラス内で呼び出されるファンクションクラスをjava.util.function.Functionクラスを継承して実装する
       * Functionクラスは、@Beanで登録するか、application.ymlで、 spring.cloud.function.scan.packagesのプロパティとして指定する。

  #. POJOクラス(model)
       * Function内で使用するフィールドを定義する。フィールド名は、jsonで定義する名称とあわせておく。
       * json→modelへのマッピングは、SpringBootApiGatewayRequestHandlerのconvertEventメソッドにて実行される。

.. note:: イベントの種類ごとにハンドラクラスが存在するが、今回はorg.springframework.cloud.function.adapter.aws.SpringBootApiGatewayRequestHandlerを継承する

Lambda関数およびAPIGatewayとの連携
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* 作成したアプリケーションとLambda関数を連携させるために、Lambda関数を以下の条件で作成する。

.. csv-table::

   "ランタイム", "Java8"
   "ハンドラ", "ハンドラクラスのFQDN（com.example.app.ApiGatewayEventHandler）"
   "環境変数", "FUNCTION_NAME = apiGatewayEventFunctionとする（ファンクション名を先頭小文字で）"
   "関数パッケージ", "作成したjarファイルをS3にアップロードしておき、S3のアップロード場所を指定する"

* 作成したLambda関数とAPIGatewayを連携するために、APIGatewayを以下の条件で作成する。

.. csv-table::

   "リソースパス", "/sample"
   "メソッド", "POST"
   "統合タイプ", "Lambda関数"
   "Lambdaプロキシ統合の使用", "チェックする"
   "Lambda関数", "作成したLambda関数名を選択"


.. note:: Lambdaプロキシ統合とは、API Gatewayが受け取ったパラメータやリクエストヘッダなどの情報をlambdaのevent引数にわたすようにする方法のこと。
   lambdaのevent引数から、com.amazonaws.services.lambda.runtime.events.APIGatewayProxyRequestEventとして、
   SpringBootApiGatewayRequestHandlerが、パラメータと受け取りjsonをmodelクラスに変換している。


* 作成後、devステージにデプロイする。エンドポイントのURLに対して、以下コマンドによりリクエストを実行。

  .. sourcecode:: bash
   :linenos:

   $ curl -d '{"test":"TestMessage"}' https://xxxxxxxx.execute-api.ap-northeast-1.amazonaws.com/dev/sample

