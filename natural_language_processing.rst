======================
正規表現と自然言語処理
======================

.. contents:: :depth: 3

.. seealso::

   :title-reference:`Introduction to Information Retrieval`

      chapter 2
      chapter 3
      chapter 6

動機付け
========

情報検索は、集めたオブジェクトから特徴量を抽出し、得られた特徴から検索用のインデックスを作成します。例えば今回の作成しているWeb検索システムの場合、クローラが集めたWebページが検索対象オブジェクトとなり、それぞれのページに記述されているテキストや埋めこまれている画像、メタデータなどがコンテンツとなります。そして、それらのコンテンツから、時間表現や単語頻度、画像ヒストグラムなどを特徴量として抽出できます。

このように、あるオブジェクトから特徴を抽出し、それをベクトルとして表現したものを、このオブジェクトの **特徴ベクトル** と呼びます。特徴ベクトルを用いた検索については、「文書の検索」の回で改めて解説します。

今回扱う正規表現と自然言語処理は、テキストコンテンツから特徴量を抽出するのに用いることができます。例えば次のような文書があったとしましょう。::

  Python(パイソン)は、オランダ人のグイド・ヴァンロッサムが作った
  オープンソースのプログラミング言語。
  
  最新リリース
  3.2 / 2011年2月20日

このような文書から

   時間表現の抽出
       2011年2月20日

   単語(名詞)の抽出
       Python、パイソン、オランダ人、グイド・ヴァンロッサム、オープンソース、プログラミング、言語

のような特徴を抽出できるようにすることが、今回の目標です。

正規表現
========

正規表現とは文字列のパターンを表現する表記法のことで、マッチする文字列を直接指定せず、パターンを指定することで、表記ゆれを吸収しての操作が可能となります。正規表現は多くのプログラミング言語で実装されていますが、それぞれの実装内容が微妙に異なっています。今回は、多くの正規表現実装に共通して用いられているメタ文字について触れ、Pythonで正規表現を提供するreモジュールの使い方について説明します。

メタ文字
--------

