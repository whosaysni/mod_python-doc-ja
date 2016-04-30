.. _introduction:

************
はじめに
************

.. _performance:

パフォーマンス
================

を使う大きな利点の一つは、従来の CGI に対する大幅な速度向上です。
以下に示すのはかなり大雑把なテストの結果です。
このテストは、 Red Had Linux 7.3 の動作している 1.2 GHz の Pentium マシン上で行いました。
4 種類のスクリプトは `Ab <http://httpd.apache.org/docs-2.0/programs/ab.html>`_ で呼び出し、いずれのスクリプトも標準モジュールの cgi を import しています (通常の Python CGI スクリプトは、みな cgi の import で始めるからです)。
その後、 ``Hello!`` を出力させています。
結果は、並列度 1 で、10000 回のリクエストを処理させたときの値です::

   Standard CGI:               23 requests/s
   Mod_python cgihandler:     385 requests/s
   Mod_python publisher:      476 requests/s
   Mod_python handler:       1203 requests/s


.. _apache_api:

Apache HTTP サーバ API
======================

Apache は、リクエストの処理を、リクエストの読み出し，ヘッダの解析，アクセス権限の
チェックといった *フェイズ* に分割します。
各フェイズは *ハンドラ* (handler) と呼ばれる関数で実装できます。
従来，ハンドラは C で書かかれ Apache モジュールの形にコンパイルされていました。
これに対して， mod_python は Python で Apache ハンドラを書いて，Apache の機能を拡張できます。
Apache のリクエスト処理過程についての詳しい情報は、
`Apache Developer Documentation <http://httpd.apache.org/docs/2.4/developer/>`_ や、
`Mod_python - Integrating Python with Apache <http://www.modpython.org/python10/>`_
を参照してください．

今のところ、 mod_python では、Apache HTTP サーバ API の一部だけしかアクセスできません。
このプロジェクトのゴールは、API を100% カバーすることではありません。
それよりも、 API の最も便利な部分や、API をより「Python的に」使う方法にフォーカスしています。

.. _intro_other:

Other Features
==============

Mod_python also provides a number features that fall in the category
of web development, e.g. a parser for embedding Python in HTML
(:ref:`pyapi-psp`), a handler that maps URL space into modules and
functions (:ref:`hand-pub`), support for session (:ref:`pyapi-sess`)
and cookie (:ref:`pyapi-cookie`) handling.

.. seealso::

   `Apache HTTP Server Developer Documentation <http://httpd.apache.org/docs/2.4/developer/>`_
      for HTTP developer information

   `Mod_python - Integrating Python with Apache <http://www.modpython.org/python10/>`_
      for details on how mod_python interfaces with Apache HTTP Server
