
.. |br| raw:: html

   <br />

.. _directives:
.. Apache Configuration Directives


*******************************
Apache設定ディレクティブ
*******************************

.. _dir-handlers:
.. Request Handlers

リクエストハンドラ
====================

.. _dir-handlers-syn:
.. Python*Handler Directive Syntax

Python*Handlerディレクティブの構文
-------------------------------------

.. index::
   single: Python*Handler Syntax


リクエストハンドラのディレクティブは、全て次のような構文です:

``Python*Handler handler [handler ...] [ | .ext [.ext ...] ]``

*handler* は呼び出し可能オブジェクトで、単一の引数、リクエストオブジェクトをとります。
*.ext* はファイルの拡張子です。

一つの行には複数のハンドラを指定できます。
複数指定すると、ハンドラを左から右へ順に呼びます。
同じハンドラの設定を繰り返して指定でき、同じ結果になります。
すなわち、全てのハンドラが最初から最後まで順次実行されます。

ハンドラ列のうちいずれかが ``apache.OK`` または ``apache.DECLINED`` 以外の値を返すと、それ以降のハンドラの実行を中止します。
ハンドラが ``apache.OK`` や ``apache.DECLINED`` すと何が起きるかは、処理フェイズによって異なります。

mod_python 3.3 以前は、リクエストの処理フェイズに関係なく、ハンドラのいずれかが ``apache.OK`` 以外の値を返すと、その時点でフェイズの処理を中断していました。

ハンドラリストの後ろには、 ``|`` を続け、その後ろにファイル拡張子を複数個指定できます。
この指定を行うと、該当するファイル拡張子に対してのみ、ハンドラが実行されるよう制限します。
この機能は trans フェイズ後に実行されるハンドラにのみ作用します。

*ハンドラ* の構文は、以下の通りです:

``module[::object]``

*module* は完全なモジュール名 (パッケージ名のドット表記も利用できます) か、モジュールコードのファイルの実際のパスです。
モジュールは mod_python のモジュール import 機構、 ``apache.import_module()`` で読み込まれます。
モジュールの import 方法について詳しく知りたければ、この関数のドキュメントを参照してください。

オプションの *object* はモジュール内のオブジェクト名です。
オブジェクトの指定にもドット(.)を含められます。
ドットを含める場合、左から右に名前解決を行ってゆきます。
解決処理中に ``<class>`` 型のオブジェクトに遭遇すると、 mod_python はリクエストオブジェクトを引数にしてクラスのインスタンスを生成しようとします。

オブジェクト名を指定しない場合、ハンドラのディレクティブ名をすべて小文字にして、先頭の ``python`` を除いた名前をデフォルトとして使います。
例えば、 ``PythonAuthenHandler`` に対するデフォルトのオブジェクト名は ``authenhandler`` です。

例::

   PythonAuthzHandler mypackage.mymodule::checkallowed

ハンドラの詳細については「 :ref:`pyapi-handler` 」を参照してください。

.. note::
   ``::`` を使っているのは、パフォーマンス上の理由からです。
   Python は、モジュール内のオブジェクトを使用する場合、まず最初にモジュールを import しておく必要があります。
   区切りを単に ``.`` にしてしまうと、区切った語が、それぞれパッケージ、モジュール、クラスなどのどれにあたるかを順に評価していくプロセスがかなり複雑になってしまいます。
   そこで、(Pythonらしくない) ``::`` を使うことで、モジュールを指定する部分がどこで終わり、モジュール内のオブジェクトの指定がどこから始まるのかを判定するためのオーバヘッド mod_python から省き、パフォーマンスをそこそこ稼いでいます。

.. index::
   pair: phase; order

このドキュメントの各ハンドラは、Apache が各フェイズで呼び出す順番に並んでいます。

.. _dir-handlers-pp:

Python*Handlers と Python のモジュールサーチパス
---------------------------------------------------

.. index::
   pair: pythonpath, Python*Handler

``Python*Handler`` ディレクティブを *ディレクトリセクション* (``<Directory></Directory>`` や ``<DirectoryMatch></DirectoryMatch>`` の中、または ``.htaccess`` ファイルの中) で使うと、そのディレクトリは自動的に Python モジュールサーチパス (``sys.path``) の先頭に付加されます。 *ただし* 、 ``PythonPath`` を明に設定した場合は別です。

``Python*Handler`` ディレクティブを *ロケーションセクション* (``<Location></Location>``  や ``<LocationMatch></LocationMatch>`` の中) で使った場合には、モジュールパスは変更されないので、ハンドラモジュールを読み込むためにパスを設定したい場合などには ``PythonPath`` ディレクティブが必要です。