単一の文字にマッチするメタ文字

   :literal:`'.'`
       
      **(任意の1文字)** Pythonでは改行文字以外の任意の一文字にマッチします。DOTALLフラグを用いると、改行にもマッチするようになります。
   
   :literal:`'[...]'`
   
      **(括弧内の任意の1文字)** 文字の集合を指定するのに使用します。括弧内の任意の1文字に対応します。文字は個々に記述するか、文字の範囲を、2つの文字と ``'-'`` でそれらを分離して指定することができます。例えば、 ``[ab]`` は文字 ``'a'`` 、 ``'b'`` とマッチします。 ``[a-z]`` は任意の小文字と、 ``[a-zA-Z0-9]`` は任意の英数字にマッチします。文字集合に ``'-'`` や ``']'`` を含めたい場合は、その前にバックスラッシュを付けるか、 ``[`` の直後に置きます。
   
   :literal:`'[^...]'`
   
      **(括弧内以外の任意の1文字)** ``[...]`` の *補集合* とマッチします。例えば、 ``[^ab]`` は文字 ``'a'`` 、 ``'b'`` 以外とマッチします。

量指定子?: "繰り返し"を提供するために付加されるメタ文字

   :literal:`'?'`
   
      直前にある正規表現を0回か1回繰り返したものにマッチします。
   
   :literal:`'*'`
   
      直前にある正規表現を0回以上繰り返したものにマッチします。
   
   :literal:`'+'`
   
      直前にある正規表現を1回以上繰り返したものにマッチします。
   
   :literal:`'{min,max}'`
   
      直前にある正規表現を ``'min'`` 回以上 ``'max'`` 回以下繰り返したものにマッチします。
   
   :literal:`'??'`, :literal:`'*?'`, :literal:`'+?'`
   
      ``'?'``, ``'*'``, ``'+'`` は全てできるだけ多くのテキストにマッチするようになっており、このようなマッチを *最長一致* と言います。一方、 ``'?'`` を修飾子の後に追加するとできるだけ少ないテキストにマッチするようになり、これを *最小一致* と言います。

位置を表すメタ文字

   :literal:`'^'`
   
      **(行の先頭の位置)** 文字列の先頭とマッチします。PythonではMULTILINEオプションを指定すると各改行文字の直後にマッチするようになります。
   
   :literal:`'$'`
   
      **(行の文末の位置)** 文字列の末尾か文字列の末尾の改行の直前にマッチします。PythonではMULTILINEオプションを指定すると各改行文字の直前にマッチするようになります。

その他のメタ文字

   :literal:`'|'`
   
      **(選択 OR)** 任意の正規表現 ``A`` と ``B`` に対して、 ``A|B`` は ``A`` か ``B`` のどちらかとマッチする正規表現を作成します。
   
   :literal:`'(...)'`
   
      **(グループ化)** 丸括弧の中にどのような正規表現があってもマッチし

文字クラス

   :literal:`'\\s'`
   
      **(空白文字(タブ、スペース、改行))** Pythonでは ``[\t\n\r\f\v]`` と同じ意味です。
   
   :literal:`'\\S'`
   
      **(\s以外の任意の文字)** Pythonでは ``[^\t\n\r\f\v]`` と同じ意味です。
   
   :literal:`'\\w'`
   
      **(英数字とアンダーステア)** ``[a-zA-Z0-9_]`` と同じ意味です。
   
   :literal:`'\\W'`
   
      **(\w以外の任意の文字)** ``[^a-zA-Z0-9_]`` と同じ意味です。
   
   :literal:`'\\d'`
   
      **(数字)** ``[0-9]`` と同じ意味です。
   
   :literal:`'\\D'`
   
      **(\d以外の任意の文字)** ``[^0-9]`` と同じ意味です。

注意点
^^^^^^

* メタ文字自身を使う場合は前に ``\`` を付ける必要がある。
* ``|`` は *遅い*

     A) ^(a|b|c|d|e|f)+$
     B) ^(?:a|b|c|d|e|f)+$
     C) ^[a-f]+$

  これらは全て同じ文字列にマッチしますが、 *BはAの3倍、CはAの20倍高速* に動作します。

* ``[]`` の中では特殊文字は効果を持ちません。なので、 ``[.]`` は文字 ``.`` に **のみ** マッチします。
* 任意の深さを持つ入れ子構造に正規表現をマッチさせることは **できません。** (ライブラリによる拡張はある)

正規表現を理解するのに良いWebアプリ
-----------------------------------

#. RegExr

   正規表現は実際に記述しなければ理解することが難しいので、自分で試してみることが重要でが、毎回毎回Pythonのreモジュールを使って試すのは大変ですし、時間もかかります。
   
   RegExrは入力した正規表現の適合箇所を簡単に確認することができるWebアプリケーションです。
   上のテキストボックスに正規表現を入力すると、下の文書の中でそれにマッチする箇所を表示してくれるので、トライアンドエラーのサイクルが短くすることができます。
   
   プログラムを書く前に、意図した通りに表現できているかを確かめる用途にも使うことができます。
   
   URL: http://www.gskinner.com/RegExr/
   
   .. image:: /images/RegExr.png

#. strfriend

   正規表現は理論的にはオートマトンを用いて説明することができます。
   
   strfriendは入力された正規表現を表す非決定性オートマトンを出力してくれるWebアプリケーションです。
   これを用いて正規表現を可視化することで、複雑で難しい正規表現が理解しやすくなるかも知れません。
   
   URL: http://www.strfriend.com/
   
   .. image:: /images/strfriend1.png
   
   メールアドレスにマッチする正規表現を入力した場合
   
   .. image:: /images/strfriend2.png

Pythonでの使用法
----------------

.. seealso::

   Python公式ドキュメント
      `7.2. re - 正規表現操作 <http://www.python.jp/doc/nightly/library/re.html>`_

