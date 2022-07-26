=====================================================
AWS Step Functionsについて
=====================================================
`ディベロッパーガイド <https://docs.aws.amazon.com/ja_jp/step-functions/latest/dg/welcome.html>`_ など調べてまずはまとめた。

AWS Step Functions とは
^^^^^^^^^^^^^^^^^^^^^^^^
* サーバレスなワークフロー処理を作成することができるサービス。ステートマシンとタスクで構成されている。
* ステートマシンとは、ワークフロー全体の処理のこと。ステートマシンの一つ一つの処理単位をStateと呼ぶ。
* Stateは以下の7種類がある。

  .. csv-table::

   "Task", "単一の処理をおこなう基本的なState。タスクの実行基盤としてLambdaやECS等選択可能。"
   "Pass", "前stateのoutputをそのままパススルーするState。デバック用途で使うことが多い。"
   "Wait", "指定した時間待機するState。"
   "Parallel", "複数のTaskを並列に処理する。Parallel内の処理が完了したら次処理に遷移する。"
   "Choice", "IF文のような条件分岐用のState。前処理のoutputの値により分岐するなどする。"
   "Succeed,Fail", "Choiceとあわせて使用する。Succeed,Failに到達した段階でステートマシンは終了する。"
   "Map", "動的並列処理をおこなうState。inputに配列を受け取り、配列の数だけ処理を繰り返す。"

ユースケース
^^^^^^^^^^^^^^^^^^^^^^^^
* 複数Lambda関数の順次処理、アプリケーションのジョブ管理、大容量ファイルの並列実行などの用途で用いる。
* 具体的には以下のとおり。



  #. データ処理
       * バッチ処理用のAWSBatch、ビッグデータ処理用のAmazonEMR、データ準備用のAWSGlue、データ分析用のAthena、コンピューティング用のAWSLambdaなど他のAWSサービスと連携する。

  #. 機械学習
       * Amazon SageMaker上で、データの前処理、後処理、特徴エンジニアリング、データ検証、モデル評価といった機械学習のワークフローをオーケストレートできる。

  #. マイクロサービスのオーケストレーション
       * 複数のマイクロサービスを組み合わせてワークフローとして管理ができる。
       * 長時間実行されるワークフローは"標準ワークフロー"が適している。
       * 即時対応が必要な、短期間の大量のワークフローは"同期Expressワークフロー"が適している。（ウェブベースまたはモバイルアプリケーションに適用）
       * ただちに応答を必要としない短期間のワークフローの場合、"非同期Expressワークフロー"が適している。

  #. ITおよびセキュリティのオートメーション
       * ソフトウェアのアップグレードとパッチ適用、脆弱性に取り組むためセキュリティ更新プログラムのデプロイ、インフラストラクチャの選択、データの同期、サポートチケットのルーティングなどの操作を管理


同期Express ワークフローとは
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* ワークフローをスタートすると、Step Functionsは、"タスクが完了するまで待機"し、結果を返すというもの。
* サービス例外が発生した場合、Step Functionsは最初から再起動しないため、ロジックがべき等となるようつくる必要がある。
* ワークフローは Amazon API Gateway、AWS Lambda から、または StartSyncExecution API コールを使って呼び出すことができる。

少し手を動かしてみた
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* 以下チュートリアルで、複数のAWS Lambda関数を連携させたサーバーレスワークフローを作成した。
  https://aws.amazon.com/jp/getting-started/hands-on/create-a-serverless-workflow-step-functions-lambda/


自身の考え、次のアクション
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* 1処理で、複数のAPIを呼び出すケースなどにおいては、StepFunctionによるワークフロー化が有効ではないかと思っている。
* APIGatewayやLambdaとの連携方法に関するチュートリアルを実施。
  https://docs.aws.amazon.com/ja_jp/step-functions/latest/dg/tutorial-api-gateway.html
  https://docs.aws.amazon.com/ja_jp/step-functions/latest/dg/tutorial-creating-lambda-state-machine.html


参考情報
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* マイクロサービスのオーケストレーションにStepFunctionの同期Expressワークフローが有効
  https://docs.aws.amazon.com/ja_jp/step-functions/latest/dg/use-cases-orchestration.html

* 同期Expressワークフローの起動方法
  https://docs.aws.amazon.com/ja_jp/step-functions/latest/dg/concepts-express-synchronous.html

* StepFunctionを使用してAPIgatewayを呼ぶ
  https://docs.aws.amazon.com/ja_jp/step-functions/latest/dg/connect-api-gateway.html

* 同期Expressワークフローが最近リリース
  https://www.infoq.com/jp/news/2021/04/synchronous-aws-step-functions/

* メモ
  https://zenn.dev/pirosikick/scraps/460467f55ccc86
