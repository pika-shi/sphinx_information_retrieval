==========
HTTPとHTML
==========

*インターネット* とは、世界中に点在する、家庭や職場などのコンピュータを接続した小規模なイントラネットを、相互に接続した地球規模のコンピュータネットワークです。
そして、私たちが普段利用している *World Wide Web* (単にWWWやWebとも表記される)は、インターネットを介して *ハイパーテキスト* を取得する仕組みのことであり、ここで説明する **HTTP (Hyper Text Transfer Protocol)** とは、その名の通り、Web上でハイパーテキストをやり取りするための規格のことです。

.. note::

   ハイパーテキストとは、相互に参照する機能を持った文書のことで、このような参照のことを *ハイパーリンク* や単に *リンク* と呼びます。

そして、 **HTML (Hyper Text Markup Language)** とは、これもその名の通り、ハイパーテキストを記述するための規格のことです。

.. note::

   いわゆる *Web2.0* 以降のWebでは、HTMLだけでなく *JavaScript* というプログラミング言語を用いたWebページの作成が盛んに行われるようになりましたが、ここではそれらの話題には触れません。

Webは大雑把に分けると、ハイパーテキストを持っているサーバと、ハイパーテキストを閲覧するクライアントから構成されており、このようなシステムを *サーバクライアントモデル* といいます。

.. note::

   Internet ExplorerやFireFoxなどのWebブラウザは最も利用されているクライアントソフトウェアです。

今回は、主にクライアント側からの視点で解説を行い、サーバ側からの話題は他の回で行います。

HTTP
====

PythonでのHTTP通信
------------------

PythonでHTTPリクエストを発行し、HTTP通信でサーバからWebページをダウンロードするには、urllibやurllib2といったモジュールを利用します。ここではurllib2を用いた方法について解説します。

#. Webページの取得

   純粋にHTTP通信でWorld Wide WebからWebページを取ってきたいだけなら、urllib2.urlopenメソッドの返り値をreadします。::

       >>> import urllib2
       >>> url = 'http://sample.com/'
       >>> raw_html = urllib2.urlopen(url).read()

   Webページは文字コードがばらばらなので、文字化けを起こさないように、文字コードを統一する必要があります。日本語文書の文字コード変換にはnkfモジュールが便利です。これはnkfコマンドのPython用のインタフェースで、次のように書くことで文書をutf-8に強制変換することができます。nkfモジュールはPythonにはバンドルされていないので別途インストールが必要です。::

       >>> import nkf
       >>> html = nkf.nkf('-w', raw_html)

#. GET

   GETでクエリを投げるにはURLの末尾にクエリのキーとバリューを次のように設定します。::

       >>> url = 'http://sample.com/?key1=value1&key2=value2'

   このようなクエリ文字列を **URLにエンコードされている(url encoded)** と言い、urllib.urlencodeメソッドを用いると簡単に作成することができます。::

       >>> import urllib, urllib2
       >>> query = urllib.urlencode({'key1': 'value1', 'key2': 'value2'})
       >>> print query
       key1=value1&key2=value2
       >>> url = 'http:/sample.com/&' + query
       >>> raw_html = urllib2.urlopen(url).read()

#. POST

   POSTでリクエストを投げるには、urlopenメソッドの第二引数にurl encodedな文字列を渡します。第二引数がNoneでない場合、urllib2は自動的にPOSTになります。::

       >>> import urllib, urllib2
       >>> query = urllib.urlencode({'key1': 'value1', 'key2': 'value2'})
       >>> url = 'http://sample.com/'
       >>> raw_html = urllib2.urlopen(url, query).read()

#. HTTPリクエストヘッダの変更

   HTTPリクエストのヘッダを変更したい場合は次のように、urllib2.Requestインスタンスを利用します。

   HTTPリクエストヘッダのUser-AgentはHTTP通信する際に用いるソフトウェアまたはハードウェアの情報を指します。クローラを自作した場合は、User-Agentにクローラの名前などを設定するとよいでしょう。

   Acceptヘッダはレスポンスで受け入れ可能なメディアタイプを指定するために使われ、* を用いたワイルドカード記法を用いることもできます。例えば、htmlを要求したい場合は \*/html のように指定します。::

       >>> import urllib2
       >>> url = 'http://sample.com/'
       >>> request = urllib2.Request(url)
       >>> request.add_header('User-Agent', 'My Simple Crawler/1.0')
       >>> request.add_header('Accept', '*/html')
       >>> raw_html = urllib2.urlopen(request).read()

   urllib2.Requestでも第二引数にurl encodedな文字列を渡すことで、POSTリクエストを発行することができます。

   .. warning::

      例えAcceptで \*/html のように指定しても *これを守るかどうかはサーバ次第* であることに注意して下さい。

   .. seealso::

      この他にも様々なヘッダフィールドがRFCで規定されています。

