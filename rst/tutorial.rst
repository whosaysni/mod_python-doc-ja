
.. _tutorial:

****************
チュートリアル
****************

*で、どうやったらちゃんと動くんですか？*

この節は、インストールが終わって、mod_python プログラミングを始めてみる人のためのガイドです。
インストールマニュアルではないので注意してください。

この章を読んだら、 :ref:`pythonapi` の節の、すくなくとも冒頭の部分にも目を通しておきましょう。

.. _tut-pub:

\section{publisher ハンドラを使ってみる\label{tut-pub}}

publisher ハンドラを使ってみる
========================================


この節では、詳しいことはさておいて、 mod_python を使い始めてみたい人のために、 publisher ハンドラの概要を簡単に説明します。
mod_python ハンドラの動作の仕組みや、ハンドラとは一体何なのか、といった話題は、チュートリアルの後の節で詳しく説明します。

:ref:`hand-pub` は、 mod_python の標準ハンドラの一つです。
このハンドラを使うには、設定ファイルに以下のように書きます::

   AddHandler mod_python .py
   PythonHandler mod_python.publisher
   PythonDebug On

以下の例は、フィードバックを送信する簡単なフォームです。
このフォームは、名前、電子メールアドレス、そしてコメントをユーザに入力させ、その情報を使ってウェブ管理者にメッセージを送ります。
この簡単なアプリは、二つのファイル: データを集めるためのフォームの「 :file:`form.html` 」と、
フォームのアクションターゲットの「 :file:`form.py` 」からなります。

フォームの HTML ソースを以下に示します::

Here is the html for the form::

   <html>
      フィードバックを入力して送ってください:
      Please provide feedback below:
   <p>                           
   <form action="form.py/email" method="POST">

      Name:    <input type="text" name="name"><br>
      Email:   <input type="text" name="email"><br>
      Comment: <textarea name="comment" rows=4 cols=20></textarea><br>
      <input type="submit">

   </form>
   </html>  

``<form>`` タグの ``action`` 要素は ``form.py/email`` を指しています。
次に、以下のような内容で :file:`form.py` という名前のファイルを作成します::

   # coding: utf-8

   import smtplib
   from email.MIMEText import MIMEText

   WEBMASTER = "webmaster"   # webmaster e-mail
   SMTP_SERVER = "localhost" # your SMTP server

   def email(req, name, email, comment):

       # ユーザが全ての情報を入力したか確かめる
       if not (name and email and comment):
           return u"必要な情報が入力されていません。戻って正しく入力してください。"

       # メッセージテキストを作成する
       msg = u"""\
   コメントを送ります:

   %s

   敬具

   %s

   """ % (email, WEBMASTER, comment, name)

       # 送信する
       mime_msg = MIMEText(msg.encode('utf-8'), 'plain', 'UTF-8')
       mime_msg['From'] = email
       mime_msg['MIME-Version'] = '1.0'
       mime_msg['Subject'] = 'feedback'
       mime_msg['Content-Type'] = 'text/html; charset=utf-8'
       mime_msg['Content-Transfer-Encoding'] = 'quoted-printable'
       conn = smtplib.SMTP(SMTP_SERVER)
       conn.sendmail(email, [WEBMASTER], str(mime_msg))
       conn.quit()

       # ユーザにフィードバックの内容を返す
       s = u"""\
   <html>

   %s さん<br>

               
   コメントありがとうございました。すぐにお返事いたします。

   </html>""" % (name)

       return s

ユーザが「送信 (Submit)」ボタンを押すと、publisher ハンドラはフォームの各フィールドの情報をキーワード引数にして、 :mod:`form` モジュール内の関数 :func:`email` を呼び出します。
その際、リクエストオブジェクトを ``req`` 引数で渡します。

必要がなければ、 ``req`` を渡す必要はありません。
publisher ハンドラは賢いので、関数が受け取れる引数だけを渡します。

データは関数の戻り値を介してブラウザに戻されます。

publisher ハンドラは、 mod_python を使ったプログラミングをとても簡単にしながらも、リクエストオブジェクトにアクセスできるので、 mod_python の力を余すところなく使えます。
同じようなことは、「ネイティブの」mod_python ハンドラでもできます。
例えば、 ``req.headers_out`` でヘッダをカスタマイズしたり、 :exc:`apache.SERVER_ERROR` 例外を送出してエラーを返したり、 :meth:`req.write()` や :meth:`req.read()` を使って、クライアントに対して直接データを読み書きしたりできます。

publisher ハンドラの詳しい情報は :ref:`hand-pub` の節を参照してください。

