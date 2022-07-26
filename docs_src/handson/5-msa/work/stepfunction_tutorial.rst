=====================================================
AWS Step Functionsチュートリアル
=====================================================
`StepFunctionによるLambdaのエラーハンドリング <https://aws.amazon.com/jp/getting-started/hands-on/handle-serverless-application-errors-step-functions-lambda/>`_ に関するチュートリアルを実施した結果を整理する。

チュートリアルの流れ
^^^^^^^^^^^^^^^^^^^^^^^^

1. Lambda関数を作成する
^^^^^^^^^^^^^^^^^^^^^^^^
* [名前]は任意。[実行ロール]は[基本的なLambdaアクセス権限で新しいロールを作成]とする。
* [ランタイム] は [Python 3.6] とする。
* コードに以下の内容を入力し、保存する。作成後、ARNをメモしておく。

  .. sourcecode:: python
   
   class TooManyRequestsException(Exception): pass
   class ServerUnavailableException(Exception): pass
   class UnknownException(Exception): pass                
         
   def lambda_handler(event, context):
      statuscode = event["statuscode"]
      if statuscode == "429":
      raise TooManyRequestsException('429 Too Many Requests')
      elif statuscode == "503":
      raise ServerUnavailableException('503 Server Unavailable')
      elif statuscode == "200":
      return '200 OK'
      else:
      raise UnknownException('Unknown error')




2. IAMロールを作成する
^^^^^^^^^^^^^^^^^^^^^^
* [信頼されたエンティティタイプ] からAWSのサービスにチェックし、ユースケースでStepFunctionを選択する。
* [ロール名] は「step_functions_basic_execution」としておく。

3. Step Functionsのステートマシンを作成する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* ステートマシンの作成には、Amazon States Language（ASL）というJSONベースの独自の構造化言語で記述されたテンプレートを使用する。 
* ASLには以下の内容を入力し、保存する。


     .. sourcecode:: json

      {
      "Comment": "An example of using retry and catch to handle API responses",
      "StartAt": "Call API",
      "States": {
       "Call API": {
         "Type": "Task",
         "Resource": "arn:aws:lambda:REGION:ACCOUNT_ID:function:FUNCTION_NAME",
         "Next" : "OK",
         "Comment": "Catch a 429 (Too many requests) API exception, and resubmit the failed request in a rate-limiting fashion.",
         "Retry" : [ {
           "ErrorEquals": [ "TooManyRequestsException" ],
           "IntervalSeconds": 1,
           "MaxAttempts": 2
         } ],
         "Catch": [ 
           {
             "ErrorEquals": ["TooManyRequestsException"],
             "Next": "Wait and Try Later"
           }, {
             "ErrorEquals": ["ServerUnavailableException"],
             "Next": "Server Unavailable"
           }, {
             "ErrorEquals": ["States.ALL"],
             "Next": "Catch All"
           }
         ]
       },
       "Wait and Try Later": {
         "Type": "Wait",
         "Seconds" : 1,
         "Next" : "Change to 200"
       },
       "Server Unavailable": {
         "Type": "Fail",
         "Error":"ServerUnavailable",
         "Cause": "The server is currently unable to handle the request."
       },
       "Catch All": {
         "Type": "Fail",
         "Cause": "Unknown error!",
         "Error": "An error of unknown type occurred"
       },
       "Change to 200": {
         "Type": "Pass",
         "Result": {"statuscode" :"200"} ,
         "Next": "Call API"
       },
       "OK": {
         "Type": "Pass",
         "Result": "The request has succeeded.",
         "End": true
       }
      }
      }


* StepFunctionsマネジメントコンソールの[ステートマシンの作成]から、[コードでワークフローを記述]を選択し、タイプは[標準]を選択する。
* 定義画面上で、ASLのコードを貼り付け、Resourceの部分（7行目）に作成したLambda関数のARNを記載する。ロールは既存のロールを選択し、前の手順で作成したロールを記載する。

4. ステートマシンのテスト
^^^^^^^^^^^^^^^^^^^^^^^^^^
* [実行の開始]から、ステートマシンを実行する。inputの内容を変更し、作成したステートマシンの挙動を確認した。
* 正常系(statuscode=200）、異常系（statuscode=503）、リトライ（statuscode=429）、その他例外（statuscode=999）のパターンで入力値を変更した。
* 実行結果は、ワークフローと出力値として得られる。正常系のケースでは、以下のように表示される。また、[実行イベント履歴]から発生例外などの詳細ログが得られる。

    .. image:: https://d1.awsstatic.com/tmt/handle-serverless-application-errors-step-functions-lambda/04d.0a1f72a7639691b8be32d6335af987ddb08db0a4.png


Workflow Studioを用いたステートマシンの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* StepFunctionsのステートマシン作成画面から「ワークフローを視覚的に設計」を選択する。
* 次の画面へ遷移すると、「ワークフローを設計」画面となり、以下のようにGUIベースでステートマシンの作成が可能となる。

.. image:: https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2021/09/c56bc88b3e31533407116e400fcbb431.png

* Lambda関数を挿入する場合は、[アクション]の[Lambda:Invoke]をドラッグ＆ドロップし、[Functionname]にLambda関数名を入力する。
* エラー処理を実装する場合は、[エラー処理]タブの[キャッチャーの編集]を選択し、[Errors]にエラー名を入力する
* 異常終了させる場合は、[キャッチャーの編集]から[新しい状態の追加]を選択し、[フロー]の[Fail]をドラッグ＆ドロップし、Errorオプション、Causeオプションを入力する。


