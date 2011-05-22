==========
Python入門
==========

課題
====

課題1 クイックソート
--------------------

次のコードをsort.pyという名前で保存し、テストが通るように実装せよ。::

   # -*- coding: utf-8 -*-
   import unittest


   def qsort(seq):
       '''クイックソート

       引数で与えられたseqをソートした新しいオブジェクトを返す。

           >>> import random
           >>> l = [random.randint(1, 20) for i in xrange(10)]
           >>> c = l[:]  # テスト用にlのコピーを作成

       ソート結果を組み込みのsorted関数を使ってソートした結果と比較する::

           >>> for a, b in zip(qsort(l), sorted(l)):
           ...     assert a == b
           ...

       qsortは引数で与えられたseqを破壊しない::

           >>> for a, b in zip(l, c):
           ...     assert a == b
           ...

       '''
       pass


   if __name__ == '__main__':
       import doctest
       doctest.testmod()

また、次のテストコードをtest_sort.pyという名前でsort.pyと同じディレクトリに保存せよ。::

   # -*- coding: utf-8 -*-
   import random
   import unittest


   from sort import qsort


   class TestSortFunctions(unittest.TestCase):

       def test_positive(self):
           seq = [i for i in reversed(range(10))]  # [9, 8, 7, ..., 0]
           self.assertEqual(qsort(seq), sorted(seq))

       def test_negative(self):
           seq = [i for i in reversed(range(-10, 0))]  # [-1, -2, -3, ..., -10]
           self.assertEqual(qsort(seq), sorted(seq))

       def test_randint(self):
           seq = [random.randint(-100, 100) for i in range(10)]
           self.assertEqual(qsort(seq), sorted(seq))

       def test_random_float(self):
           seq = [random.random() for i in range(10)]
           self.assertEqual(qsort(seq), sorted(seq))


   if __name__ == '__main__':
       unittest.main()

テストは次のように実行することができる。::

   $ python sort.py
   $ python test_sort.py

課題2 オブジェクト指向プログラミング
------------------------------------

Pythonでは__(アンダースコア2つ)で囲まれたメソッドを、特殊メソッドと呼ぶ。特殊メソッドを決められた仕様に沿って実装することで、様々なポリモーフィズムの恩恵を得ることができる。 例えば、__iter__特殊メソッドを実装したクラスのインスタンスはfor文の中で用いることができるようになる。