#. マッチするものを全て列挙する場合、findallを使います。::

       >>> import re
       >>> text = 'pythonとはlightweightな、programming言語である'
       >>> re.findall('\w+', text)
       ['python', 'lightweight', 'programming']

   findallはグループにも対応しています。::

       >>> text = 'pythonとはlightweightな、programming言語である'
       >>> re.findall('(\w+)とは(\w+)', text)
       [('python', 'lightweight')]

   グループが邪魔な場合は(?:...)の様に、?:をグループの最初につけます。::

      >>> re.findall('(?:\w+)とは(?:\w+)', text)
      ['python\xe3\x81\xa8\xe3\x81\xaflightweight']

#. マッチ部分に対応するMatchObjectを取得したい場合は、finditerを使います。::

       >>> import re
       >>> text = 'pythonとはlightweightな、programming言語である'
       >>> for mo in re.finditer('(\w+)とは(\w+)', text)
       ...     print mo.group(0)
       ...     print mo.group(1)
       ...     print mo.group(2)
       ...
       pythonとはlightweight
       python
       lightweight

   MatchObjectは名前付きのグループを使った時に特に便利です。次のようにgroupdictを使うことで、グループ名をキーとした辞書が返されます。::

       >>> text = 'pythonとはlightweightな、programming言語である'
       >>> re.findall('(?P<first>\w+)とは(?P<second>\w+)', text)
       [('python', 'lightweight')]
       >>> for mo in re.finditer(pattern, text):
       ...     print mo.groupdict()
       ...
       {'first': 'python', 'second': 'lightweight'}

   例えば日付表現を抽出する場合、次のように名前付きグループを作ることで、マッチした箇所の抽出するプログラムの可読性を高めることができます。::

       >>> pattern = '(?P<year>[1-9]\d{1,3})年(?P<month>1[0-2]|[1-9])月(?P<day>3[01]|[12]\d|[1-9])日'
       >>> text = '''リリース
       ... 3.2/ 2011年2月20日
       ... 2.7.1/ 2010年11月27日
       ... '''
       >>> for mo in re.finditer(pattern, text):
       ...    # mo.group(2)と比べて月を抽出していることが明確になる。
       ...    print mo.groupdict()['month']
       ...
       2
       11

#. 文字列を先頭から順番に見ていき、正規表現にマッチする最初の箇所が欲しい場合はsearchを使います。searchの返り値はMatchObjectなので、groupdictを利用することができます。::

       >>> import re
       >>> text = 'pythonとはlightweightな、programming言語である'
       >>> mo = re.search('l\w+', text)
       >>> print s.group()
       lightweight

#. 文字列が先頭から正規表現にマッチしているかを知りたい場合はmatchを使います。::

       >>> import re
       >>> text = 'pythonとはlightweightな、programming言語である'
       >>> re.match('l\w+', text) # 先頭はlで始まらない
       None
       >>> print re.match('\w+', text).group()
       python

   逆にmatchを使うと暗黙的に文字列の先頭からを意味することになるので、注意して下さい。

#. 正規表現パターンから正規表現オブジェクトに変換するのは時間のかかる処理です。そのため、繰り返し利用される正規表現パターンはcompileを使うことで、正規表現オブジェクトを再利用することができます。::

       >>> import re
       >>> regex = re.compile('\w+')  # regexを繰り返し再利用することができる
       >>> text = 'pythonとはlightweightな、programming言語である'
       >>> regex.findall(text)
       ['python', 'lightweight', 'programming']

   ただし、re.match(), re.search(), re.compile()は渡された最後の物がキャッシュとして残るので、正規表現パターンが1種類しかでてこない場合は、compileを利用する必要はありません。

