TERASOLUNA Server Frameworkの概要
=====================================================
本章ではTERASOLUNA Server Frameworkの概要をまとめる。

本ページで掲載している図は `TERASOLUNA Server Framework <http://terasolunaorg.github.io/guideline/current/ja/index.html>`_ から引用している。



フレームワークの構成
------------------------------------------
* TERASOLUNA Server Frameworkは、SpringFrameworkを中心としたOSSの組み合わせで構成されている。
* SpringFrameWorkのDI、AOPをベースとし、アプリケーション層では、Spring MVCを、インフラ層ではMyBatis3かSpringDataJPAを用いる。

.. image:: http://terasolunaorg.github.io/guideline/current/ja/_images/introduction-software-stack.png




アーキテクチャの概要
------------------------------------------
* TERASOLUNA Server Frameworkでは、アプリケーションを、次の3レイヤに分割している。
* 入力から出力までのデータの流れは、アプリケーション層→ドメイン層→インフラストラクチャ層となる。

.. image:: http://terasolunaorg.github.io/guideline/current/ja/_images/ApplicationLayer.png

アプリケーション層
^^^^^^^^^^^^^^^^^^^
* クライアントとのデータの入出力を制御する層。以下の実装をおこなう。
    * 入力データの妥当性チェック
    * データの入出力を行うオブジェクトへのマッピング
    * クライアントからのリクエストハンドリング
    * リクエスト内容に対応するドメイン層のコンポーネントの呼び出し


ドメイン層
^^^^^^^^^^^^^^^^^^
* ビジネスルールを実行(業務処理を提供)する層。以下の実装をおこなう。
    * Domain Object
    * Domain Objectに対するビジネスルールのチェック
    * Domain Objectに対するビジネスルールの実行
    * Domain Objectに対するCRUD操作(インタフェース実装)


インフラストラクチャ層
^^^^^^^^^^^^^^^^^^^^^^
* ドメイン層(Repositoryインタフェース)の実装を提供する層。以下の実装をおこなう。
   * RepositoryImplの実装(CRUD処理)
   * データベースとEntityのマッピング




アーキテクチャの思想
-----------------------------------------
* ドメイン層は、他の層からは疎であり、アプリケーション層とインフラストラクチャ層に依存してはならない。
* ドメイン層の変更によって、アプリケーション層に変更が生じるのは良いが、アプリケーション層の変更によって、ドメイン層の変更が生じるべきではない。
* そのため、以下の図のとおり、ドメイン層がコアとなり、アプリケーション層、インフラストラクチャ層がそれに依存する形となっている。また、レイヤ間の結合部は、インタフェースとして公開することで、疎結合となっている。

.. image:: http://terasolunaorg.github.io/guideline/current/ja/_images/LayerDependencies.png
