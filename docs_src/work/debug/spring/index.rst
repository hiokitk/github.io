=====================================================
SpringのDebugメモ
=====================================================
本章ではSpringのチュートリアル実施時に詰まった内容を備忘録として残す。



IntelliJでパッケージ階層がずれる
---------------------------------
発生事象
^^^^^^^^^
以下のエラーが発生。

.. sourcecode:: bash
   :linenos:
   
   org.apache.ibatis.binding.BindingException: Invalid bound statement (not found): com.example.todo.domain.repository.todo.TodoRepository.findAll


原因
^^^^^^
Repositoryのパッケージ名が誤っていた。

対処内容
^^^^^^^^^
`リンク <https://qiita.com/SSM3G/items/261fd2dde3c2b62ce566>`_
の方法を参照し、設定のツリーの外観から、「中間のパッケージを省略」のチェックを外して、階層を確認し修正したところ、解消。