#. 複数行にまたがる文字列に対し、各行の行頭や(各改行の直後)や行末(改行の直前)にマッチさせたい場合、re.MULTILINEオプションを指定した上で、^や$を使います。::

       >>> import re
       >>> pattern = '^\w+'
       >>> text = '''python
       ... パイソン
       ... ルビー ruby
       ... perl
       ... C言語
       ... '''
       >>> re.findall(pattern, text, re.MULTILINE)
       ['python', 'perl', 'C']
       >>> re.findall(pattern, text, re.M)  # re.MでもOK
       ['python', 'perl', 'C']

   逆に、re.MULTILINEをつけ忘れると、^と$は文字列の最初と最後にのみマッチするようになります。::

       >>> re.findall('^\w+', text)
       ['python']


自然言語処理
============

.. warning::

   ここでは情報検索のために、与えられた文書に対して単語ベースの特徴ベクトルを作成することを主眼において、自然言語処理について説明していますが、これは本来の意味での自然言語処理が指す領域からすると、極めて限定的な話題のみを扱っていることを意味します。

.. seealso::

   自然言語処理について、より深く学びたい場合は :title-reference:`入門自然言語処理(O'REILLY)` をオススメします。 本書はPythonの自然言語処理モジュールnltkを用いた自然言語処理の入門書です。
   第12章「 `Pythonによる日本語自然言語処理 <http://nltk.googlecode.com/svn/trunk/doc/book-jp/ch12.html>`_ 」はWebから無料で読むこともできます。
      
形態素解析
----------

自然言語処理ではまず、自然言語で記述された文書を文法や辞書を情報源として、形態素(Morpheme, 言語で意味を持つ最小単位)に分割する必要があります。形態素解析とは、自然言語を形態素に分割し、それぞれの品詞を判別する作業のことをいいます。

例えば、「すもももももももものうち」という文章は次のように分解することができます。

+------+----+
|単語  |品詞|
+======+====+
|すもも|名詞|
+------+----+
|も    |助詞|
+------+----+
|もも  |名詞|
+------+----+
|も    |助詞|
+------+----+
|もも  |名詞|
+------+----+
|の    |助詞|
+------+----+
|うち  |名詞|
+------+----+

文書の特徴ベクトルの構築という観点からすると、主に「名詞」「形容詞」「動詞」を抽出することになります。

フリーの形態素解析器としては、次のようなものがあります。

* 日本語
   #. MeCab http://mecab.sourceforge.net/
   #. JUMAN http://www-lab25.kuee.kyoto-u.ac.jp/nl-resourece/juman.html
* 英語
   #. Stanford Parser http://nlp.stanford.edu/software/lex-parser.shtml
   #. TreeTagger http://www.ims.uni-stuttgart.de/projekte/corplex/TreeTagger/

注意点としては、次のようなものが挙げられます。

* 一般的に処理が重い
* 崩れた表現はうまく処理できない
     2ちゃんねる、ニコニコ動画、Twitterなどで記述されている文章など
* 崩れた表現ほど処理に時間がかかる(その上結果も悪い)
* 新語はうまく処理できないことが多い

最後の新語に対応できない、という問題は形態素解析器が単語辞書をベースに動いていることに起因します。つまり、辞書に存在しない単語は未知語として処理することになります。

この問題は、新語を単語辞書に追加することである程度対応することができますが、全ての単語を網羅することは現実的ではなく、現在は統計情報を用いた推定を行うのが主流となっているようです。

.. note::

   品詞のことは英語で Part-of-Speech, 略してPOS(ピーオーエスと読まれることが多い?)。
   英語の形態素解析器は pos tagger で検索すると、多くの情報がヒットします。

正規化
------

形態素解析を行うことで、文書を語(形態素)に分割することができましたが、現在の状態では、例えば「python」と「PYTHON」と「Python」や「woman」と「women」などの語が区別されています。

しかしながら、一般的にこれらの単語は同じ物として扱いたいことが多いので、形態素解析を行った次は、語の正規化を行います。英単語の正規化は大きく分けて次の3ステップで行います。

#. 大文字・小文字の統一
      一般的に小文字に統一されることが多い

      python, PYTHON, Python -> python

#. ステミング(stemming)

      与えられた語の語幹を取り出す

      database, databases -> databas

      initial, initialize, initialization -> initi