次のコードをlinkedlist.pyという名前で保存し、テストが通るように連結リストを実装せよ。::

   # -*- coding: utf-8 -*-


   from collections import Iterator, Sequence, MutableSequence
   
   
   class MyLinkedList(object): # Sequence):  # MutableSequence):
   
       def __init__(self):
           'コンストラクタ'
           self.head = Node(None)
   
       def __contains__(self, value):
           '''inステートメントで呼ばれる特殊メソッド
   
           collections.Sequence基底クラスのMixinメソッド
   
           inステートメントで呼ばれる。value in self の結果をTrueかFalseで返す
   
               >>> l = MyLinkedList()
               >>> l.append(1)
               >>> 1 in l  # l.__contains__(1) に等しい
               True
               >>> 2 in l
               False
   
           '''
           pass
   
       def __iter__(self):
           '''iter関数で呼ばれる特殊メソッド
   
           collections.Sequence基底クラスのMixinメソッド
   
           イテレータデザインパターンにおけるイテレータオブジェクトを返す。Python
           においては、主にfor文で用いられる。
   
               for a in b:
                   do(a)
   
           は次のコードと等しい。
   
               iterator = iter(b)  # = b.__iter__()
               try:
                   while True:
                       a = next(iterator)  # = iterator.next()
                       do(a)
               except StopIteration:
                   pass
   
           (余談) Python3からはnext組み込み関数で呼ばれるのが、nextメソッドから
           __next__特殊メソッドに変更されました。
   
               >>> l = MyLinkedList()
               >>> l.append(1)
               >>> iterator = iter(l)
               >>> isinstance(iterator, MyIterator)
               True
               >>> next(iterator)
               1
   
           iteratorオブジェクトは最後まできたらStopIteration例外を発生させなければ
           ならない。
   
               >>> item = next(iterator)
               Traceback (most recent call last):
                   ...
               StopIteration
   
           __iter__特殊メソッドを実装することでfor文で使うことができるようになる。
   
               >>> l.append(2)
               >>> for i in l:
               ...     print i
               ...
               1
               2
   
           '''
           pass
   
       def __len__(self):
           '''len関数で呼ばれる特殊メソッド
   
               >>> l = MyLinkedList()
               >>> len(l)
               0
               >>> l.append(1)
               >>> len(l)
               1
               >>> l.append(2)
               >>> len(l)
               2
   
           '''
           pass
   
       def __getitem__(self, key):
           '''インデックス記法で呼ばれる特殊メソッド
   
           collections.Sequence基底クラスの抽象メソッド
   
           配列のようなインデックス記法を用いた場合に呼び出され、``key''番目の要素
           の値を返す。
   
               >>> l = MyLinkedList()
               >>> l.append(1)
               >>> l[0]    # l.__getitem__(0)
               1
   
           設定されていないインデックスにアクセスされた場合はIndexError例外を発生
           させなければならない。
   
               >>> l[2]
               Traceback (most recent call last):
                   ...
               IndexError
   
           ``key''が数字以外だった場合TypeError例外を発生させなければならない
   
               >>> l['string']
               Traceback (most recent call last):
                   ...
               TypeError
   
           ``key''の値は-len(self) <= key < len(self)でなければならず、範囲外の
           場合はIndexError例外を発生させなければならない。
   
               >>> l.append(2)
               >>> l[-2]
               1
               >>> l[2]
               Traceback (most recent call last):
                   ...
               IndexError
               >>> l[-3]
               Traceback (most recent call last):
                   ...
               IndexError
   
           '''
           pass
   
       def __setitem__(self, key, value):
           '''インデックス記法を用いた代入文で呼ばれる特殊メソッド
   
           collections.MutableSequence基底クラスの抽象メソッド
   
           ``key''番目の値を``value''に置き換える。
   
               >>> l = MyLinkedList()
               >>> l.append(1)  # [1]
               >>> l[0] = 2     # [2]
               >>> l[0]
               2
   
           設定されていないインデックスに対して代入を行おうとした場合はIndexError
           例外を発生させなければならない。
   
               >>> l[1] = 2
               Traceback (most recent call last):
                   ...
               IndexError
   
           ``key''が数字以外だった場合TypeError例外を発生させなければならない
   
               >>> l['string'] = 2
               Traceback (most recent call last):
                   ...
               TypeError
   
           ``key''の値は-len(self) <= key < len(self)でなければならず、範囲外の
           場合はIndexError例外を発生させなければならない。
   
               >>> l.append(2) # [2, 2]
               >>> l[-2] = 3   # [3, 2]
               >>> l[0]
               3
               >>> l[2]
               Traceback (most recent call last):
                   ...
               IndexError
               >>> l[-3]
               Traceback (most recent call last):
                   ...
               IndexError
   
           '''
           pass
   
       def __delitem__(self, key):
           '''インデックス記法を用いて削除を行う
   
           collections.MutableSequence基底クラスの抽象メソッド
   
           delステートメントで``key''番目の値を削除する
   
               >>> l = MyLinkedList()
               >>> l.append(1)
               >>> l.append(2)
               >>> l[0]
               1
               >>> del l[0]
               >>> l[0]
               2
   
           設定されていないインデックスの削除を行おうとした場合はIndexErrorを発生
           させなければならない。
   
               >>> del l[1]
               Traceback (most recent call last):
                   ...
               IndexError
   
           ``key''が数字以外だった場合TypeError例外を発生させなければならない
   
               >>> del l['string']
               Traceback (most recent call last):
                   ...
               TypeError
   
           ``key''の値は-len(self) <= key < len(self)でなければならず、範囲外の
           場合はIndexError例外を発生させなければならない。
   
               >>> del l[-1]
               >>> l[0]
               Traceback (most recent call last):
                   ...
               IndexError
   
           '''
           pass
   
       def __iadd__(self, value):
           '''+=オペランドで呼ばれる特殊メソッド
   
           collections.MutableSequence基底クラスのMixinメソッド
   
           戻り値が代入される
   
               >>> l = MyLinkedList()
               >>> l += [1]
               >>> l[0]
               1
               >>> l += [2]
               >>> l[1]
               2
   
           '''
           pass
   
       def __reversed__(self):
           '''reversed組み込み関数で呼ばれる特殊メソッド
   
           collections.Sequence基底クラスのMixinメソッド
   
           逆方向にたどるためのイテレータオブジェクトを返す。
   
               >>> l = MyLinkedList()
               >>> l.append(1)
               >>> l.append(2)
               >>> iterator = reversed(l)
               >>> isinstance(iterator, MyReverseIterator)
               True
               >>> next(iterator)
               2
               >>> next(iterator)
               1
   
           最後まできたらStopIteration例外を発生させなければならない
   
               >>> next(iterator)
               Traceback (most recent call last):
                   ...
               StopIteration
   
           for文で用いることも可能
   
               >>> for i in reversed(l):
               ...     print i
               ...
               2
               1
   
           '''
           pass
   
       def append(self, value):
           '''リストの末尾に``value''を加える
   
           collections.MutableSequence基底クラスのMixinメソッド
   
               >>> l = MyLinkedList()
               >>> l.append(1)
               >>> l.append(2)
   
           '''
           pass
   
       def pop(self):
           '''リストの末尾から値を一つ取り出す
   
           collections.MutableSequence基底クラスのMixinメソッド
   
               >>> l = MyLinkedList()
               >>> l.append(1)
               >>> l.pop()
               1
   
           空のリストに対して読んだ場合、IndexError例外を発生させなければならない
   
               >>> l.pop()
               Traceback (most recent call last):
                   ...
               IndexError
   
           '''
           pass
   
       def reverse(self):
           '''自分自身の順番を逆にする
   
           collections.MutableSequence基底クラスのMixinメソッド
           破壊的メソッドであることに注意。また、このメソッド自体は値を返さない。
   
               >>> l = MyLinkedList()
               >>> l.append(1)
               >>> l.append(2)
               >>> l.reverse()
               >>> l.pop()
               1
               >>> l.pop()
               2
   
           '''
           pass
   
       def index(self, value):
           '''``value''が最初に現れるインデックスを返す
   
           collections.Sequence基底クラスのMixinメソッド
   
               >>> l = MyLinkedList()
               >>> l.append(1)
               >>> l.append(2)
               >>> l.append(1)
               >>> l.index(1)
               0
               >>> l.index(2)
               1
   
           ``value''が存在しない場合はValueError例外を発生させなければならない。
   
               >>> l.index(3)
               Traceback (most recent call last):
                   ...
               ValueError
   
           '''
           pass
   
       def insert(self, key, value):
           '''リストへの挿入
   
           collections.MutableSequence基底クラスの抽象メソッド
   
           ``key''番目に``value''を挿入する。
   
               >>> l = MyLinkedList()
               >>> l.append(1)       # [1]
               >>> l.insert(0, 2)    # [2, 1]
               >>> l.insert(1, 3)    # [2, 3, 1]
               >>> l.insert(-1, 4)   # [2, 3, 4, 1]
               >>> l.insert(-3, 6)   # [2, 6, 3, 4, 1]
               >>> l.insert(-5, 5)   # [5, 2, 6, 3, 4, 1]
               >>> l.pop()
               1
               >>> l.pop()
               4
               >>> l.pop()
               3
   
           ``key``が大きすぎるもしくは小さすぎる場合は端に挿入される。例外は発生
           しない。
   
               >>> l.append(1)       # [5, 2, 6, 1]
               >>> l.insert(100, 2)  # [5, 2, 6, 1, 2]
               >>> l.pop()
               2
               >>> l.insert(-100, 0) # [0, 5, 2, 6, 1]
               >>> l.pop()
               1
               >>> l.pop()
               6
               >>> l.pop()
               2
               >>> l.pop()
               5
               >>> l.pop()
               0
   
           '''
           pass
   
       def count(self, value):
           '''要素の数え上げ
   
           collections.Sequence基底クラスのMixinメソッド
   
           ``value''と値が等しいオブジェクトが含まれる数を返す。
   
               >>> l = MyLinkedList()
               >>> l.append(1)
               >>> l.append(2)
               >>> l.append(1)
               >>> l.count(1)
               2
               >>> l.count(3)
               0
   
           同一のオブジェクトである必要はない
   
               >>> 1 is 1.0  # ヒント1
               False
               >>> 1 == 1.0  # ヒント2
               True
               >>> l.count(1.0)
               2
   
           '''
           pass
   
       def remove(self, value):
           '''要素の削除
   
           collections.MutableSequence基底クラスのMixinメソッド
   
           ``value''と値が等しいオブジェクトの内、最もインデックスの小さいものを
           削除する
   
               >>> l = MyLinkedList()
               >>> l.append(1)
               >>> l.append(2)
               >>> l.append(1)
               >>> l.append(2)
               >>> l.remove(2)
               >>> l.pop()
               2
               >>> l.pop()
               1
               >>> l.pop()
               1
               >>> l.pop()
               Traceback (most recent call last):
                   ...
               IndexError
   
           存在しない値が指定された場合はValueError例外を発生させなければならない
   
               >>> l.remove(2)
               Traceback (most recent call last):
                   ...
               ValueError
   
           '''
           pass
   
       def extend(self, seq):
           '''要素の拡張
   
           collections.MutableSequence基底クラスのMixinメソッド
   
           ``seq''を末尾に追加する。``seq''はiterableなオブジェクトでなければなら
           ない。
   
               >>> l = MyLinkedList()
               >>> l.extend([1,2,3])
               >>> l.pop()
               3
               >>> l.pop()
               2
               >>> l.pop()
               1
   
           iterableかどうかは次のようにして知ることができる
   
               >>> from collections import Iterable
               >>> isinstance([], Iterable)
               True
               >>> isinstance((), Iterable)
               True
               >>> isinstance({}, Iterable)
               True
               >>> isinstance(1, Iterable)
               False
               >>> isinstance('', Iterable)  # 文字列はiterable
               True
               >>> 'string'[1]
               't'
   
           '''
           pass
   
   
   class MyIterator(Iterator):
       '''MyLinkedListインスタンスに対するイテレータ'''
   
       def next(self):
           '''イテレータプロトコル
   
           collections.Iterator基底クラスの抽象メソッド
   
           最後まできた場合はStopIteration例外を発生させなければならない。
           詳しくはMyLinkedList.__iter__特殊メソッドのコメント参照
           '''
           pass
   
       # python3から__next__特殊メソッドに変更された
       __next__ = next
   
   
   class MyReverseIterator(Iterator):
       '''MyLinkedListインスタンスに対する逆方向のイテレータ'''
   
       def next(self):
           '''イテレータプロトコル
   
           collections.Iterator基底クラスの抽象メソッド
   
           最後まできた場合はStopIteration例外を発生させなければならない。
           詳しくはMyLinkedList.__reversed__特殊メソッドのコメント参照
           '''
           pass
   
       # python3から__next__特殊メソッドに変更された
       __next__ = next
   
   
   class Node(object):
       '''ノード'''
       def __init__(self, value):
           self.value = value
           self.next = None
           self.prev = None
   
       def __repr__(self):
           'printステートメントなどで呼ばれる'
           return repr(self.value)
   
   
   if __name__ == '__main__':
       import doctest
       doctest.testmod()

テストは次のように実行することができる。::

   $ python linkedlist.py