また、ロケーションセクション中で ``Python*Handlers`` を使うと、 mod_python は、 URI からファイルへのマッピングを行なうリクエスト処理フェイズ (``ap_hook_map_to_storage``) のハンドラを無効にします。
なぜなら、通常は ``<Location>`` とファイルシステムにはリンクがない上に、ファイルシステム上からファイルを探して対応付けるために、不要で実行コストの大きいファイルシステムコールが発生するからです。
この仕様の重要な副作用として、リクエストが mod_python のハンドラが定義された ``<Location>`` にマッチすると、そのリクエストの以降の処理では、 ``<Directory>`` や ``<DirectoryMatch>`` ディレクティブとその内容は無視されるということにも注意してください。

.. _dir-handlers-prrh:

PythonPostReadRequestHandler
----------------------------

.. index::
   single: PythonPostReadRequestHandler

`書式: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Syntax>`_ *Python\*Handler Syntax* |br|
`コンテキスト: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Context>`_ server config, virtual host |br|
`オーバライド: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Override>`_ not None |br|
`モジュール: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Module>`_ mod_python.c |br|


このハンドラは、Apacheがリクエストの内容を読み込み終わった後で、かつ、他のフェイズの処理が行われるよりも前に呼び出されます。
このハンドラは入力ヘッダフィールドに基づいて何らかの判定を行う場合に便利です。

複数のハンドラを指定した場合、いずれかのハンドラが
``apache.OK`` や ``apache.DECLINED`` 以外の値を返すと、それ以降のハンドラはこのフェイズでは実行しません。

.. note::

   このリクエスト処理フェイズでは、URI はまだパス名に変換されていません。
   従って、 ``<Directory>`` や ``<Location>`` 、 ``<File>`` といったディレクティブの中や :file:`.htaccess` ファイルの中でこのディレクティブを指定しても Apache は設定値を使えません。
   このディレクティブを置けるのはメインの設定ファイルのみで、そのコードはメインのインタプリタが実行します。
   また、このフェイズはリクエストのコンテキストタイプ (例えば、要求が Python プログラムを指しているのか、それとも gif ファイルなのか) を特定するよりも前に発生するので、このハンドラで指定した Python のルーチンは、このサーバに対する、 (Pythonプログラム以外も含む) *あらゆる* リクエストに対して呼び出されてしまいます。
   このことは、パフォーマンスが優先課題の場合には充分考慮してください。

.. _dir-handlers-th:

PythonTransHandler
------------------

.. index::
   single: PythonTransHandler

`書式: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Syntax>`_ *Python\*Handler Syntax* |br|
`コンテキスト: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Context>`_ server config, virtual host |br|
`オーバライド: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Override>`_ not None |br|
`モジュール: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Module>`_ mod_python.c |br|

このハンドラは、元のリクエストの URI を、 Apache がデフォルトの規則で (Alias ディレクティブなどの効果によって) 書き換えてしまうより前に、 URI から実際のファイル名を切り出したりするのに使えます。

複数のハンドラを指定した場合、ハンドラのいずれかが ``apache.DECLINED`` 以外の値を返すと、それ以降のハンドラの実行を停止します。

.. note::

   このリクエスト処理フェイズでは、URI はまだパス名に変換されていません。
   従って、 ``<Directory>`` や ``<Location>`` 、 ``<File>`` といったディレクティブの中や、  :file:`.htaccess` ファイルの中で指定しても、 Apache はこのディレクティブを実行できません。
   このディレクティブを置けるのはメインの設定ファイルのみで、
   そのコードはメインのインタプリタが実行します。


.. _dir-handlers-hph:

PythonHeaderParserHandler
-------------------------

.. index::
   single: PythonHeaderParserHandler

`書式: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Syntax>`_ *Python\*Handler Syntax* |br|
`コンテキスト: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Context>`_ server config, virtual host, directory, htaccess |br|
`オーバライド: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Override>`_ not None |br|
`モジュール: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Module>`_ mod_python.c |br|


このハンドラは、リクエスト処理の早期に、モジュールがリクエストヘッダを調べて、何らかの適切な動作を行うチャンスを設けるために呼び出されます。

複数のハンドラを指定した場合、いずれかのハンドラが
``apache.OK`` や ``apache.DECLINED`` 以外の値を返すと、それ以降のハンドラはこのフェイズでは実行しません。


.. _dir-handlers-pih:

PythonInitHandler
------------------

.. index::
   single: PythonInitHandler

`書式: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Syntax>`_ *Python\*Handler Syntax* |br|
`コンテキスト: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Context>`_ server config, virtual host, directory, htaccess |br|
`オーバライド: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Override>`_ not None |br|
`モジュール: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Module>`_ mod_python.c |br|