.. _tut-overview:
.. _Quick Overview of how Apache Handles Requests:

Apache によるリクエスト処理の概要
===================================

Apache は、リクエストを複数の :dfn:`フェイズ` で処理します。
例えば、最初のフェイズではユーザを認証し、次のフェイズではユーザが特定のファイルを閲覧する許可を有しているか調べ、続いて (次のフェイズで) ファイルを読み出してクライアントに送信する、といった具合です。
典型的な静的ファイルのリクエスト処理では、 (1) 要求された URI を実際のファイルの場所に変換する、 (2) ファイルを読み出してクライアントに送信する、 (3) リクエストをログに記録する、という三つのフェイズがあります。
厳密にどのフェイズがどのように処理されるかは、設定によって大きく変わります。

:dfn:`ハンドラ` (handler) は、一つのフェイズを処理する関数です。
ある特定のフェイズの処理に対して、複数のハンドラを利用できることもあり、その場合、 Apache は各ハンドラを順番に呼び出します。
各フェイズに対して、まず、Apache の標準のハンドラを呼び出します (ほとんどは、デフォルトの設定ｈでは極めて基本的な処理しかしないか、あるいは何も行いません)。続いて、 mod_python のような Apache モジュールが提供している追加のハンドラを呼び出します。

mod_python は、Apache で使える全てのハンドラを提供しています。
各々のハンドラは、設定ディレクティブで特に設定しない限り、何も実行しません。
mod_python の設定ディレクティブは ``PythonAuthenHandler`` のように、 ``Python`` で始まって ``Handler`` で終わる名前で、一つのハンドラを一つの Python の関数に対応させます。
つまり、mod_python の主な機能は、みなさんのような開発者が書いた Python の関数と、Apache のハンドラと間を取り持つディスパッチャの働きをすることにあります。

もっともよく使うハンドラは ``PythonHandler`` です。
このハンドラは、実際のコンテンツを提供する際の、リクエストのフェイズを処理します。
このフェイズには名前がないので、 :dfn:`汎用` (generic) ハンドラと呼ばれることがあります。
このハンドラの Apache のデフォルトの動作は、ファイルの読み込みとクライアントへの送信です。
読者のみなさんがアプリを書く場合、たいていはこのハンドラをオーバライドすることになるでしょう。
どんなハンドラを利用できるのか知りたければ、 :ref:`directives` の節を参照してください。

.. _tut-what-it-do:
.. _So what Exactly does Mod-python do?:

mod_python は実際何をやっているのか
=====================================

仮に、以下のように設定しているとしましょう::

   <Directory /mywebdir>
       AddHandler mod_python .py
       PythonHandler myscript
       PythonDebug On
   </Directory>

注意: ここでは、 ``/mywebdir`` は物理的な絶対パスです。

さらに、以下のような内容の Python プログラムが :file:`/mywebdir/myscript.py`
 にあるとします (Windows ユーザは、ファイル名のスラッシュをバックスラッシュに置き換えてください)::

   from mod_python import apache

   def handler(req):

       req.content_type = "text/plain"
       req.write("Hello World!")

       return apache.OK

すると，こんなことが起きます: まず、 ``AddHandler`` ディレクティブは、
Apache に、 :file:`/mywebdir` とそれ以下のディレクトリにある :file:`.py` で終わる全てのファイルへのリクエストを、全て mod_python で処理するよう指示します。
``PythonHandler myscript`` ディレクティブは、 mod_python の汎用ハンドラに `myscript` を使ってリクエストを処理するよう指示します。
``PythonDebug On`` ディレクティブは、 Python のエラーが発生した場合に、エラー出力を (ログへの記録に加えて) クライアントに送信するよう mod_python に指示します。
開発中には、この機能はとても便利です。

さて、リクエストが送られてくると、Apache は mod_python のハンドラを呼び出してリクエスト処理フェイズを開始します。
mod_python のハンドラは、設定の中から、呼びだされたハンドラに関するディレクティブをチェックして、定義されているハンドラを探します (mod_python はディスパッチャのように働くことを思い出してください)。
ここで示した例題では、generic ハンドラ以外で、mod_python がするべきことは何もありません。
generic ハンドラの番になると、mod_python は ``PythonHandler myscript`` ディレクティブがあることに気づき、以下のような処理を行います:

``PythonHandler`` 
``sys.path`` に追加されていなければ追加します。

* ``PythonHandler`` ディレクティブの定義されているディレクトリを、 ``sys.path`` の先頭に (まだ付加していなければ) 付加します。

