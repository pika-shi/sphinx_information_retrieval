==============
データの永続化
==============

.. contents:: :depth: 3

動機付け
========

例えば、あるWebサイトから取ってきたHTMLファイルを使って実験をしたい時に、実験の度にサーバにアクセスしていたのでは時間が掛かかるだけでなく、過度のクローリングを行うと逮捕されてしまう危険性もあります。そのため、一度ダウンロードしたデータをローカルマシンのファイルシステムに保存し、次回からは保存したファイルを使うようにしなければなりません。また、計算するのに時間のかかるデータがあるなら、計算を一度だけ行い、次回からはそのデータを使いまわすことでプログラムを高速化することもできます。

標準入出力とファイル操作
========================

シェル
------

UNIX環境でプログラミングをする際に最低限知っておきたいシェルの使い方について解説します。ちなみに、ここでは bash での使い方を解説します。

.. seealso::

   詳しくはシェルを専門で扱っているページを参照して下さい。

#. コマンドの結果をファイルに書きだす。

   コマンドの出力には標準出力と標準エラー出力の2種類があり、通常は両方が端末に表示されます。これらをファイルに保存したい場合、次のようにすることで、file1に標準出力、file2に標準エラー出力が書きだされます::

       % command 1> file1 2> file2
       % command > file1 2> file2  # 1は省略できる

   片方だけを保存することもできます。以下の場合、標準エラー出力は端末に表示されます::

       % command > file1

   UNIX環境には **/dev/null** というスペシャルファイルがあり、そこに書きこまれたデータは全て破棄されます。なので、このファイルを利用することで、標準出力のみを画面に表示し、標準エラー出力は廃棄する、というようなことも簡単にできます::

       % command 2> /dev/null

#. コマンドの結果をファイルに追記する。

   ファイルの末尾に追加したい場合は >> を使う::

       % command >> file1 2>> file2

#. 標準出力を他のコマンドに渡す。

   パイプ | を使う。以下の場合、command1 の標準出力を標準入力として受け取った command2 の標準出力を標準入力として受け取った command3 の標準出力を画面に表示されます。::

      % command1 | command2 | command3

   パイプラインを上手に使えると非常に楽ができます。

#. ファイルの内容を標準入力でコマンドに渡す。

   標準入力でファイルを渡すには < を使うか、 *cat* コマンドと | を組み合わせて使います::

      % command < file
      % cat file | command # 同じ

Pythonでの標準入出力
--------------------

sysモジュールにIO関連の機能があります。

* sys.stdin : 標準入力
* sys.stdout : 標準出力
* sys.stderr : 標準エラー出力

#. 標準出力と標準エラー出力

   標準出力は *sys.stdout.write()* か *print* 文を使い、標準エラー出力は *sys.stdout.write()* を使います::

      # stdio1.py
      import sys
      sys.stdout.write('foo\n')  # print 'foo' でも可
      sys.stderr.write('bar\n')
      
   このコマンドを実行すると以下のようになります::

      % python stdio1.py 1> stdout 2> stderr
      % cat stdout
      foo
      % cat stderr
      bar

#. 標準入力

   標準入力を一行ずつ受け取って処理するには以下のようにします::

      for line in iter(sys.stdin.readline, ''):
          line = line.strip()  # 末尾の改行文字を取り除く
          do_something(line)

Pythonでのファイル入出力
------------------------

#. ファイルへの出力

   *open* 組み込み関数を使います。第1引数にファイルへのパスを、第2引数に開き方を指定します。'w' で書き込み、'a' で追記、'b' でバイナリモードなどがあります。fileを開いたら、最後に必ず *close()* しなければなりません。以下のように *try-finally* 文を使うとよいでしょう::

      try:
          f = open('tmp', 'w')
          f.write('hoge\n')
      finally:
          f.close()

   以下のように *with* 文を使うと、自動的に *close()* されます::

      with open('tmp', 'w') as f:
          f.write('hoge\n')

#. ファイルからの入力

   *open* 組み込み関数の第2引数に 'r' を指定すると読み込みモードでファイルを開くことができます。また、第2引数を省略した場合も、読み込みモードになります::

      try:
         f = open('tmp')
         for line in f:
             do_something(line)
      finally:
         f.close()

   こちらも *with* 文を使えます::

      with open('tmp') as f:
          for line in f:
              do_something(line)

   *linecache* モジュールを使うと、任意に行を簡単に読み出すことができます::

      import linecache
      line = linecache.getline('tmp', 2)

   *linecache* はファイルをキャッシュしていくので、読み込んだファイルが必要なくなったら *clearcache()* を使ってキャッシュをクリアしましょう::

      linecache.clearcache()