リクエスト処理フェイズの中で最初に呼び出されるハンドラで、 :file:`.htaccess` とディレクトリの内外で使用できます。

複数のハンドラを指定した場合、いずれかのハンドラが ``apache.OK`` や ``apache.DECLINED`` 以外の値を返すと、それ以降のハンドラはこのフェイズでは実行しません。

このハンドラは、実際には異なる2つのハンドラの別名です。
メイン設定ファイル中でディレクトリタグの外に設定したときは、 ``PostReadRequestHandler`` の別名です。
ディレクトリタグの内側 (``PostReadRequestHandler`` を置けない) に設定したときには、 ``PythonHeaderParserHandler`` の別名です。

\*(このアイデアはmod_perlをもとにしています)*


.. _dir-handlers-ach:

PythonAccessHandler
-------------------

.. index::
   single: PythonAccessHandler

`書式: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Syntax>`_ *Python\*Handler Syntax* |br|
`コンテキスト: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Context>`_ server config, virtual host, directory, htaccess |br|
`オーバライド: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Override>`_ not None |br|
`モジュール: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Module>`_ mod_python.c |br|

このルーチンは、リクエストされたリソースに対し、モジュールでアクセス制限をかけるのに使われます。

複数のハンドラを指定した場合、いずれかのハンドラが ``apache.OK`` や ``apache.DECLINED`` 以外の値を返すと、それ以降のハンドラはこのフェイズでは実行しません。

例えば、このハンドラを使って、 IP アドレスによる制限を行えます。
アクセス不許可を示したければ、このハンドラで ``HTTP_FORBIDDEN`` などを返します。

.. _dir-handlers-auh:

PythonAuthenHandler
-------------------

.. index::
   single: PythonAuthenHandler

`書式: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Syntax>`_ *Python\*Handler Syntax* |br|
`コンテキスト: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Context>`_ server config, virtual host, directory, htaccess |br|
`オーバライド: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Override>`_ not None |br|
`モジュール: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Module>`_ mod_python.c |br|


このハンドラは、認証情報をチェックするときに呼ばれます。
認証情報は、リクエストと同時に送信されてきます。
送られてきた認証情報を、ユーザがデータベースの中にあるかどうか、また [暗号化された] パスワードが、データベース上のパスワードと一致しているかを調べるなどしてチェックしてください。

複数のハンドラを指定した場合、いずれかのハンドラが ``apache.DECLINED`` 以外の値を返すと、それ以降のハンドラはこのフェイズでは実行しません。

ユーザ名を得るには、 ``req.user`` を使ってください。
また、ユーザが入力したパスワードを得るには :meth:`req.get_basic_auth_pw()` 関数を使ってください。

``apache.OK`` を返すと、認証が成功したことを意味します。
``apache.HTTP_UNAUTHORIZED`` を返すと、大抵のブラウザは認証ダイアログ (パスワード入力ダイアログ) を再表示します。
``apache.HTTP_FORBIDDEN`` を返すと、ブラウザはエラーを表示し、認証(パスワード)ダイアログを表示しません。
認証に成功しても、ユーザが特定のURLにアクセスできないような場合には、 ``HTTP_FORBIDDEN`` を使ってください。

認証ハンドラの例は以下のようになります::

   def authenhandler(req):

       pw = req.get_basic_auth_pw()
       user = req.user
       if user == "spam" and pw == "eggs":
           return apache.OK
       else:
           return apache.HTTP_UNAUTHORIZED

.. note::

   :meth:`req.get_basic_auth_pw()` は :meth:`req.user` の値を使う前に呼び出さねばなりません。
   Apacheは :meth:`req.get_basic_auth_pw()` を呼び出すまで、認証情報のデコードを試みません。


.. _dir-handlers-auzh:

PythonAuthzHandler
-------------------

.. index::
   single: PythonAuthzHandler

`書式: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Syntax>`_ *Python\*Handler Syntax* |br|
`コンテキスト: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Context>`_ server config, virtual host, directory, htaccess |br|
`オーバライド: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Override>`_ not None |br|
`モジュール: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Module>`_ mod_python.c |br|


このハンドラは ``AuthenHandler`` の後に実行されます。
このハンドラは、ユーザが特定のリソースにアクセスできるかどうかを調べるときに使います。
とはいえ、その処理を ``AuthenHandler`` 中で完結してしまう場合もよくあります。

複数のハンドラを指定した場合、いずれかのハンドラが ``apache.DECLINED`` 以外の値を返すと、それ以降のハンドラはこのフェイズでは実行しません。