* ``myscript`` という名前のモジュールを import しようと試みます。
  (``myscript`` が、 ``PythonHandler`` ディレクティブの指定されているディレクトリの直下ではなく、サブディレクトリにある場合、import はうまくいかないので注意してください。
  サブディレクトリが ``sys.path`` に入っていないからです。
  ``PythonHandler subdir.myscript`` のように、パッケージ表記を使えば、この問題を回避できます。)

* ``myscript`` モジュール中から関数 ``handler`` を探します。

* リクエストオブジェクトを渡して ``handler`` を呼び出します (リクエストオブジェクトについては、後で詳しく述べます)。



* この時点で、処理はスクリプトの中に移ります。スクリプトを一行づつ見ていきましょう:

  * ::

       from mod_python import apache

    この行で、Apache へのインタフェースを提供している apache モジュールを
import します。
    ごく少数の例外を除いて、mod_python のプログラムには、だいたいこの行があるはずです。

  .. index::
     single: handler

  * ::

       def handler(req):


    :dfn:`ハンドラ` (handler) 関数の宣言です。
    mod_python は、ディレクティブ名を小文字表記にして、先頭の ``python`` を取り除いた名前をハンドラ名に使うので、 ``PythonHandler`` は ``handler`` という名前になるのです。
    ハンドラ関数は別の名前にもできます。その場合は、ディレクティブで  ``::`` を使って明示します。
    例えば、ハンドラが ``spam`` という関数だったなら、ディレクティブは ``PythonHandler myscript::spam`` になったでしょう。

    ハンドラは :ref:`pyapi-mprequest` という引数を取らねばなりません。
    リクエストオブジェクトとは、例えばクライアントの IP アドレス、ヘッダ、 URI といった、あるリクエストに関する全ての情報の入ったオブジェクトです。
    クライアントへの返信も request オブジェクトを介して行います。
    つまり、「応答オブジェクト (response object)」はありません。

  * ::

       req.content_type = "text/plain"

    この行では、コンテンツタイプを ``text/plain`` に設定しています。
    デフォルトの設定は ``text/html`` ですが、この例のハンドラが生成するのは HTMLではなく、 ``text/plain`` の方がふさわしいからです。
    コンテンツタイプの設定は、必ず ``'req.write'`` を呼ぶ *前に* 行ってください。
    先に ``'req.write'`` を呼び出してしまうと、クライアントにレスポンス HTTP ヘッダが送信されてしまい、後でコンテンツタイプ (や、他のヘッダ) を変更しても、何の効果もありません。

  * ::

       req.write("Hello World!")

    文字列``Hello World!`` をクライアントに送信します。.

  * ::

       return apache.OK

    全ての処理がうまくいき、リクエストの処理を終えたことを Apache に伝えます。
    処理がうまく行かなかった場合には、この行で :const:`apache.HTTP_INTERNAL_SERVER_ERROR` や :const:`apache.HTTP_FORBIDDEN` を返すことになります。
    処理がうまく行かなかった場合、 Apache はエラーをログに記録して、クライアント向けのエラーメッセージを生成します。

.. note::

  大事なのは、ハンドラコードを実行するために、必ずしも URL で :file:`myscript.py` を参照しなくてもよいということです。
  必要なのは、 URL に :file:`.py` が入っていることだけです。
  ``AddHandler mod_python .py`` は、 mod_python に対して、特定のファイルではなく、 (``.py`` という拡張子の) *ファイルタイプ* に対するアクセスを、ハンドラで処理するよう指示しています。
  そのため、 URL でどんなファイル名を指定するかは関係なく、ファイルが存在している必要すらないのです。
  この設定の下では、 ``http://myserver/mywebdir/myscript.py`` も ``http://myserver/mywebdir/montypython.py`` も、全く同じ結果を返します。


.. _tut-more-complicated:
.. _Now something More Complicated - Authentication:

今度はもっと複雑なものを - 認証
==================================

基本的なハンドラの書き方を理解したところで、もう少し複雑なものに取り組んでみましょう。

ディレクトリをパスワードで保護したいとしましょう。ログイン名は ``spam`` 、パスワードは ``eggs`` にします。

まず、認証が必要な場合には、 *認証 (authentication)* ハンドラを呼び出すようApache に伝えておく必要があります。
設定には、 ``PythonAuthenHandler`` を使います。
設定ファイルは以下のようになります::

   <Directory /mywebdir>
       AddHandler mod_python .py
       PythonHandler myscript
       PythonAuthenHandler myscript
       PythonDebug On
   </Directory>

