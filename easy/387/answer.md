# Step1

かかった時間：4min

計算量：
s.length = Nとして

時間計算量：O(N)

空間計算量：O(N)

```python
from collections import defaultdict


class Solution:
    def firstUniqChar(self, s: str) -> int:
        word_to_count = defaultdict(int)
        for c in s:
            word_to_count[c] += 1
        for index, c in enumerate(s):
            if word_to_count[c] == 1:
                return index

        return -1
```
思考ログ：
- 文字ごとの出現回数を記録して、再度先頭から舐めて重複がないものを探す
- 2回舐めてるのが無駄な気がするが、1回で済ませる方法が思い浮かばない、、

# Step2

講師役目線でのセルフツッコミポイント：
- 1-passにできるか

参考にした過去ログなど：
- https://github.com/fhiyo/leetcode/pull/18#discussion_r1629829157
  - counter 
    - 実質辞書と変わらなそうだが、この際にドキュメント読んどく（以前読んだ気もする）
    - https://docs.python.org/3/library/collections.html#collections.Counter  
      > カウンタオブジェクトは辞書のインターフェースを持ちますが、存在しない要素に対して KeyError を送出する代わりに 0 を返すという違いがあります  
      
      > カウンタオブジェクトは等価、部分集合、上位集合のための次の拡張比較 (rich comparison) 演算子をサポートします: ==, !=, <, <=, >, >=。これらの比較は、存在しない要素をカウントがゼロであるとみなします。すなわち、 Counter(a=1) == Counter(a=1, b=0) は真を返します。
  - ```char_to_count```より```char_to_frequency```の方がしっくりくるかも
  - 計算量O(N^2)の方法は確かに考えたがあまりメリットがなさそうで選択しなかった
  - defaultdict
  - 自然言語で説明すると複雑な変数は分けましょう。私は、grue や bleen みたいなものだと思っています。
    - https://en.wikipedia.org/wiki/New_riddle_of_induction
  - 1passの解法、だけど結局min取ってるのでそんなに優位性はないかと思いきや
  - OrderedDict
    - https://docs.python.org/ja/3/library/collections.html#collections.OrderedDict
      > 組み込みの dict クラスが挿入順序を記憶しておく機能 (この新しい振る舞いは Python 3.7 で保証されるようになりました) を獲得した今となっては、順序付き辞書の重要性は薄れました。
- https://github.com/su33331/practice/pull/2#discussion_r1616123515
  - 26文字char配列を使う方法
  - defaultdictの代わりに
    ```python
    for c in s:
        character_count[c] = character_count.get(c, 0) + 1
    ```
    - でも速くはない
      - https://takeg.hatenadiary.jp/entry/2019/06/02/181647
- https://github.com/SuperHotDogCat/coding-interview/pull/22
  - そういえば```iter()```のドキュメントを読んだことがない
    - https://docs.python.org/ja/3/library/functions.html#iter
    - https://docs.python.org/ja/3/library/stdtypes.html#typeiter
- https://github.com/goto-untrapped/Arai60/pull/17/files
- https://github.com/sakupan102/arai60-practice/compare/dc300ec4b8c1...bb4d99dfb732
- https://github.com/hayashi-ay/leetcode/pull/28/files
  > 半年後に読んだ別の同僚が不安にならないか、環境の変化に対して頑健か、なのです。
- https://github.com/shining-ai/leetcode/pull/15
  - LinkedHashMap
    - ```value```を```ListNode```にしちゃって```key```のつながりを管理する
    - OrderedDict
      - https://github.com/python/cpython/blob/main/Lib/collections/__init__.py

1-passの方法 その1
```python
class Solution:
    def firstUniqChar(self, s: str) -> int:
        char_to_index = {}
        seen = set()
        for i, c in enumerate(s):
            if c in seen:
                continue
            if c in char_to_index:
                char_to_index.pop(c)
                seen.add(c)
                continue
            char_to_index[c] = i

        if not char_to_index:
            return -1

        return min(char_to_index.values())
```
思考ログ：
- Step1の実装より早い感じ（leetcode上で何回か計っただけなのでアレだが、半分くらい？）
- 重複があった時早いのと、minが早いのか？
- あと、辞書の順序が保持されていることを利用すればminなしで書ける（python3.7~）
  ```python
  return next(iter(char_to_index.values()))
  ```

1-passの方法 その2
```python
class Solution:
    def firstUniqChar(self, s: str) -> int:
        INVALID_INDEX = 10**6
        char_to_index = {}
        for i, c in enumerate(s):
            if c in char_to_index:
                char_to_index[c] = INVALID_INDEX
                continue
            char_to_index[c] = i
        
        first_unique_character_index = min(char_to_index.values())
        if first_unique_character_index == INVALID_INDEX:
            return -1

        return first_unique_character_index
```
思考ログ：
- 一応一つの辞書で全部やってしまう方式でも実装してみた
- 最初はなるほどなあと感心したが（確かに上手いのだが）確かに変数名がつけにくい（上の実装では良いものが思いつかなかったので半ば諦めている）
  > 自然言語で説明すると複雑な変数は分けましょう。私は、grue や bleen みたいなものだと思っています。  
  > https://en.wikipedia.org/wiki/New_riddle_of_induction
  - まとめて済ませてしまいたい気持ちにはなるが、（命名も含め）実装に不自然さを感じたら上記のことを思い出したい
- 無効なインデックスに何を使うかという議論もあった
  > 2回以上出現した際の値のマーカーとしてはlen(s)以上の値なら何でもよいのかと思いますが、特定の数値だとその数値に何か意味があるのかなと思ってしまいそうだなと感じました。min(a,b)の単位元がinfなので個人的にはinfにするのがわかりやすいと思いました。
  - ただ、自分もfloatを混在させたくないなあという気持ちがあり、取りうる範囲外の大きな値を設定した
  - しかし、しっくりくるわけでもない、そしてこれも上記の話に繋がるのだろう
- 実行速度的には、1-passの方法 その1よりは遅くStep1よりは早い感じ
  - 今回は重複があってもminを取ったあとだったので若干遅くなっているのか

配列で分布を記録する方法
```python
class Solution:
    def firstUniqChar(self, s: str) -> int:
        LETTERS_NUM = 26
        letters_histogram = [0] * LETTERS_NUM
        for c in s:
            letters_histogram[ord(c) - ord('a')] += 1
        
        for i, c in enumerate(s):
            if letters_histogram[ord(c) - ord('a')] == 1:
                return i

        return -1
```
思考ログ：
- ```LETTERS_NUM```がしっくりこないが

# Step3

かかった時間：4min

```python
class Solution:
    def firstUniqChar(self, s: str) -> int:
        char_to_index = {}
        seen = set()
        for i, c in enumerate(s):
            if c in seen:
                continue
            if c in char_to_index:
                char_to_index.pop(c)
                seen.add(c)
                continue
            char_to_index[c] = i

        if not char_to_index:
            return -1
        
        return next(iter(char_to_index.values()))
```
思考ログ：
- 色々勉強になったので1-passの方法で

# Step4

```python
```
思考ログ：

