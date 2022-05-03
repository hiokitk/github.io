SpringBootのアプリケーション作成(Spring Initializr)
=====================================================
* 本章ではSpringBootのチュートリアルの内容について簡単にまとめる
* `実施したハンズオン <https://spring.pleiades.io/quickstart>`_ では、IntelliJからSpring Initializrによりサンプルアプリケーションを作成・実行した。
* `続いて実施したハンズオン <https://spring.pleiades.io/guides/gs/rest-service/>`_ では、さらにサンプルのRESTful Webサービスを作成・実行した。


前提事項
^^^^^^^^^^^^^^^^^^^^^^^
検証環境は以下の通りである。

* Windows10
* JDK1.8.0_201
* Apache Maven 3.6.0


ハンズオンの実施内容
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   #. プロジェクトの作成
   #. サンプルコードの作成
   #. アプリケーションの実行
   #. サンプルコードの作成（RESTful Webサービス）
   #. アプリケーションの実行（RESTful Webサービス）
  
1.プロジェクトの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* 「ファイル」→「新規」→「プロジェクト」から、ジェネレータにSpringInitializrを選択してプロジェクトを作成する。
* アプリケーション名、プロジェクトの場所、JDKのバージョンを選択し、次へを押下し、依存関係としてSpring Webを選択する。


2.サンプルコードの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* src/main/java/com/example/demo フォルダーの DemoApplication.java ファイルを開き、以下のコードを追加して保存する。
  * @GetMapping("/hello") は、http://localhost:8080/hello に送信されるリクエストに応答する
  * @RequestParamはリクエストでname値を期待しているが、存在しない場合は、デフォルトで"World"が使用される
   
.. sourcecode:: java
   :linenos:

    package com.example.demo;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.RequestParam;
    import org.springframework.web.bind.annotation.RestController;

    @SpringBootApplication
    @RestController
     public class DemoApplication {

      public static void main(String[] args) {
      SpringApplication.run(DemoApplication.class, args);
      }

      @GetMapping("/hello")
      public String hello(@RequestParam(value = "name", defaultValue = "World") String name) {
      return String.format("Hello %s!", name);
      }
     }


3.アプリケーションの実行
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* IDE上でアプリケーションを実行し、http://localhost:8080/hello と入力すると「Hello World!」と表示される。
* URL の末尾に ?name=XXX を追加すると「Hello XXX!」と表示される。


4.サンプルコードの作成（RESTful Webサービス）
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* 以下のリソースクラス（src/main/java/com/example/restservice/Greeting.java ）を作成する。
  * idおよびcontentのフィールド、コンストラクター、アクセサーを持つJavaオブジェクトを定義する。

.. sourcecode:: java
    :linenos:

      package com.example.restservice;

      public class Greeting {
       private final long id;
       private final String content;

       public Greeting(long id, String content) {
          this.id = id;
          this.content = content;
       }
       public long getId() {
	  return id;
       }
       public String getContent() {
	  return content;
       }
     }

* 以下のコントローラクラス（src/main/java/com/example/restservice/GreetingController.java）を作成する。
  * /greetingに対するGETリクエストを処理し、Greetingクラスを生成して返す。
  * オブジェクトのデータは、Springのメッセージコンバータにより自動で、JSON形式に変換され、HTTPレスポンスに書き込まれる。
  * @GetMappingにより、/greetingへのGETリクエストがgreeting()メソッドにマップされる
  * 

.. sourcecode:: java
    :linenos:

     package com.example.restservice;

     import java.util.concurrent.atomic.AtomicLong;
     import org.springframework.web.bind.annotation.GetMapping;
     import org.springframework.web.bind.annotation.RequestParam;
     import org.springframework.web.bind.annotation.RestController;

     @RestController
     public class GreetingController {

	private static final String template = "Hello, %s!";
	private final AtomicLong counter = new AtomicLong();

	@GetMapping("/greeting")
	public Greeting greeting(@RequestParam(value = "name", defaultValue = "World") String name) {
		return new Greeting(counter.incrementAndGet(), String.format(template, name));
	}
     }

5.アプリケーションの実行（RESTful Webサービス）
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* 「http://localhost:8080/greeting」にすると、 JSON形式でのレスポンスが返却される。
* idはリクエスト回数。contentは、nameパラメータの値が返却される（デフォルトは、world）。
* 正常に動作することを確認できたら、以下のコマンドにより、jar化する。

.. sourcecode:: bash
    :linenos:
 $ cd "アプリケーションのrootディレクトリ"
 $ mvn package
 $ java -jar target/rest-service-0.0.1-SNAPSHOT.jar