.. _dir-handlers-tph:

PythonTypeHandler
-------------------

.. index::
   single: PythonTypeHandler


`書式: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Syntax>`_ *Python\*Handler Syntax* |br|
`コンテキスト: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Context>`_ server config, virtual host, directory, htaccess |br|
`オーバライド: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Override>`_ not None |br|
`モジュール: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Module>`_ mod_python.c |br|


このルーチンは、様々なドキュメントタイプ情報を決定したり設定したりするために呼び出されます。
ドキュメントタイプ情報とは、 (``r->content_type`` で判る) Content-type や、言語などです。

複数のハンドラを指定した場合、いずれかのハンドラが ``apache.DECLINED`` 以外の値を返すと、それ以降のハンドラはこのフェイズでは実行しません。


.. _dir-handlers-fuh:

PythonFixupHandler
-------------------

.. index::
   single: PythonFixupHandler

`書式: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Syntax>`_ *Python\*Handler Syntax* |br|
`コンテキスト: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Context>`_ server config, virtual host, directory, htaccess |br|
`オーバライド: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Override>`_ not None |br|
`モジュール: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Module>`_ mod_python.c |br|


このハンドラは、ヘッダフィールドの特別な修正などを行なうのに使います。
コンテンツハンドラの直前に呼び出されます。

複数のハンドラを指定した場合、いずれかのハンドラが ``apache.OK`` や ``apache.DECLINED`` 以外の値を返すと、それ以降のハンドラはこのフェイズでは実行しません。

.. _dir-handlers-ph:

PythonHandler
-------------

.. index::
   single: PythonHandler

`書式: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Syntax>`_ *Python\*Handler Syntax* |br|
`コンテキスト: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Context>`_ server config, virtual host, directory, htaccess |br|
`オーバライド: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Override>`_ not None |br|
`モジュール: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Module>`_ mod_python.c |br|

メインのリクエストハンドラです。
多くのアプリケーションではこのハンドラだけを指定することになるでしょう。

複数のハンドラを指定した場合、いずれかのハンドラが ``apache.OK`` や ``apache.DECLINED`` 以外の値を返すと、それ以降のハンドラはこのフェイズでは実行せず、その戻り値をコンテンツハンドラフェーズ全体の戻り値にします。
最終的に戻り値が ``apache.DECLINED`` だった場合、Apache はデフォルトハンドラへのフォールバックを試み、リクエストを静的ファイルの要求として処理しようとします。

.. _dir-handlers-plh:

PythonLogHandler
----------------

.. index::
   single: PythonLogHandler

`書式: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Syntax>`_ *Python\*Handler Syntax* |br|
`コンテキスト: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Context>`_ server config, virtual host, directory, htaccess |br|
`オーバライド: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Override>`_ not None |br|
`モジュール: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Module>`_ mod_python.c |br|


このルーチンはモジュール固有のログ関連の処理を行なうために呼び出されます。

複数のハンドラを指定した場合、いずれかのハンドラが ``apache.OK`` や ``apache.DECLINED`` 以外の値を返すと、それ以降のハンドラはこのフェイズでは実行しません。

.. _dir-handlers-pch:

PythonCleanupHandler
--------------------

.. index::
   single: PythonCleanupHandler


`書式: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Syntax>`_ *Python\*Handler Syntax* |br|
`コンテキスト: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Context>`_ server config, virtual host, directory, htaccess |br|
`オーバライド: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Override>`_ not None |br|
`モジュール: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Module>`_ mod_python.c |br|


一番最後のハンドラで、Apache がリクエストオブジェクトを捨てる直前に呼ばれます。

他のハンドラと違い、戻り値は無視されます。
エラーはエラーログに書きこまれますが、 ``PythonDebug`` が ``On`` になっていても、クライアントには何も送信しません。

このハンドラは ``rec.add_handler()`` 関数の引数には使えません。
動的に後処理 (cleanup) を登録したければ、あらかじめ ``req.register_cleanup()`` を使っておいてください。

一度後処理が始まると、それ以上後処理は登録できません。
そのため、 ``req.register_cleanup()`` はこのハンドラ内では効果がありません。

このディレクティブで登録したハンドラは、 ``req.register_cleanup()`` で
登録した後処理ハンドラよりも *後* に実行されます。

.. _dir-filter:


フィルタ
==========

.. _dir-filter-if:

PythonInputFilter
-----------------

.. index::
   single: PythonInputFilter

`書式: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Syntax>`_ PythonInputFilter handler name |br|
`コンテキスト: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Context>`_ server config |br|
`モジュール: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Module>`_ mod_python.c |br|


