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
 * リソースの操作はHTTPメソッドによって指定。(HTTPのPOST/GET/PUT/DELETE）
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
* 処理の実行結果として、HTTPステータスコードと、レスポンスボディにデータがある場合は、JSON形式で表示される。
* RESTAPIを扱う上では、Controller層において、以下のアノテーションを使う。

.. csv-table::

 "@ResponseStatus","クライアントにレスポンスを返す際のステータスコードを設定する。"
 "@RequestBody","POSTリクエストにおいて、ボディからデータを取得し、json形式からResourceオブジェクトに変換する。"
 "@PathVariable","REST形式のURLのパラメータをController側で受け取るために使う。"
 "@GetMapping","GETリクエスト用のアノテーション。指定したリソースを取得する(READ)。"
 "@PostMapping","POSTリクエスト用のアノテーション。指定したリソースを追加する(CREATE)。"
 "@PutMapping","PUTリクエスト用のアノテーション。指定したリソースを更新する(UPDATE)。"
 "@DeleteMapping","DELETEリクエスト用のアノテーション。指定したリソースを削除する(DELETE)。"


* 入出力データの扱いとして、Formオブジェクトの代わりにResourceオブジェクトを使う。beanmapperでModelに変換して、serviceに渡す。そしてModelをResourceに変換する。
* MappingJackson2HttpMessageConverterによって、Controllerの引数と返り値で扱うJavaBeanがJSONに変換される。Controllerの実装としては、処理実行後のModelをResourceに変換したオブジェクトを返却する。ResourceからJSONへの変換はフレームワーク側で実行され、実装で意識する必要ない。
* レスポンスのHTTPステータスコードとしては、以下が返却される。

.. csv-table::

  "項番","HTTPステータスコード","説明","適用条件"
  "(1)","200 OK","リクエストが成功した事を通知するHTTPステータスコード。","リクエストが成功した結果として、レスポンスのエンティティボディに、リクエストに対応するリソースの情報を出力する際に応答する。"
  "(2)","201 Created","新しいリソースを作成した事を通知するHTTPステータスコード。","POSTメソッドを使用して、新しいリソースを作成した際に使用する。レスポンスのLocationヘッダに、作成したリソースのURIを設定する。"
  "(3)","204 No Content","リクエストが成功した事を通知するHTTPステータスコード。","リクエストが成功した結果として、レスポンスのエンティティボディに、リクエストに対応するリソースの情報を出力しない時に応答する。"
  "(4)","400 Bad Request","リクエストの構文やリクエストされた値が間違っている事を通知するHTTPステータスコード。","エンティティボディに指定されたJSONやXMLの形式不備を検出した場合や、JSONやXML又はリクエストパラメータに指定された入力値の不備を検出した場合に応答する。"
  "(5)","404 Not Found","指定されたリソースが存在しない事を通知するHTTPステータスコード。","指定されたURIに対応するリソースがシステム内に存在しない場合に応答する。"
  "(6)","409 Conflict",リクエストされた内容でリソースの状態を変更すると、リソースの状態に矛盾が発生ため処理を中止した事を通知するHTTPステータスコード。","排他エラーが発生した場合や業務エラーを検知した場合に応答する。エンティティボディには矛盾の内容や矛盾を解決するために必要なエラー内容を出力する必要がある。"
  "(7)","405 Method Not Allowed","使用されたHTTPメソッドが、指定されたリソースでサポートしていない事を通知するHTTPステータスコード。","サポートされていないHTTPメソッドが使用された事を検知した場合に応答する。レスポンスのAllowヘッダに、許可されているメソッドの列挙を設定する。"
  "(8)","406 Not Acceptable","指定された形式でリソースの状態を応答する事が出来ないため、リクエストを受理できない事を通知するHTTPステータスコード。","レスポンス形式として、拡張子又はAcceptヘッダで指定された形式をサポートしていない場合に応答する。"
  "(9)","415 Unsupported Media Type","エンティティボディに指定された形式をサポートしていないため、リクエストが受け取れない事を通知するHTTPステータスコード。","リクエスト形式として、Content-Typeヘッダで指定された形式をサポートしていない"
  "(10)","500 Internal Server Error","サーバ内部でエラーが発生した事を通知するHTTPステータスコード。","サーバ内で予期しないエラーが発生した場合や、正常稼働時には発生してはいけない状態を検知した場合に応答する。"
  
  
* REST向けの例外ハンドリングの補助クラスとして、ResponseEntityExceptionHandlerが用意されているため、ResponseEntityExceptionHandlerを継承した例外ハンドリング用のクラスを作成し、レスポンスボディにエラー情報を出力するための実装をおこなう。


