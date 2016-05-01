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

その他の機能
==============

Mod_python は、 Web 開発の領域の様々な機能を提供しています。
HTML に Python を埋め込んで実行するためのパーザ(:ref:`pyapi-psp`)、URL 空間をモジュールや関数のマップするためのハンドラ  (:ref:`hand-pub`)、セッションのサポート (:ref:`pyapi-sess`)、クッキー操作などです。

.. seealso::

   `Apache HTTP Server Developer Documentation <http://httpd.apache.org/docs/2.4/developer/>`_
      HTTP 開発者向けの情報です。

   `Mod_python - Integrating Python with Apache <http://www.modpython.org/python10/>`_
      mod_python と Apache HTTP Server のインタフェースに関する情報です。