.. seealso::

   Python公式ドキュメント
      `10.9. linecache - テキストラインにランダムアクセスする <http://www.python.jp/doc/nightly/library/linecache.html>`_

CSVファイルの利用
-----------------

CSV(Comma Separated Values、カンマ区切り値列)と呼ばれる形式は、スプレッドシートやデータベース間でのデータのインポートやエクスポートにおける最も一般的な形式です。CSV形式で作成されたファイルは、Excelでも簡単に処理することができるので、実験結果をCSV形式のファイルで保存するのはよいプラクティスと言えます。

CSVはその名の通り、通常はカンマで値を区切りますが、その他の文字（例えばタブ *\t* やコロン *:* )で区切ることもできます。このような区切り文字を **delimiter** と呼びます。

Pythonでは *csv* モジュールが提供されています。以下で簡単な使用例を紹介します。::

   >>> data = do_something()
   >>> data
   {'kanto': ('tokyo', 'kanagawa'), 'kansai': ('kyoto', 'osaka')}
   >>> import csv, sys
   >>> writer = csv.writer(open('result.csv', 'w'))
   >>> for key, values in data.iteritems():
   ...     writer.writerow([key] + values)
   ...

これで作られるのは以下のようなファイルです::

   % cat result.csv
   kanto,tokyo,kanagawa
   kansai,kyoto,osaka

これを読み出すには以下のようにします::

   >>> import csv
   >>> reader = csv.reader(open('result.csv'))
   >>> data = {}
   >>> for row in reader:
   ...     data[row[0]] = tuple(row[1:])
   ...
   >>> data
   {'kansai': ['kyoto', 'osaka'], 'kanto': ['tokyo', 'kanagawa']}

.. seealso::

   Python公式ドキュメント
      `13.1. csv - CSV ファイルの読み書き <http://www.python.jp/doc/nightly/library/csv.html>`_

オブジェクトの永続化
--------------------

CSV は Excel などのソフトウェアでデータを共有するのに便利ですが、Pythonオブジェクトをそのまま保存して再利用したくなる場合も多々あります。そのような場合、Pythonでは *pickle* モジュール及び *shelve* モジュールを使います。 *pickle* モジュールより高速な *cPickle* が提供されている環境では、 *cPickle* を使いましょう。

*pickle* はPythonオブジェクトをそのまま *塩漬け* にし、後で再利用するためのモジュールです。オブジェクトを保存するために変換することを **シリアライズ** 、シリアライズされたデータからオブジェクトを再構築することを **デシリアライズ** と言います。シリアライズされたデータは、既に人間には解読不能な状態なので、圧縮することでデータIOの時間を短縮するようにするとよいでしょう。以下では *gzip* モジュールを使って、シリアライズされたデータを圧縮してから保存し、逆にデータをロードする際は、まず *gzip* で解凍してから *pickle* を使ってデシリアライズしています::

    >>> import cPickle
    >>> import gzip
    >>> def save(filename, obj):
    ...     try:
    ...         f = gzip.open(filname, 'wb')
    ...         cPickle.dump(obj, f, 2)
    ...     finally:
    ...         f.close()
    ...
    >>> def load(filename):
    ...     try:
    ...         f = gzip.open(filename, 'rb')
    ...         return cPickle.load(f)
    ...     finally:
    ...         f.close()
    ...
    >>> save('tmp.gz', 'hoge')
    >>> hoge = load('tmp.gz')
    >>> print hoge
    hoge

*shelve* モジュールを使うと、ファイルを辞書オブジェクトのように使うことができます。詳しくはドキュメントを参照して下さい。

.. seealso::

   Python公式ドキュメント
      `11.1. pickle - Python オブジェクトの整列化 <http://www.python.jp/doc/nightly/library/pickle.html>`_
      `11.4. shelve - Python オブジェクトの永続化 <http://www.python.jp/doc/nightly/library/shelve.html>`_

ロギング
--------

スクリプトの途中で様々な情報をファイルに書き出したい場合があります。例えば、特定のエラーが発生した時にそれを残したり、デバッグ用の文字列を保存したり、といった感じです。そのような要求には *logging* モジュールを使うと簡単に応えることができます::

   >>> import logging
   >>> logging.basicConfig(filename='log.out', level=logging.ERROR)
   >>> logging.debug('Debugging')
   >>> logging.info('Information')
   >>> logging.warning('Warning')
   >>> logging.error('Error')
   >>> logging.critical('Critical')