*name* という名前で入力フィルタ *handler* を登録します。
*Handler* はモジュール名で、 ``::`` のあとに続けて呼出し可能オブジェクトの名前を指定できます。
呼出し可能オブジェクトの名前を省略した場合、デフォルト値の ``inputfilter`` になります。
慣例で、 *name* に登録するフィルタ名は大抵は全て大文字にします。

handler で参照する *モジュール* は、完全なモジュール名にできます (パッケージのドット表記を使えます)。
また、モジュールのコードが書かれたファイルの実際のパスでもかまいません。
モジュールは mod_python のモジュールインポート機構である :func:`apache.import_module` がロードします。
モジュールをインポートするメカニズムを詳しく知りたければ、 :func:`apache.import_module` のドキュメントを参照してください。

フィルタを有効にするには ``AddInputFilter`` を設定します。

.. _dir-filter-of:

PythonOutputFilter
------------------

.. index::
   single: PythonOutputFilter

`書式: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Syntax>`_ PythonOutputFilter handler name |br|
`コンテキスト: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Context>`_ server config |br|
`モジュール: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Module>`_ mod_python.c |br|

*name* という名前で出力フィルタ *handler* を登録します。
*Handler* はモジュール名で、 ``::`` のあとに続けて呼出し可能オブジェクト名をつけられます。
呼出し可能オブジェクトの名前を省略した場合、デフォルト値の ``outputfilter`` になります。
慣例で *Name* に登録するフィルタ名は大抵は全て大文字にします。

handler で参照する *モジュール* は、完全なモジュール名にできます (パッケージのドット表記を使えます)。
また、モジュールのコードが書かれたファイルの実際のパスでもかまいません。
モジュールは mod_python のモジュールインポート機構である :func:`apache.import_module` がロードします。
モジュールをインポートするメカニズムを詳しく知りたければ、 :func:`apache.import_module` のドキュメントを参照してください。

フィルタを有効にするには ``AddOutputFilter`` を設定します。

.. _dir-conn:

接続ハンドラ
==================

.. _dir-conn-ch:

PythonConnectionHandler
-----------------------

.. index::
   single: PythonConnectionHandler

`書式: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Syntax>`_ PythonConnectionHandler handler |br|
`コンテキスト: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Context>`_ server config |br|
`モジュール: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Module>`_ mod_python.c |br|


接続を接続ハンドラ*handler* で処理するよう指定します。
*Handler* には単一の引数、接続オブジェクトが渡されます。

*Handler*はモジュール名で、
``::`` のあとに続けて呼出し可能オブジェクト名をつけられます。
呼出し可能オブジェクトの名前を省略した場合、デフォルトで ``connectionhandler`` になります。

handler で参照する *モジュール* は、完全なモジュール名にできます (パッケージのドット表記を使えます)。
また、モジュールのコードが書かれたファイルの実際のパスでもかまいません。
モジュールは mod_python のモジュールインポート機構である :func:`apache.import_module` がロードします。
モジュールをインポートするメカニズムを詳しく知りたければ、 :func:`apache.import_module` のドキュメントを参照してください。

.. _dir-other:

その他のディレクティブ
=========================

.. _dir-other-epd:

PythonEnablePdb
---------------

.. index::
   single: PythonEnablePdb

`書式: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Syntax>`_ PythonEnablePdb {On, Off} |br|
`Default: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Default>`_ PythonEnablePdb Off |br|
`コンテキスト: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Context>`_ server config, virtual host, directory, htaccess |br|
`オーバライド: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Override>`_ not None |br|
`モジュール: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Module>`_ mod_python.c |br|


``On`` の場合、 :mod:`mod_python` は ``pdb.runcall()`` 関数を使って、 Pythonデバッガ :mod:`pdb` の下でハンドラを実行します。

:mod:`pdb` は対話的なツールなので、このディレクティブを使う場合には ``-DONE_PROCESS`` オプション付きで httpd を起動してください。
ハンドラコードに到達すると、すぐに pdb のプロンプトが表示され、コードをステップごとに進めて変数を調べられるはずです。

.. _dir-other-pd:

PythonDebug
-----------

.. index::
   single: PythonDebug


`書式: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Syntax>`_ PythonDebug {On, Off} |br|
`Default: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Default>`_ PythonDebug Off |br|
`コンテキスト: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Context>`_ server config, virtual host, directory, htaccess |br|
`オーバライド: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Override>`_ not None |br|
`モジュール: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Module>`_ mod_python.c |br|


