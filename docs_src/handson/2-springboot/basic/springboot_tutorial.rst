SpringBootのアプリケーション作成
=====================================================
本章ではSpringBootのチュートリアルの内容について簡単にまとめる
`実施したハンズオン <https://spring.pleiades.io/spring-boot/docs/current/reference/html/getting-started.html#getting-started.first-application>`_
では、POMからSpringBootのアプリケーションを作成、実行、jar化した。


前提事項
^^^^^^^^^^^^^^^^^^^^^^^
検証環境は以下の通りである。

* Windows10
* JDK1.8.0_201
* Apache Maven 3.6.0


ハンズオンの実施内容
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   #. POMの作成
   #. クラスパスの依存関係を追加
   #. サンプルコードの作成
   #. アプリケーションの実行
   #. アプリケーションのjar化
  
1.POMの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* プロジェクトのビルドに必要となるPOM（Project Object Model）ファイルを作成する。
* Mavenではプロジェクトに関する情報をPOMに集約し、POMの情報に基づきプロジェクト全体を管理する。
* 主なタグの説明は、以下のとおり。

.. sourcecode:: xml
   :linenos:

   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>

   <groupId>com.example</groupId>
   <artifactId>myproject</artifactId>
   <version>0.0.1-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.6.7</version>
    </parent>

    <!-- Additional lines to be added here... -->
    
    </project>




.. csv-table::

   "groupId", "プロジェクトを一意に識別する名前。プロジェクトのルートパッケージ名を指定するのが一般的。"
   "artifactId", "プロジェクトの成果物の名前。 作成する JAR や WAR, EAR ファイルなどの名前に使用される。"
   "version", "プログラムの現時点でのバージョンを指定する。生成されたJARファイルの名前にも使われる。"
   "parent", "親プロジェクトを指定する。Spring-bootプロジェクトを構成するため、spring-boot-starter-parent から継承する。"
   "dependencies", "依存ライブラリを指定する。指定したライブラリをリモートリポジトリからダウンロードし、クラスパスに追加される。"


2.クラスパスの依存関係を追加
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* 必要な依存関係を追加するために、pom.xml を編集する。dependenciesセクションにspring-boot-starter-webを追加する。
  * SpringBootのスターターには、プロジェクトを迅速に立ち上げて実行するために必要な多くの依存関係のセットが含まれている。
  * spring-boot-starter-webは、SpringMVCを用いて、Webアプリケーションを構築するためのスターター。埋め込みコンテナーとしてTomcatを使用。

 .. sourcecode:: xml
    :linenos:

    <dependencies>
      <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-web</artifactId>
      </dependency>
    </dependencies>


* 追加した依存関係が有効になっていることを以下コマンドにより確認する。

.. sourcecode:: bash
   :linenos:
   
   $ cd "pomファイルをダウンロードしたディレクトリ"
   $ mvn -f -DoutputFile=mvn_dependency_tree.txt dependency:tree


3.サンプルコードの作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* Mavenは、src/main/java からソースをコンパイルするため、ディレクトリ構造を作成する。
* ディレクトリ作成後、src/main/java/MyApplication.javaという名前で以下ファイルを作成する。
  * SpringApplicationのrunメソッドに実行クラスを引数として指定することで、Springアプリケーションが起動する。
  * @RequestMappingにより、/ パスを持つリクエストはhomeメソッドにマッピングされる。
  * @EnableAutoConfigurationにより、Springが提供している各種プロジェクトのBean定義がなされる。（AutoConfigure）
   
.. sourcecode:: java
   :linenos:

   @RestController
   @EnableAutoConfiguration
    public class MyApplication {

     @RequestMapping("/")
      String home() {
        return "Hello World!";
      }

      public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
      }
    } 


4.アプリケーションの実行
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* 以下コマンドにより作成したアプリケーションを実行する。

.. sourcecode:: bash
   :linenos:

   $ mvn spring-boot:run

* Web ブラウザーを localhost:8080 で開くと、「Hello World!」と表示される。
* アプリケーションを終了するには、ctrl-c を押す。


5.アプリケーションのjar化
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* 実行可能なjarを生成するために、spring-boot-maven-pluginをpom.xmlに追加する。

.. sourcecode:: xml
   :linenos:
   
   <build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
   </build>


* 以下コマンドにより、jarを生成する。

.. sourcecode:: bash
   :linenos:

   $ mvn package


* 以下コマンドにより、生成したjarを実行する。

.. sourcecode:: bash
   :linenos:

   $ java -jar target/demo-0.0.1-SNAPSHOT.jar


