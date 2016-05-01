.. _installation:

************
インストール
************

.. note::

   インストールやその他の問題に関して、参考になるのは依然として mod_python メーリングリストです。
   表題にsubscribe と書いたメールを mod_python mod_python-request@modpython.org に送るか、 `mod_python メーリングリストのページ <http://mailman.modpython.org/mailman/listinfo/mod_python>`_ に行ってみましょう。

.. _inst-prerequisites:

インストール要件
==================

理想的なケースでは、 OS がパッケージ済みの mod_python を提供しています。
そうでなければ、自分でコンパイルせねばなりません。
このバージョンの mod_python には、下記が必要です:

* Python 2 (2.6 以降) または Python 3 (3.3 以降)
* Apache 2.2 以降。 Apache 2.4 を強く推奨します。

mod_python をコンパイルするには、Apache と Python の両方の include ファイルと、Python のライブラリが、システムにインストールされていなければなりません。
Python と Apache をソースコードからインストールしたなら、必要なものはすべて揃っているはずです。
しかし、パッケージ済みのソフトウェアを使っている場合には、 mod_python のコンパイルに必要な include ファイルやライブラリの入った開発者向けのパッケージを追加で入れる必要があるでしょう。
詳しくは、 OS のドキュメントを調べてみてください。 (ヒント: python-devel や python-dev, apache-devel, apache-dev, httpd-dev といったパッケージを探してみてください)

.. _inst-compiling:

コンパイル
============

.. _inst-configure:

:file:`./configure` を実行する
---------------------------------

:file:`./configure` スクリプトを実行すると、実行環境を解析して、使っているシステムに特化した専用の Makefile を作成します。
autoconf が実行する標準のオプション以外に、 :file:`./configure` には以下のオプションがあります:

.. index::
   single: apxs
   pair: ./configure; --with-apxs

* :program:`apxs` というプログラムを利用できるか調べます。
:program:`apxs` は標準の Apache 配布物に入っていて、コンパイルに必要です。

  :program:`apxs` の場所は、以下のように、 :option:`with-apxs` オプションでマニュアル指定もできます::

     $ ./configure --with-apxs=/usr/local/apache/bin/apxs

  このオプションはいつも指定するよう勧めます。

.. index::
   single: libpython.a
   pair: ./configure; --with-python

* Python のバージョンを調べ、Pythonバイナリ中にコンパイルされている様々なパラメタから、 :file:`libpython` がどこにあるか判定を試みます。
  デフォルトでは :envvar:`PATH` 中に見付かった :program:`python` を使います。

  パス上で最初に見付かる Python バイナリが mod_python の実行にふさわしくないバージョンだったり、他のバージョンのPythonを使いたい場合には、以下の例のように :option:`with-python` を指定して、別の Python の在処を指定できます::

     $ ./configure --with-python=/usr/local/bin/python2.3

.. index::
   pair: ./configure; --with-mutex-dir

* Apache の排他制御ロック用のディレクトリを設定します (APR の選んだ排他制御メカニズムがロックディレクトリを必要とする場合)。

  排他制御ロックを使うのは、 :ref:`mod_python の Sessions <pyapi-sess>` と :ref:`PSP <hand-psp>` (内部で Session を維持しているため) だけです。 mod_python の Session や PSP を使わないのなら、この設定は関係ありません。

  デフォルトの場所は :file:`/tmp` です。ディレクトリは実在せねばならず、Apache プロセスのオーナ権限で書き込めなければなりません。

  :option:`with-mutex-dir` オプションを使って、以下のように指定します::

     $ ./configure --with-mutex-dir=/var/run/mod_python

  排他制御ディレクトリは、実行時に :ref:`dir-other-po` ``mod_python.mutex_directory`` で指定できます。
  :ref:`inst-apacheconfig` を参照してください。

  *New in version 3.3.0*

.. index::
   pair: ./configure; --with-max-locks

* mod_python が確保する排他制御ロックの最大数を指定します。

  排他制御ロックを使うのは、 :ref:`mod_python の Sessions <pyapi-sess>` と :ref:`PSP <hand-psp>` (内部で Session を維持しているため) だけです。 mod_python の Session や PSP を使わないのなら、この設定は関係ありません。

  システムによっては、ロックに使える mutex リソースが限られています。
  この値を増やすと、セッションのロック時のパフォーマンスが向上することがあります。
  デフォルトは 8 です。高いパフォーマンスが必要なら、 32 程度が合理的です。

  :option:`with-max-locks` オプションは以下のように指定します::

     $ ./configure --with-max-locks=32

  ロックの数は、実行時に :ref:`dir-other-po` ``mod_python.mutex_locks`` で指定できます。
  :ref:`inst-apacheconfig` を参照してください。

  *New in version 3.2.0*

.. index::
   single: flex
   pair: ./configure; --with-flex

* :program:`flex` を指定して、バージョンを固定します。
  :program:`flex` が :envvar:`PATH` 上にない場合、 :program:`configure` は失敗します。
  また、正しくないバージョンがあると、 :program:`configure` は警告を出力します。
  :file:`src/psp_parser.c` を作りなおす必要がなければ、警告メッセージは無視してかまいません。

  このパーザは、PSPが使います(:ref:`pyapi-psp` 参照)。
  パーザは C で書かれていて、 :program:`flex` を使っています。
  (リエントラントバージョンの :program:`flex`  2.5.31 以降が必要です)

  パス上で最初に見つかる flex がコンパイルに適していない場合や、使いたくないバージョンである場合には、以下のようにして
  :option:`with-flex` を指定してください::

     $ ./configure --with-flex=/usr/local/bin/flex

  *New in version 3.2.0*