通常、Python のエラーが捕捉されずに上がってくると、トレースバック出力はログに送られます。
PythonDebug を On に設定すると、 :exc:`IOError` が出るまでは、トレースバックの出力は (ログの他に) クライアントにも送信されます。
ただし、出力時の :exc:`IOError` の場合にはエラーログだけに送信されます。

このディレクティブは開発時に非常に有用です。
しかし、このディレクティブを使うと、意に反した、場合によっては重大なセキュリティ上の
情報をクライアントに暴露してしまう恐れがあるので、実運用では使わないよう勧めます。

.. _dir-other-pimp:

PythonImport
------------

.. index::
   single: PythonImport





`書式: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Syntax>`_ PythonImport *module* *interpreter_name* |br|
`コンテキスト: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Context>`_ server config |br|
`モジュール: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Module>`_ mod_python.c |br|


``interpreter_name`` に指定した名前を持つインタプリタの下で Python モジュール ``module`` を import するよう、サーバに指示します。
import は子プロセスの初期化時に起こるので、実際には起動した子プロセスあたり一度だけモジュールの import が起こります。

handler で参照する *モジュール* は、完全なモジュール名にできます (パッケージのドット表記を使えます)。
また、モジュールのコードが書かれたファイルの実際のパスでもかまいません。
モジュールは mod_python のモジュールインポート機構である :func:`apache.import_module` がロードします。
モジュールをインポートするメカニズムを詳しく知りたければ、 :func:`apache.import_module` のドキュメントを参照してください。

``PythonImport`` は、例えば、データベース接続の初期化など、時間がかかり、リクエスト処理時に実行するのが望ましくない初期化タスクに便利です。

初期化のコードが失敗することがあり、それによってモジュールのインポートが失敗する場合は、関数名を指定したもう一つの書き方を使ってください:

``PythonImport *module::function* *interpreter_name*``

ここに指定した関数は、モジュールが正しくインポートされた時にのみ実行されます。
関数は引数なしで呼びだされます。

.. note::

   モジュールのインポートが起きるとき、設定は完全に読み込まれてはいないため、 PythonInterpreter を含む他のどのディレクティブも、このディレクティブでインポートするモジュールに影響を及ぼせません。
   この制約があるので、インタプリタ名を明示的に指定しておいて、後のリクエスト処理の仮定で、ここでの処理結果に依存した操作をするときに、名前の一致するインタプリタが使われるようにせねばならないのです。
   どんな名前のインタプリタでリクエストを処理しているかわからないときは、リクエストオブジェクトの :attr:`request.interpreter` メンバを調べてください。

多重インタプリタ (Multiple Interpreters) の節も参照して下さい。

.. _dir-other-ipd:

PythonInterpPerDirectory
------------------------

.. index::
   single: PythonInterpPerDirectory

`書式: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Syntax>`_ PythonInterpPerDirectory {On, Off} |br|
`Default: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Default>`_ PythonInterpPerDirectory Off |br|
`コンテキスト: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Context>`_ server config, virtual host, directory, htaccess |br|
`オーバライド: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Override>`_ not None |br|
`モジュール: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Module>`_ mod_python.c |br|


インタプリタの名前を、サーバ名からではなく、リクエスト中のファイルのディレクトリ名(``req.filename``) からつけるよう、 :mod:`mod_python` に指示します。
その結果、デフォルトポリシの場合と違って、二つのスクリプトが別々のディレクトリに置かれていると、それらが互いに別個のサブインタプリタで実行されます。
デフォルトポリシでは、同じ仮想サーバ上のあるスクリプトは、同じサブリンタプリタで実行されます。

例えば、 :file:`/directory/subdirectory` があると仮定します。 :file:`/directory` には、 ``PythonHandler`` ディレクティブの定義された :file:`.htaccess` ファイルがあり、 :file:`/directory/subdirectory` には :file:`.htaccess` がないとします。
 :file:`/directory`  の下のスクリプトと :file:`/directory/subdirectory` の下のスクリプトは、同じ仮想サーバを介してアクセスされていれば、同じインタプリタ下で実行されます。
``PythonInterpPerDirectory`` が有効な場合は各ディレクトリ毎に別個の二つのインタプリタになります。

.. note::

   URIの変換より前の早い段階のリクエスト処理フェイズ (PostReadRequestHandlerやTransHandler) では、URIがまだ変換されていないため、パスがまだ決まっていません。
   その間は、PythonInterpPerDirectoryがOnであったとしても、ハンドラはメインのインタプリタによって実行されます。
   この動作は期待にそぐわないかもしれませんが、残念ながら回避する方法はありません。

.. seealso::

   :ref:`pyapi-interps`
       for more information


.. _dir-other-ipdv:

PythonInterpPerDirective
------------------------

.. index::
   single: PythonInterpPerDirective


