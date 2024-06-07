# Step1

かかった時間：6min

計算量：

strs.length = N <br>
max(strs[i].length) = Mとして

時間計算量：O(N*MlogM)

空間計算量：O(N*M)

```python
from collections import defaultdict

class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        group_to_strs = defaultdict(list)
        for _str in strs:
            group_to_strs[tuple(sorted(_str))].append(_str)
        return list(group_to_strs.values())
```
思考ログ：
- GroupIDをソートされた文字列（のタプル）にして、該当する_strを放り込んでいく作戦

# Step2

講師役目線でのセルフツッコミポイント：

参考にした過去ログなど：
- https://github.com/fhiyo/leetcode/pull/15
  - trailing_underscoreについて
    - https://peps.python.org/pep-0008/#descriptive-naming-styles
  - 文字の度数分布をキーにする選択肢が見えてなかった
  - というより、自覚はあるがやはり見えている選択肢や考慮べき事項が少ない、と先達の方の過去ログを読んで毎回思う
    - 気をつけてはいるつもりだが、無意識下にacceptを目的にしているところがあるのだと思う、そのため何か一つ解決策が浮べば安心してしまう、みたいなメンタリティなんだと思う
- https://github.com/nittoco/leetcode/pull/13
- https://github.com/t-ooka/leetcode/pull/2#discussion_r1594808347
- https://github.com/SuperHotDogCat/coding-interview/pull/13
- https://github.com/sakupan102/arai60-practice/pull/13
- https://github.com/YukiMichishita/LeetCode/pull/5
- https://github.com/tshimosake/arai60/pull/4
- https://github.com/ryoooooory/LeetCode/pull/1
- https://github.com/hayashi-ay/leetcode/pull/19
  - 上でも話題に出ていたが、```dict.values```は```view object```を返す件
    - https://docs.python.org/3/library/stdtypes.html#dict-views
  - 問題の設定を少し変えて解いたりしてる、こういうのも積極的に考えてみると練習になるかも
    > 答えの配列に重複を許さない版。LeetCode上の想定とは異なるのでfailする

文字の度数分布をキーにして分類
```python
from collections import defaultdict


class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        def char_histogram(s: str):
            char_histogram = [0] * (ord('z') - ord('a') + 1)
            for c in s:
                char_histogram[ord(c) - ord('a')] += 1
            return tuple(char_histogram)

        group_key_to_anagrams = defaultdict(list)
        for s in strs:
            group_key_to_anagrams[char_histogram(s)].append(s)
        return list(group_key_to_anagrams.values())
```
思考ログ：
- ```char_histogram```の```tuple```を返すところは色々やり方がありそう
  - ```''.join(map(lambda t: chr(ord('a') + t[0]) * t[1], enumerate(counts)))```とか
    - https://github.com/fhiyo/leetcode/pull/15
- magic numberは、普通に定数を定義するといい
  - ```CHAR_NUM = 26```とか

itertools.groupbyを使ってみる
```python
from itertools import groupby
from collections import defaultdict

class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        group_key_to_anagrams = defaultdict(list)
        for key, group in groupby(strs, key=lambda s: ''.join(sorted(s))):
            group_key_to_anagrams[key] += list(group)
        return list(group_key_to_anagrams.values())
```
思考ログ：
- itertools.groupby  
  - https://docs.python.org/ja/3/library/itertools.html#itertools.groupby
  - なんか直感的な挙動と違うので注意が必要そう
    > key 関数の値が変わるたびに休止または新しいグループを生成します
    > ```[k for k, g in groupby('AAAABBBCCDAABBB')] → A B C D A B```
  
# Step3

かかった時間：2min

```python
from collections import defaultdict


class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        group_key_to_anagrams = defaultdict(list)
        for s in strs:
            group_key_to_anagrams[tuple(sorted(s))].append(s)
        
        return list(group_key_to_anagrams.values())
```
思考ログ：
- 特になし

# Step4

```python
```
思考ログ：