二つのハンドラの指定が、同じスクリプトを指しています。
これでかまいません。というのも、ご存知のように、mod_python は異なるハンドラに対して異なる名前の関数をスクリプトから探し出すからです。

次に、Basic HTTP 認証を使って、有効なユーザだけを許可するよう、Apache に指示
します (かなり基本的な Apache の設定なので、ここでは詳しく説明しません)。
設定ファイルは以下のようになりました::

   <Directory /mywebdir>
      AddHandler mod_python .py
      PythonHandler myscript
      PythonAuthenHandler myscript
      PythonDebug On
      AuthType Basic
      AuthName "Restricted Area"
      require valid-user
   </Directory>

Apache のバージョンによっては、 ``AuthAuthoritative`` や ``AuthBasicAuthoritative`` ディレクティブを ``Off`` にして、 Basic 認証の処理が mod_python のハンドラに回ってくるようにせねばなりません。

さて、今度は :file:`myscript.py` に認証のハンドラを書きましょう。
認証ハンドラは以下のようになります::


   from mod_python import apache

   def authenhandler(req):

       pw = req.get_basic_auth_pw()
       user = req.user

       if user == "spam" and pw == "eggs":
          return apache.OK
       else:
          return apache.HTTP_UNAUTHORIZED

一行づつ見てゆきましょう:

* ::

     def authenhandler(req):

  ハンドラ関数の宣言です。
  上でも説明したように、mod_python は、ディレクティブの名前 (``PythonAuthenHandler``)から先頭の ``Python`` を落とし、小文字にした文字列を関数名にするので、関数名は ``authenhandler`` です。
     
* ::

     pw = req.get_basic_auth_pw()

  これで、パスワードを取得します。
  Basic HTTP 認証は、パスワードを base64 エンコード形式で、ほんのちょっとだけ分かりにくくして伝送します。
  この関数は、パスワードをデコードして、文字列にして返します。
  この関数はユーザ名を取得する前に呼び出さねばなりません。

* ::

     user = req.user
  
  ユーザが入力したユーザ名を取得します。

* ::

     if user == "spam" and pw == "eggs":
         return apache.OK

  ユーザが入力した値を比較し、期待通りの値なら、  :const:`apache.OK` を返して、Apache に処理を先に進めさせます。
  Apache は、このリクエストのフェイズが完了したものとみなし、次のフェイズに進みます。
  (``.py`` ファイルに対するリクエストなら、次は :func:`handler()` を呼び出します。)

* ::

     else:
         return apache.HTTP_UNAUTHORIZED 

  期待通りの値でなければ、Apache に :const:`HTTP_UNAUTHORIZED` を返させます。
  この戻り値を受け取ったブラウザは、通常、ユーザ名とパスワードの入力を促すダイアログボックスを表示します。

.. _tut-404-handler:

.. _Your Own 404 Handler:

404 ハンドラを自作する
========================

場合によっては、 HTTP 404 (:const:`HTTP_NOT_FOUND`) や、HTTP 200 以外の結果を返したいことがあります。
これにはちょっとしたトリックが必要です。
ハンドラが :const:`HTTP_NOT_FOUND` を返すと、 Apache はエラーページをレンダリングして返します。
そのため、自分のハンドラで自作のエラーページを返したいとなると、問題が発生します。

そこで、 ``req.status = apache.HTTP_NOT_FOUND`` をセットしてページをレンダリングし、 ``return(apache.OK)`` を返してください::

   from mod_python import apache

   def handler(req):
      if req.filename[-17:] == 'apache-error.html':
         # エラーを Apache に伝えて、エラーページを出力させる
         return(apache.HTTP_NOT_FOUND)
      if req.filename[-18:] == 'handler-error.html':
         # 自作のエラーページを出力する
         req.status = apache.HTTP_NOT_FOUND
         pagebuffer = '存在しないページか、どこかに移動してしまったページです。'
      else:
         # ファイルコンテンツを生成する
         pagebuffer = open(req.filename, 'r').read()

      # 上の3つのケースのうち後者2つのためのフォールスルー
      req.write(pagebuffer)
      return(apache.OK)

レスポンスハンドラ以外のフェーズでエラーページを生成して返したい時は、 ``apache.OK`` の代わりに、 ``apache.DONE`` を返さねばなりません。
そうしないと、後続のハンドラフェーズが実行されてしまうからです。
``apache.DONE`` は、リクエストの処理を直ちに停止させます。
レスポンスハンドラを複数積み重ねて使っている場合も、同じフェーズに登録されている後続のハンドラを実行させたくない場合は、必要に応じて ``apache.DONE`` を使ってください。