`書式: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Syntax>`_ PythonInterpPerDirective {On, Off} |br|
`Default: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Default>`_ PythonInterpPerDirective Off |br|
`コンテキスト: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Context>`_ server config, virtual host, directory, htaccess |br|
`オーバライド: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Override>`_ not None |br|
`モジュール: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Module>`_ mod_python.c |br|


サブインタプリタの名前をつけるよう :mod:`mod_python` に指示します。
名前は、有効な Python*Handler ディレクティブのあるディレクトリの名前になります。

例えば、 :file:`/directory/subdirectory` があると仮定します。
:file:`/directory` にはPythonHandlerディレクティブの入った :file:`.htaccess` ファイルがあり、  :file:`/directory/subdirectory` には別の PythonHandler の入った :file:`.htaccess` ファイルがあるとします。
デフォルトでは、同じ仮想サーバを介してアクセスされていれば、 :file:`/directory`  の下のスクリプトと  :file:`/directory/subdirectory` の下のスクリプトは同じインタプリタ下で実行されます。
``PythonInterpPerDirective`` が有効な場合、各ディレクティブ毎に別個の二つのインタプリタになります。

.. seealso::

  \seetitle[pyapi-interps.html]{\ref{pyapi-interps} 節、複数のインタプリタ}
           {詳しい情報です}
   :ref:`pyapi-interps`
       for more information

.. _dir-other-pi:

PythonInterpreter
-----------------

.. index::
   single: PythonInterpreter


`書式: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Syntax>`_ PythonInterpreter *name* |br|
`コンテキスト: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Context>`_ server config, virtual host, directory, htaccess |br|
`オーバライド: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Override>`_ not None |br|
`モジュール: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Module>`_ mod_python.c |br|


インタプリタの名前を強制的に *name* にするよう :mod:`mod_python` に指示します。
デフォルトの仕様や、 :ref:`dir-other-ipd`および :ref:`dir-other-ipdv` ディレクティブで設定した名前をオーバライドします。

このディレクティブを使うと、通常なら別々のサブインタプリタ下で行われるスクリプトの実行を、同じサブインタプリタ下で行えます。
DocumentRoot の下で使うと、サーバ全体で一つのサブインタプリタを使うようになります。

.. seealso::

   :ref:`pyapi-interps`
       詳しい説明です。

.. _dir-other-phm:

PythonHandlerModule
-------------------

.. index::
   single: PythonHandlerModule

`書式: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Syntax>`_ PythonHandlerModule *module* |br|
`コンテキスト: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Context>`_ server config, virtual host, directory, htaccess |br|
`オーバライド: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Override>`_ not None |br|
`モジュール: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Module>`_ mod_python.c |br|

PythonHandlerModule は、他の Python*Handler ディレクティブの代わりとして使えます。
このハンドラでモジュールを指定すると、各種ハンドラ関数を探すときのデフォルトの検索対象モジュールとして使われ、このモジュール中に関数があればそれを実行します。

例えば以下のような設定は::

   PythonAuthenHandler mymodule
   PythonHandler mymodule
   PythonLogHandler mymodule


このように単純に書けます。::

   PythonHandlerModule mymodule


.. _dir-other-par:

PythonAutoReload
----------------

.. index::
   single: PythonAutoReload


`書式: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Syntax>`_ PythonAutoReload {On, Off} |br|
`Default: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Default>`_ PythonAutoReload On |br|
`コンテキスト: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Context>`_ server config, virtual host, directory, htaccess |br|
`オーバライド: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Override>`_ not None |br|
`モジュール: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Module>`_ mod_python.c |br|


``Off`` にセットすると、モジュールファイルの更新日時を調べないよう :mod:`mod_python` に指示します。

デフォルトでは、 :mod:`mod_python`  はファイルのタイムタンプをチェックし、以前のインポート、またはリロード時よりモジュールファイルの更新時刻が新しければ、そのモジュールをリロードします。
この仕組みでモジュールを自動的に再 import するので、モジュールを更新するたびにサーバーを再起動させる必要が無くなります。

自動リロードを無効化は、モジュールの変化がないプロダクション環境で便利です。
いくらか処理時間を節約でき、わずかなパフォーマンス向上をもたらすからです。

.. _dir-other-pomz:

PythonOptimize
--------------

.. index::
   single: PythonOptimize

`書式: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Syntax>`_ PythonOptimize {On, Off} |br|
`Default: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Default>`_ PythonOptimize Off |br|
`コンテキスト: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Context>`_ server config |br|
`モジュール: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Module>`_ mod_python.c |br|


Pythonの最適化を有効にします。Pythonの ``-O`` オプションと同じです。