この例では ERROR 以上のレベルのログだけが保存されるので、 log.out は以下のようになります::

   % cat log.out
   Error
   Critical

スクリプトを作成する際は、至る所でログを記録するようにし、ロガーのレベルを調整して保存内容を変えるようにするとよいでしょう。

.. seealso::

   Python公式ドキュメント
      `チュートリアル 11.5. ログ記録<http://www.python.jp/doc/nightly/tutorial/stdlib2.html#tut-logging>`_
      `15.6. logging - Python 用ロギング機能 <http://www.python.jp/doc/nightly/library/logging.html>`_

課題
====

課題1 標準入出力とファイル操作
------------------------------

sortシェルコマンドを模倣し、次の仕様を満たすpythonスクリプトを作成する。

#. 標準入力から入力された値をソートできる
#. コマンドライン引数でファイルを指定された場合は、ファイルをソートする
      複数のファイルを指定できる
      複数のファイルを指定された場合は、それらのファイルを横断してソートする
#. 結果は標準出力に書きだす
      -oオプションが指定された場合は、指定されたファイルに結果を出力する
#. -kオプションでソートに用いる列を指定できる
#. -tオプションで列を分割するセパレータを指定できる
#. -rオプションを付けるとソートが逆順になる
#. -hオプションを付けるとヘルプが表示される
#. -uオプションを付けると同じ内容の行の出力が抑制される

次のコードをsort.pyという名前で保存し、実装せよ。ソートの実装はsorted組み込み関数かリストのsortメソッドを用いて良い::

   # -*- coding: utf-8 -*-

   def cmdsort(opts, args):
       '''ソートコマンドのエミュレータ

       引数のoptsとargsはOptionParserによって作成される。
       例えば、-kオプションに設定された値はopts.posで取得できる。設定されていない場合はNoneが返る。
       '''
       pass


   if __name__ == '__main__':
       from optparse import OptionParser
       parser = OptionParser()
       parser.add_option('-k', dest='pos', metavar='POS1[,POS2]')
       parser.add_option('-t', dest='sep', metavar='SEPARATOR')
       parser.add_option('-o', dest='outfile', metavar='OUTFILE')
       parser.add_option('-r', dest='reverse', action='store_true')
       parser.add_option('-u', dest='unique', action='store_true')
       options, args = parser.parse_args()
       if not options.test:
           cmdsort(options, args)
       else:
           import doctest
           doctest.testmod()

課題2 データベース
------------------

次の様なスキーマのMySQLのデータベースがあるとする。::

   DROP TABLE IF EXISTS page;
   CREATE TABLE page (
       id INTEGER UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
       url VARCHAR(255) NOT NULL UNIQUE
   );

   DROP TABLE IF EXISTS word;
   CREATE TABLE word (
       id INTEGER UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
       word VARCHAR(255) NOT NULL UNIQUE
   );

   DROP TABLE IF EXISTS location;
   CREATE TABLE location (
       pageid INTEGER UNSIGNED NOT NULL,
       wordid INTEGER UNSIGNED NOT NULL,
       location INTEGER NOT NULL,
       FOREIGN KEY(pageid) REFERENCES page(id) ON DELETE CASCADE,
       FOREIGN KEY(wordid) REFERENCES word(id) ON DELETE RESTRICT,
       PRIMARY KEY (pageid, location)
   );

   DROP TABLE IF EXISTS link;
   CREATE TABLE link (
       id INTEGER UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
       fromid INTEGER UNSIGNED NOT NULL,
       toid INTEGER UNSIGNED NOT NULL,
       FOREIGN KEY(fromid) REFERENCES page(id) ON DELETE CASCADE,
       FOREIGN KEY(toid) REFERENCES page(id) ON DELETE CASCADE,
       UNIQUE (fromid, toid)
   );

   DROP TABLE IF EXISTS linkword;
   CREATE TABLE linkword (
       linkid INTEGER UNSIGNED NOT NULL,
       wordid INTEGER UNSIGNED NOT NULL,
       FOREIGN KEY(linkid) REFERENCES link(id) ON DELETE CASCADE,
       FOREIGN KEY(wordid) REFERENCES word(id) ON DELETE RESTRICT,
       PRIMARY KEY (linkid, wordid)
   );