#. 見出し語化/レンマ化(lemmatization)

      与えられた語の、辞書における見出し語を求める

      women -> woman

      databases -> database

ストップワードの除去
--------------------

与えられた文書の形態素解析を行った後、語の正規化を行いました。これで、文書の単語出現頻度を作ることができます。しかしながら、自然言語処理は多くの文書に共通して現れる単語が多く存在します。例えば、theという単語は多くの文書に現れるため、仮にtheが文書に出現したとしても、その文書を特徴付けるものとはなりません。

このようなほとんど全ての文書に出現する語をストップワードと呼び、正規化して得られた語リストから除去する必要があります。

ストップワードは事前に知識として与えることが可能で、日本語と英語のストップワードは例えばSlothLibプロジェクトからダウンロードすることができます。

* `日本語ストップワード <http://svn.sourceforge.jp/svnroot/slothlib/CSharp/Version1/SlothLib/NLP/Filter/StopWord/word/Japanese.txt>`_
* `英語ストップワード <http://svn.sourceforge.jp/svnroot/slothlib/CSharp/Version1/SlothLib/NLP/Filter/StopWord/word/English.txt>`_

Pythonでの使用法
----------------

#. 形態素解析 MeCab

   MeCab.Taggerクラスのインスタンスを生成し、parseメソッドを呼ぶことで解析結果を文字列として取得できます。::

      >>> import MeCab
      >>> tagger = MeCab.Tagger('-Ochasen')
      >>> print tagger.parse('本日は晴天なり')
      本日	ホンジツ	本日	名詞-副詞可能		
      は	ハ	は	助詞-係助詞		
      晴天	セイテン	晴天	名詞-一般		
      なり	ナリ	なり	助動詞	文語・ナリ	基本形
      EOS

   parseToNodeメソッドを使うこともできます。parseToNodeメソッドはMeCab.Nodeクラスのインスタンスを返し、nextメソッドでノードをたどることができます。これには文頭、文末形態素というものが含まれているので、これらを無視したい場合は次のように利用します。::

      >>> node = tagger.parseToNode('本日は晴天なり')
      >>> node = node.next  # 「文頭」を無視
      >>> while node.next is not None:  # 「文末」のnextがNoneであることを利用して「文末」を無視
      ...     print node.surface, node.feature
      ...     node = node.next  # 次に移動
      ...
      本日 名詞,副詞可能,*,*,*,*,本日,ホンジツ,ホンジツ,,
      は 助詞,係助詞,*,*,*,*,は,ハ,ワ,,
      晴天 名詞,一般,*,*,*,*,晴天,セイテン,セイテン,,
      なり 助動詞,*,*,*,文語・ナリ,基本形,なり,ナリ,ナリ,,

   次のように書くことで、文章から名詞のみを抽出することができます。::

      >>> node = tagger.parseToNode('本日は晴天なり')
      >>> node = node.next
      >>> while node.next is not None:
      ...     if node.feature.split(',')[0] == '名詞':
      ...         print node.surface
      ...     node = node.next
      ...
      本日
      晴天

   .. seealso::

      コンストラクタにはmecabの実行形式に与えるパラメータを文字列として与えることができます。
      ここではchasen互換モードでMeCabを呼び出しています。詳しくはMeCabのドキュメントを参照してください。
         `MeCab: Yet Another Part-of-Speech and Morphological Analyzer <http://mecab.sourceforge.net>`_