.. _dir-other-po:

PythonOption
------------

.. index::
   single: PythonOption

`書式: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Syntax>`_ PythonOption key [value] |br|
`コンテキスト: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Context>`_ server config, virtual host, directory, htaccess |br|
`オーバライド: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Override>`_ not None |br|
`モジュール: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Module>`_ mod_python.c |br|


キーと値のペアをテーブルに保存して、あとで :meth:`request.get_options` で取り出せるようにします。
設定ファイル ( :file:`httpd.conf` や :file:`.htaccess` など) と、Pythonプログラムの間で情報を受け渡すのに便利です。
値を省略したり空文字 (``""``) にすると、そのキーを設定から除去します。


予約済の PythonOption キーワード
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``PythonOption`` で指定できるキーワードの中には、 mod_python の様々な動作を設定するのに使われているものがあります。 mod_python.\* で始まるキーワードは予約済みで、 mod_python の中で使うことを想定しています。

ユーザは、自分のアドオンモジュールを作る際、独自の名前空間を使って、決してグローバルの名前空間を使わないようにしましょう。

以下の PythonOption は、実際に mod_python で使われています。

| mod_python.mutex_directory
| mod_python.mutex_locks
| mod_python.psp.cache_database_filename
| mod_python.session.session_type
| mod_python.session.cookie_name
| mod_python.session.application_domain
| mod_python.session.application_path
| mod_python.session.database_directory
| mod_python.dbm_session.database_filename
| mod_python.dbm_session.database_directory
| mod_python.file_session.enable_fast_cleanup
| mod_python.file_session.verify_session_timeout
| mod_python.file_session.cleanup_grace_period
| mod_python.file_session.cleanup_time_limit
| mod_python.file_session.database_directory
| mod_python.wsgi.application
| mod_python.wsgi.base_uri

| session *Deprecated in 3.3, use mod_python.session.session_type*
| ApplicationPath *Deprecated in 3.3, use mod_python.session.application_path*
| session_cookie_name *Deprecated in 3.3, use mod_python.session.cookie_name*
| session_directory *Deprecated in 3.3, use mod_python.session.database_directory*
| session_dbm *Deprecated in 3.3, use mod_python.dbm_session.database_filename*
| session_cleanup_time_limit *Deprecated in 3.3, use mod_python.file_session.cleanup_time_limit*
| session_fast_cleanup *Deprecated in 3.3, use mod_python.file_session.enable_fast_cleanup*
| session_grace_period *Deprecated in 3.3, use mod_python.file_session.cleanup_grace_period*
| session_verify_cleanup *Deprecated in 3.3, use mod_python.file_session.cleanup_session_timeout*
| PSPDbmCache *Deprecated in 3.3, use mod_python.psp.cache_database_filename*


.. _dir-other-pp:

PythonPath
----------

.. index::
   single: PythonPath

`書式: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Syntax>`_ PythonPath *path* |br|
`コンテキスト: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Context>`_ server config, virtual host, directory, htaccess |br|
`オーバライド: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Override>`_ not None |br|
`モジュール: <http://httpd.apache.org/docs-2.4/mod/directive-dict.html#Module>`_ mod_python.c |br|


PythonPath ディレクティブは、 PythonPath をセットします。
パスはPython のリスト表記で指定せねばなりません::


   PythonPath "['/usr/local/lib/python2.0', '/usr/local/lib/site_python', '/some/other/place']"

このディレクティブで設定したパスは、既存のパスへの追加ではなく、置き換えになります。
ただし、パスの指定値は ``eval`` で評価されるので、ディレクトリを追加したければ、以下のように指定できます::

   PythonPath "sys.path+['/mydir']"

:mod:`mod_python` は PythonPath ディレクティブに関係する eval の実行回数を最低限にしようと試みます。というのも、 eval は低速で、とりわけ、 :file:`.htaccess` ファイル内で指定すると毎回ヒットするたびに評価されるので、強烈な悪影響を及ぼすことがあるからです。
:mod:`mod_python` はPythonPathディレクティブの引数 (評価する前の状態) を覚えておき、値を評価する前に覚えておいた値と比較して、値が同じならば何も行いません。
そのため、コード内で sys.path を変更している場合、このディレクティブがパスの内容を元に戻す手段になるなどと当てにしてはなりません。

PythonPath ディレクティブを複数回指定しても、効果が足し合わされたりはせず、最後に指定したディレクティブが以前のディレクティブの設定を上書きします。

.. note::

   このディレクティブをセキュリティ対策に使ってはなりません。
   Pythonパスはスクリプトで簡単に操作できるからです。









