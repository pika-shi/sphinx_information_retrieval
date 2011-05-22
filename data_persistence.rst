==============
データの永続化
==============

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
