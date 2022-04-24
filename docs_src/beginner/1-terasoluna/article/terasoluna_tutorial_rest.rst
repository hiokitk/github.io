TERASOLUNA Server Framework（REST編）
=====================================================
本章ではTERASOLUNA Server Framework（REST編）のチュートリアルを実施時の内容をまとめる。

前提事項
--------
検証環境は以下の通りである。

* Windows10
* JDK1.8.0_201
* Tomcat8.0.23


本ページで掲載している図は `TERASOLUNA Server Framework <http://terasolunaorg.github.io/guideline/current/ja/index.html>`_ から引用している。


RESTとは
---------
「REpresentational State Transfer」の略。以下の特徴を持つ。
 * セッションなどの状態管理を行わない。（ステートレス）
 * リソースの操作はHTTPメソッドによって指定。（HTTPのGETやPOSTメソッドなど）
 * リソースはURL/URIですべてのリソースを一意に識別できる。同一のURLへの呼び出しは常に同じ結果が期待される。
 * 処理結果はXMLやHTML、JSONなどで返される。処理結果をHTTPステータスコードで通知する。


RESTful Web Serviceの開発
-------------------------
Spring MVCでは、RESTful Web Serviceを開発する上で必要となる共通的な機能がデフォルトで組み込まれており、Controllerのアノテーションを指定することで有効にできる。
 #. リクエストBODYに設定されているJSONやXML形式の電文をResourceオブジェクト(JavaBean)に変換し、Controllerクラスのメソッド(REST API)に引き渡す機能
 #. 電文から変換されたResourceオブジェクト(JavaBean)に格納された値に対して入力チェックを実行する機能
 #. Controllerクラスのメソッド(REST API)から返却したResourceオブジェクト(JavaBean)を、JSONやXML形式に変換しレスポンスBODYに設定する機能



実施したチュートリアルの内容
----------------------------
Spring MVCの機能を利用してRESTful Web Serviceを開発する場合、以下の構成となる。実装が必要となる、赤枠の部分を実装した。

 .. image:: http://terasolunaorg.github.io/guideline/current/ja/_images/RESTOverviewApplicationConstitutionOnSpringMVC.png

処理の流れは、以下のとおり。
 #. Spring MVCは、クライアントからのリクエストを受け取り、呼び出すREST API(Controllerのハンドラメソッド)を決定する。
 #. Spring MVCは、HttpMessageConverterを使用して、リクエストBODYに指定されているJSON形式の電文をResourceオブジェクトに変換する。
 #. Spring MVCは、Validatorを使用して、Resourceオブジェクトに格納されて値に対して入力チェックを行う。
 #. Spring MVCは、REST APIを呼び出す。その際に、JSONから変換した入力チェック済みのResourceオブジェクトがREST APIに引き渡される。
 #. REST APIは、Serviceのメソッドを呼び出し、EntityなどのDomainObjectに対する処理を行う。
 #. Serviceのメソッドは、Repositoryのメソッドを呼び出し、EntityなどのDomainObjectのCRUD処理を行う。
 #. Spring MVCは、HttpMessageConverterを使用して、REST APIから返却されたResourceオブジェクトをJSON形式の電文に変換する。
 #. Spring MVCは、JSON形式の電文をレスポンスBODYに設定し、クライアントへレスポンスする。


チュートリアル振り返り
----------------------------
* 「RESPONSE」の「BODY」に実行結果のJSONが表示される。
* @ResponseStatusというアノテーションをつかう。
* MappingJackson2HttpMessageConverterによって、Controllerの引数と返り値で扱うJavaBeanがJSONに変換される。