#. HTTPレスポンスヘッダの確認

   HTTPレスポンスを確認するには次のように、urllib2.urlopenメソッドの返り値に対してinfoメソッドを呼びます。::

       >>> import urllib2
       >>> response = urllib2.urlopen('http://sample.com')
       >>> info = response.info()

   特定のヘッダを確認するには、getheaderメソッドを使います。第一引数にヘッダキー、第二引数は第一引数がヘッダにない場合のデフォルトの値を指定します。省略するとNoneが返ってきます。::

       >>> info.getheader('Content-Type')
       'text/html; charset=UTF-8'
       >>> info.getheader('')  # => None
       >>> info.getheader('', 'デフォルトの値')
       'デフォルトの値'

   例えば、取得した文書がHTMLかどうかは次のように(一応)判断することができます。::

       >>> import re
       >>> content_type = info.getheader('Content-Type', '')
       >>> if re.match('\w+/html', content_type):
       ...     raw_html = response.read()
       ...

   .. warning::

      Content-Typeを正しく設定するかどうかは、Webサーバ次第なので、text/html だからと言ってそれが HTML 形式であるという保証は無いことに注意して下さい。

#. urlの構築

   取得したHTMLの中にリンクがあった場合、Aタグのhref属性の値が http:// で始まる完全なURLか、 / で始まる絶対パス指定か、これ以外の相対パス指定かで、リンク先のURLは異なってきます。

   Pythonではurlparse.urljoinを使うと簡単にリンク先URLを構築できます。次の例はbaseから取得したHTML中のhref属性からリンク先URLを構築します。::

       >>> import urlparse
       >>> base = 'http://sample.com/path/to/index.html' # このページが起点
       >>> urlparse.urljoin(base, 'http://yahoo.co.jp/') # yahooへの外部リンク
       'http://yahoo.co.jp/'
       >>> urlparse.urljoin(base, '/index.html') # 内部の絶対パス
       'http://sample.com/index.html'
       >>> urlparse.urljoin(base, 'foo/bar/hoge.html') # 内部の相対パス
       'http://sample.com/path/to/foo/bar/hoge.html'
       >>> urlparse.urljoin(base, '../hoge.html') # 内部の相対パス
       'http://sample.com/path/hoge.html'

HTML
====

PythonでのHTML解析
------------------

.. blockdiag::

   diagram {
       A [label = "Document"];
       B [label = "Element <html>"];
       C [label = "Element <head>"];
       D [label = "Element <body>"];
       E [label = "Element <title>"];
       F [label = "Element <p>"];
       G [label = "Element <p>"];
       H [label = "Element <b>"];
       I [label = "Element <b>"];
       J [label = 'Text "Page title"', color = pink];
       K [label = 'Text "This is para..."', color = pink];
       L [label = 'Text "one"', color = pink];
       M [label = 'Text "."', color = pink];
       N [label = 'Text "This is para..."', color = pink];
       O [label = 'Text "Page title"', color = pink];
       P [label = 'Text "Page title"', color = pink];
       Q [label = 'Attribute id = "firstpara"', color = green];
       R [label = 'Attribute id = "center"', color = green];
       S [label = 'Attribute id = "second..."', color = green];
       T [label = 'Attribute id = "blah"', color = green];

       A -> B -> C, D;
       C -> E -> J;
       D -> F, G;
       F -> Q, R, K, H, M;
       H -> L;
       G -> S, T, N, I, P;
       I -> O;
   }

BeautifulSoupを用いたHTML解析
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

BeautifulSoupはPythonで記述された強力なHMLT/XML解析モジュールで、タグの検索や置換などを簡単に行うことができます。

#. BeautifulSoupの使い方

   BeautifulSoupモジュールにはHTML解析器のBeautifulSoupクラスが定義されています。コンストラクタはHTMLが記述された文字列を受け取ります。::

      from BeautifulSoup import BeautifulSoup
      soup = BeautifulSoup(html)

   ここでは触れませんが、BeautifulSoupにはXML解析器のBeautifulStoneSoupクラスも定義されており、コンストラクタはXML文字列を受け取ります。::

      from BeautifulSoup import BeautifulStoneSoup
      stone_soup = BeautifulStoneSoup(xml)