.. _inst-make:

Running :file:`make`
--------------------

.. index::
   single: make

* To start the build process, simply run::

     $ make

.. _inst-installing:

Installing
==========

.. _inst-makeinstall:

.. index::
   pair: make; install

:file:`make install` を実行します。

* インストール作業のこの部分は root で行う必要があります::

      $ sudo make install

  * このコマンドは、ライブラリ (:file:`mod_python.so`) を、Apache のすべてのモジュールが入る :file:`libexec` ディレクトリにコピーします。

  * 次に、Python ライブラリを :file:`site-packages`  にコピーし、コンパイルします。

.. index::
   pair: make targets; install_py_lib
   pair: make targets; install_dso

.. note::

  Pythonライブラリだけ、あるいは DSO (mod_python.so) だけを選択的にインストールしたい場合には (この場合、常にスーパユーザ権限が必要なわけではありません)、 :program:`make` のターゲットに  :option:`install_py_lib` や :option:`install_dso` を使ってください。

.. _inst-apacheconfig:

Apache の設定
==================

.. index::
   pair: LoadModule; apache configuration
   single: mod_python.so

* *LoadModule*

  Apacheの設定ファイルに以下の設定行を追加して、モジュールをロードするよう指示する必要があります。
  設定ファイルは通常、 :file:`httpd.conf` または :file:`apache.conf` という名前です::o

     LoadModule python_module libexec/mod_python.so

  :program:`mod_python.so`  の実際のパスは場合によって異なりますが、 :program:`make install` を行うと、 :program:`mod_python.so`  がどこに置かれたか、そして ``LoadModule`` ディレクトリをどう書けばよいかを報告してくれます。

* :ref:`inst-testing` を読んで、基本的な設定パラメタを確認してください。


.. _inst-testing:

テスト
=======

#. 手元のウェブサイトから見えるディレクトリ、例えば ``htdocs/test`` を作成してください。

#. 下記のApache ディレクティブを、メインのサーバ設定ファイルに追加します::

     <Directory /some/directory/htdocs/test>
         AddHandler mod_python .py
         PythonHandler mptest
         PythonDebug On
     </Directory>

   (上の ``/some/directory`` は、お使いのシステムに合わせて変えてください。通常は Apache の ServerRoot の設定値と同じです)

   この設定は、 :file:`.htaccess` ファイルにも書けます。
   :file:`.htaccess` ファイルは、デフォルトの設定では無効になっているため、このディレクトリに、少なくとも ``FileInfo`` の設定された ``AllowOverride`` を適用しておかねばなりません。

#. 上の設定によって、 ``.py`` で終わる URL は mod_python で処理されるようになりました。
   リクエスト処理が mod_python に渡ると、 mod_python は適切な *python ハンドラ* を探して処理します。
   ここでは、 ``PythonHandler`` ディレクティブは ``mptest`` を Python ハンドラとして指定しています。
   Python ハンドラがどのように定義されているかは、後でわかります。

#. ここまでの作業でメインの設定ファイルに変更を加えているならば、変更内容を有効にするため、Apacheを再起動する必要があります。

#. :file:`htdocs/test` ディレクトリ下の :file:`mptest.py`  ファイルを編集して、以下の内容にします
   (ブラウザからカット&ペーストするときには注意しましょう。インデントが正しくなかったり、文法エラーになったりするかもしれません)::

     from mod_python import apache

     def handler(req):
         req.content_type = 'text/plain'
         req.write("Hello World!")
         return apache.OK

#. :file:`mptest.py` が参照先になるように、ブラウザに URL を指定します。
   ``'Hello World!'`` という文字列を読めるはずです。
   うまく読めなければ、次のトラブルシューティングの節を参照してください。

#. 上で行った設定のため、 ``test`` ディレクトリ中の .py で終わるファイル名なら *何でも* ブラウザに指定できるので注意してください。
   たとえば :file:`/test/foobar.py` に行っても、 :file:`mptest.py` と全く同じ動作になります。
   これは、ハンドラの実装が、今扱っている URL が何であるかに関係なく、同じ処理をしているからです。

#. 全てうまく動作しているなら、 :ref:`チュートリアル <tutorial>` に進みましょう。


.. _inst-trouble:

トラブルシューティング
========================

問題の原因を調べるためにできることが 2, 3 あります:

* エラー出力が出ていれば、注意深く調べます。

* サーバのエラーログファイルを調べます。エラーログファイルには、有用な手がかりが記録されていることがあります。

* Apache を、コマンドラインから単一プロセスモード (single process mode) で起動してみましょう::

     ./httpd -X

  この起動方法は Apache がバックグラウンドで動作するのを防ぐので、有用な情報を得られることがあります。

* mod_python 3.2.0 からは、 mod_python.testhandler で設定ファイルを診断できます。
  :file:`httpd.conf` に下記を追加してください::

     <Location /mpinfo>
       SetHandler mod_python
       PythonHandler mod_python.testhandler
     </Location>

  ブラウザで :file:`/mpinfo`  (例: :file:`http://localhost/mpinfo`) にアクセスすると、設定情報を表示します。
  この情報は、何か問題をメーリングリストに報告するときに役に立ちます。

* `メーリングリスト <http://mailman.modpython.org/mailman/listinfo/mod_python>`_ で質問してみましょう。
  質問するときには、以下のスペックを載せるようにしてください:

  * mod_python のバージョン。
  * OS のタイプ、名前とバージョン。
  * Python のバージョンと、コンパイル時に特別に指定したオプション。
  * Apache のバージョン。
  * Apache 設定ファイルや :file:`.htaccess` 中の関連する部分。
  * Python コードの関連する部分。
