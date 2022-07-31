=====================================================
AWS Step Functionsチュートリアル２
=====================================================
`APIGatewayからStepFunctionの呼び出し <https://docs.aws.amazon.com/ja_jp/step-functions/latest/dg/tutorial-api-gateway.html>`_ に関するチュートリアルを実施した結果を整理する。

チュートリアルの流れ
^^^^^^^^^^^^^^^^^^^^^^^^

1. API Gateway用のIAMロールを作成する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* [信頼されたエンティティタイプ]で、[AWSサービス]のAPI Gatewayを選択
* ロール名に、"APIGatewayToStepFunctions"と入力し、ロールを作成する
* ロール作成後、ポリシーのアタッチで、"AWSStepFunctionsFullAccess"を追加する


2. API Gateway APIを作成する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* REST API ボックスで、[構築]を選択
* [新しいAPIの作成]でAPI名を"StartExecutionAPI"とする
* [アクション]からリソースを作成する。リソース名は、"execution"とする
* [アクション]からメソッドをPOSTで作成する。[統合タイプ]を"AWSサービス"とし、StepFuncitonを選択
* [HTTP メソッド]は"POST"、[アクションの種類]は"アクション名の使用"、[アクション]は"StartExecution"とする

3. API Gateway APIのテストとデプロイ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* メソッドの実行ページで、[テスト]を選択する
* リクエストBodyに、入力データ、実行名、ステートマシンのARNを入れて、テスト実行した
* StepFunctionの入力（statuscode）により、エラーハンドリングするはずだが、失敗している状況


