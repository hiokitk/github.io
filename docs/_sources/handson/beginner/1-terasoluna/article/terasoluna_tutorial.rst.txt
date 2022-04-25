TERASOLUNA Server Framework
=====================================================
本章ではTERASOLUNA Server Frameworkのチュートリアルを実施時の内容をまとめる。

前提事項
--------
検証環境は以下の通りである。

* Windows10
* JDK1.8.0_201
* Tomcat8.0.23

本ページで掲載している図は `TERASOLUNA Server Framework <http://terasolunaorg.github.io/guideline/current/ja/index.html>`_ から引用している。



実施したチュートリアルの内容
------------------------------------------
* TODOを管理するアプリケーションを作成した。
  http://terasolunaorg.github.io/guideline/current/ja/Tutorial/TutorialTodo.html

.. image:: http://terasolunaorg.github.io/guideline/current/ja/_images/image001.png


Spirng MVCの処理フロー
------------------------------------------
リクエストを受けてからレスポンスを返すまでのSpringMVCの処理フローを以下の図に示す。

.. image:: http://terasolunaorg.github.io/guideline/current/ja/_images/RequestLifecycle.png

#. DispatcherServletが、リクエストを受け取る。
#. DispatcherServletは、リクエスト処理を行うControllerの選択をHandlerMappingに委譲する。
#. DispatcherServlet は、Controllerのビジネスロジック処理の実行をHandlerAdapterに委譲する。
#. HandlerAdapter は、Controllerのビジネスロジック処理を呼び出す。
#. Controllerは、ビジネスロジックを実行し、処理結果をModelに設定し、ビューの論理名をHandlerAdapterに返却する。
#. DispatcherServletは、ビュー名に対応するViewの解決を、ViewResolverに委譲する。ViewResolverは、ビュー名にマッピングされているViewを返却する。
#. DispatcherServletは、返却されたViewにレンダリング処理を委譲する。
#. Viewは、Modelの持つ情報をレンダリングしてレスポンスを返却する。


チュートリアル振り返り
------------------------------------------
* 実際に手を動かしてみることで、より理解が深まった箇所は以下。
 * コントローラとサービスの実装について
 * データの入出力について
 * PRGパターンについて
  

ControllerとServiceの実装について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* Controller
 * @ModelAttributeによって、戻りをModelに追加している
 * 入力チェックの結果がBindingResultに格納されており、サービス実行前にエラーハンドリングする
 * 業務エラー時の制御のため、try-catchでサービスのメソッドを呼び出し、実行結果に応じて遷移先をreturnする
 
* Service
 * 業務チェックをおこない、NGであれば例外をthrowする
 * 入力値以外のModelの値は、Service側で詰めて、repositoryのメソッドを呼び出す。


データの入出力について
^^^^^^^^^^^^^^^^^^^^^^
* @ModelAttributeによって、FormオブジェクトのパラメータをバインドしてModelオブジェクトに追加される。
* また、このアノテーションが付いたメソッドの戻り値（Modelオブジェクト）は自動でリクエストスコープに設定される。
* そのため、FormオブジェクトやModelオブジェクトもJSPから参照できる。（クラス名の頭文字を小文字にしたものが属性名）
* 別名でModelに追加したい場合は、以下のように実装する。

  .. sourcecode:: java
     :linenos:

     model.addAttribute("todos",todos);

PRGパターンについて
^^^^^^^^^^^^^^^^^^^^
* 二重送信の防止手段の一つ。
* 更新処理を行うリクエスト(POSTメソッド)に対する応答としてリダイレクトを返却し、その後ブラウザから自動的にリクエストされるGETメソッドの応答として遷移先の画面を返却するようにする。
* PRGパターンを適用することで、画面表示後にページの再読み込みを行った場合に発生するリクエストがGETメソッドになるため、更新処理の再実行を防ぐことが出来る。