#. 語の正規化

   英語の大文字・小文字を正規化する場合、次の3つのメソッドを使います。::

      >>> 'PyThOn'.lower()  # 小文字に正規化
      'python'
      >>> 'PyThOn'.upper()  # 大文字に正規化
      'PYTHON'
      >>> 'PyThOn'.capitalize()  # タイトル文字に正規化
      'Python'

   ステミング処理には、nltkモジュールのnltk.PorterStemmerやnltk.LancasterStemmerなどを使います。::

      >>> import nltk
      >>> stemmer = nltk.PorterStemmer()
      >>> words = ['database', 'databases', 'distribute', 'distribution']
      >>> [stemmer.stem(word) for word in words]
      ['databas', 'databas', 'distribut', 'distribut']

   見出し語化を行うには、nltkモジュールのnltk.WordNetLemmatizerを使いますが、この処理は時間がかかるので事前にステミング処理を行うなどして、単語の数を減らすように注意をして下さい。::

      >>> lemmatizer = nltk.WordNetLemmatizer()
      >>> words = ['women', 'databases']
      >>> [lemmatizer.lemmatize(word) for word in words]
      ['woman', 'database']

#. ストップワードの除去

   英語用のストップワードはnltk.corpus.stopwords.words('english')で取得することができます。::

      >>> from nltk.corpus import stopwords
      >>> len(stopwords.words('english'))
      127
      >>> stopwords.words('english')[:10]
      ['i', 'me', 'my', 'myself', 'we', 'our', 'ours', 'ourselves', 'you', 'your']

   日本語用のストップワードはnltkには用意されていないので、例えばSlothLibのストップワードを使うことができます。
      `SlothLib ストップワードリスト <http://svn.sourceforge.jp/svnroot/slothlib/CSharp/Version1/SlothLib/NLP/Filter/StopWord/word/Japanese.txt>`_

#. nltkのその他の機能

   nltk.FreqDistクラスはコンストラクタで単語のリストを受け取り(イテレータでも可)、term frequencyベクトルのように動作するFreqDistインスタンスを生成します。::

      >>> import nltk, MeCab
      >>> sentence = '''MeCabは 京都大学情報学研究科日本電信電話株式会社
      ... コミュニケーション科学基礎研究所 共同研究ユニットプロジェクトを
      ... 通じて開発されたオープンソース 形態素解析エンジンです.
      ... 言語, 辞書,コーパスに依存しない汎用的な設計を 基本方針としています.
      ... パラメータの推定に Conditional Random Fields (CRF) を用いており,
      ... ChaSenが採用している隠れマルコフモデルに比べ性能が向上しています．
      ... また、平均的にChaSen, Juman, KAKASIより高速に動作します.
      ... ちなみに和布蕪(めかぶ)は, 作者の好物です.'''
      >>> tagger = MeCab.Tagger()
      >>> node = tagger.parseToNode(sentence).next
      >>> words = []
      >>> while node.next is not None:
      ...     if node.feature.split(',')[0] == '名詞':
      ...         words.append(node.surface.lower())
      ...     node = node.next
      ...
      >>> fdist = nltk.FreqDist(words)
      >>> fdist['chasen'] # sentenceの中でchasenという名詞が現れた回数
      2
      >>> fdist.freq('chasen') # sentenceの中でのchasenの相対的な頻度
      0.0392156862745

課題
====

課題1 ex1_re.py
---------------

1. 与えられた文字列から **時間表現を抽出** する関数(ex11)を作成せよ。

   この課題での時間表現とは *時分秒* を表し、次の形式のいずれかとする。

   A. 1:12:13
      時分秒は:で区切られる 1時12分13秒
   B. 01:12:13
      0による桁あわせ
   C. 01:12:13 pm
      12時間表記 半角スペース1個の後にpmもしくはam
   D. 01:12:13 p.m.
      12時間表記 半角スペース1個の後にp.m.もしくはa.m.
   
   **注意点**
   
   * 0時0分0秒から23時59分59秒の間のみ抽出する
     99:99:99のような表現は抽出しない
   * 14:00:00 p.m. のような表現は抽出しない
   * HWaddr 00:23:54:91:03:09 のような表現は抽出しない
   * すべてを正規表現で行う必要はない
     正規表現で時間表現の候補を抽出 -> 無効な表現を削除

2. 与えられた文字列から時間表現を抽出し、それらを **hh:mm:ss形式に正規化** する関数(ex12)を作成せよ。

   A. 1:12:13       -> 01:12:13
   B. 01:12:13 p.m. -> 13:12:13

次のコードをex1_re.pyという名前で保存し、テストが通るように実装する::

   # -*- coding: utf-8 -*-
   
   
   def ex11(text):
       '''課題1-1
       引数の文字列(text)から時間表現を抽出する。
   
           >>> ex11('1:2:3 to 1:3:3')
           ['01:02:03', '01:03:03']
           >>> ex11('updated at 0:00:00')
           ['0:00:00']
           >>> ex11('11:15:30 pm')
           ['11:15:30 pm']
           >>> ex11('11:15:30 am')
           ['11:15:30 am']
           >>> ex11('11:15:30 p.m.')
           ['11:15:30 p.m.']
           >>> ex11('11:15:30 a.m.')
           ['11:15:30 a.m.']
           >>> ex11('12:23:34 pmi conference ...')
           ['12:23:34']

       Macアドレスなどに反応してはいけない。

           >>> ex11('2011:05:17')
           []
           >>> ex11('HWaddr 00:23:54:91:03:05')
           []
           >>> ex11('23:11: ')
           []
           >>> ex11('12:234:56')
           []
           >>> ex11('14:00:00 pm')
           []
           >>> ex11('24:00:00')
           []
           >>> ex11('99:99:99')
           []
       '''
       pass
   
   
   if __name__ == '__main__':
       import doctest
       doctest.testmod()

テストは次のようにすることで実行できる::

   $ python ex1_re.py

課題2 ex2_nlp.py
----------------

1. 与えられた単語が **ストップワードであるかどうかを判別** する関数(ex21)を作成せよ。

   * 何がストップワードであるかは好きに決めていい
   * SlothLibのストップワードリストを使用してもいい
   * nltkのストップワードリスト(英語のみ利用可能)を使用してもいい

2. 与えられた文字列（日本語ベース）を **形態素解析し、名詞のみを抽出し、正規化し、ストップワードを除去した後、単語の出現回数をカウントしたディクショナリ** を作成する関数(ex22)を作成せよ。

      例えば::

         Database (<複> databases)とは、特定のテーマに沿ったデータを集めて管理し、
         容易に検索・抽出などの再利用をできるようにしたもの。

      という文字列が入力された場合::

         {"複": 1, "データ": 1, "管理": 1, "再": 1, "抽出": 1, "database": 2,
          "特定": 1, "検索": 1, "テーマ": 1, "容易": 1, "利用": 1}

次のコードをex2_nlp.pyという名前で保存し、テストが通るように実装する。::

   # -*- coding: utf-8 -*-
   
   
   def ex21(word):
       '''課題2-1
       引数の文字列(word)がストップワードであればTrueを返す
   
           >>> ex21("こと")
           True
           >>> ex21("データベース")
           False
           >>> ex21("the")
           True
           >>> ex21("database")
           False
       '''
       pass
   
   
   def ex22(text):
       '''課題2-2
       引数の文字列(text)から名詞を抽出し、正規化、 ストップワードを除去する。
       その後、単語の出現頻度をカウントしたディクショナリを返す。
       下記はあくまでも一例
   
           >>> text = """Database (<複> databases)とは、
           ... 特定のテーマに沿ったデータを集めて管理し、
           ... 容易に検索・抽出などの再利用をできるようにしたもの。"""
           >>> tf = ex22(text)
           >>> for key in sorted(tf.keys()):
           ...     print key, tf[key]
           ...
           database 2
           テーマ 1
           データ 1
           再 1
           利用 1
           容易 1
           抽出 1
           検索 1
           特定 1
           管理 1
           複 1

       ここで得られた辞書型オブジェクトtfのように、ベクトルの各次元が単語の文書中での
       出現回数となっているものをterm frequencyベクトルという。
       多くの場合、省略して単にtfベクトルとも呼ばれる。
       '''
       pass
   
   if __name__ == '__main__':
       import doctest
       doctest.testmod()

テストは次のようにすることで実行できる::

   $ python ex2_nlp.py
