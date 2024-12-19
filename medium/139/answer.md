# Step1

かかった時間：解けず

計算量： wordDict.length=N、s.length=Mとして

時間計算量：O(N*(N+M))

空間計算量：O(N)

```python
class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        candidates = [w for w in wordDict]
        found = set()
        while candidates:
            candidate = candidates.pop()
            if candidate == s:
                return True
            if not s.startswith(candidate):
                continue
            if candidate in found:
                continue
            found.add(candidate)
            for w in wordDict:
                candidates.append(candidate + w)
        
        return False
```
思考ログ：
- ```wordDict```の要素を組み合わせて```s```を作れないか調べる
- DFSで、最初候補のキャッシュを忘れてTLEになった
  - ```s = 'aaaaaaaaa...b'```で```wordDict = ['a', 'aa', 'aaa', ...]  ```のような場合に無駄が出ないようにする
- 計算量が自信がないが、、外のwhileでN、中はstartswithで舐めるのが最大M-1（Nの場合は上のifで弾かれている）最後のforループがN
- https://docs.python.org/ja/3/library/stdtypes.html#str.startswith  

# Step2

講師役目線でのセルフツッコミポイント：

参考にした過去ログなど：
- https://github.com/nittoco/leetcode/pull/42
  - AhoCorasick
    - https://ja.wikipedia.org/wiki/%E3%82%A8%E3%82%A4%E3%83%9B%E2%80%93%E3%82%B3%E3%83%A9%E3%82%B7%E3%83%83%E3%82%AF%E6%B3%95
      > 大まかに言えば、このアルゴリズムはトライ木を構築し、サフィックス木的に文字列（例えばabc）を表すノードからその最長サフィックス（接尾部、例えばbc）を表すノードがあればそれへリンクし、さもなくば c を表すノードがあればそれへリンクし、それもなければルートノードにリンクする。
      > このような最長サフィックスへのリンクを全てのノードが持っているため、そのリンクリストを辿ることで全てのマッチングを列挙できる。
      > 実行時にはトライ木として働き、最長のマッチングを探すと共に、サフィックスのリンクリストを活用することで計算量を線形に抑えている。
      > 辞書にあるノードに到達すると、その単語とそこからリンクされている辞書に含まれるノードが、マッチングした位置と共に出力される。
    - https://github.com/nittoco/leetcode/pull/42/files#r1886869885
    - failure_linkの構成部分の理解に時間がかかった
      - 親のfailure_linkを見ると、そこには親の最長サフィックスが入っている
      - 親の最長サフィックスの子供を調べると、子の最長サフィックスの候補を調べられる
      - これを再起的に調べていくイメージ
      - 例えば、abc（親） -> abcd（子）の時、親のfailure_linkがbc、bcのfailure_linkがcだとすると、まずbcdを調べる、次にcdを調べる、最後にdを調べる、という感じに処理すればいい
- https://github.com/Ryotaro25/leetcode_first60/pull/43
  - BFS  
    - ```s```を```wordDict```で分割できるか順に見ていく
    - 分割できた点を```proceeded```に記録していく、左端を起点にして一回舐めたら、次からは```proceeded```が```TRUE```の部分から見ていく
  - トライ木
- https://github.com/philip82148/leetcode-arai60/pull/8
  - トライ木
    - https://github.com/philip82148/leetcode-arai60/pull/8/files#r1864310221  
- https://github.com/frinfo702/software-engineering-association/pull/3
- https://github.com/Yoshiki-Iwasa/Arai60/pull/67
  - https://github.com/Yoshiki-Iwasa/Arai60/pull/67/files#r1778521580
    - これは私もテストケースで気づいた。。
- https://github.com/fhiyo/leetcode/pull/40
  > s[:i]までが分割出来ているとき、s[i:j]がwordDictに含まれていればs[:j]まで分割できる。
    - この表現が端的で分かりやすいと思った
  - min & max
    - https://docs.python.org/ja/3/library/functions.html#min
    - https://docs.python.org/ja/3/library/functions.html#max
      > default 引数は与えられたイテラブルが空の場合に返すオブジェクトを指定します。 イテラブルが空で default が与えられていない場合 ValueError が送出されます。
      > 最小の要素が複数あるとき、この関数はそのうち最初に現れたものを返します。
    - https://github.com/python/cpython/blob/main/Python/bltinmodule.c
- https://github.com/sakupan102/arai60-practice/pull/40
- https://github.com/SuperHotDogCat/coding-interview/pull/23
- https://github.com/goto-untrapped/Arai60/pull/20
- https://github.com/shining-ai/leetcode/pull/39
  - 正規表現
  - ローリングハッシュ
    - https://github.com/shining-ai/leetcode/pull/39/files
      - https://cp-algorithms.com/string/string-hashing.html
- https://github.com/hayashi-ay/leetcode/pull/61
  - priority-queueを使って、到達可能な出来るだけ遠いインデックスから探索していく
- https://github.com/Exzrgs/LeetCode/pull/10/files

DP
```python
class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        # breakable[i]: s[:i]がwordDictの要素で分解可能
        breakable = [False] * (len(s) + 1)
        breakable[0] = True
        for start in range(len(s)):
            if not breakable[start]:
                continue
            for word in wordDict:
                if s.startswith(word, start):
                    breakable[start + len(word)] = True
        
        return breakable[-1]
```
思考ログ：
- ```s[:i]```まで分割できることが記録されていた場合、```s[i:j]```∈```wordDict```なら```s[:j]```まで分解可
- 候補を調べた後、```s```が最後まで分解可能だったか調べる```breakable[-1]```
- なぜこの発想ができなかったか？
  - DPに対して、配列の連続した推移のイメージが強く付いている気がするのでもう少し抽象化して頭に入れておきたい
    - 例えば、```DP[i] = DP[i - 1] & f(i)```とか、```DP[i][j] = a * DP[i - 1][j] + b * DP[i][j - 1]```とか
    - 今回は、```leetcode is breakable? <- (leet is breakable) & (code in wordDict)```みたいなことを考えればいいが、頭には、```s[i] = s[i - 1] + f(i)```の方が浮かんでしまって思考ノイズになっている

再帰
```python
class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        @cache
        def word_break_helper(i: int) -> bool:
            if i == len(s):
                return True

            for word in wordDict:
                if (s.startswith(word, i)
                    and word_break_helper(i + len(word))):
                    return True

            return False
        
        return word_break_helper(0)
```
思考ログ：
- 再帰だといつも```hoge_helper```というのは良くないか
  - というより今回はこのままだと分かりにくいまである、参考にした@fhiyoさんの```breakable_from```とか良いかも
  - https://github.com/fhiyo/leetcode/pull/40/files

sの部分列がwordDictにないか確認していく方法（その1）
```python
class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        words = set(wordDict)
        # breakable[i]: s[:i]がwordDictの要素で分解可能
        breakable = [False] * (len(s) + 1)
        breakable[0] = True
        for start in range(len(s)):
            if not breakable[start]:
                continue
            for stop in range(start + 1, len(s) + 1):
                if s[start:stop] in words:
                    breakable[stop] = True
        
        return breakable[-1]
```
思考ログ：
- 今までの方法はwordがsに含まれていないか調べる方法、今回はsの部分列にwordがあるか調べる方法
- |s| <= 300, |words| <= 1,000, |word| <= 20、なのでsを舐めるのが早そう、スライスコピーのコストと併せて検討する
  - https://github.com/fhiyo/leetcode/pull/40/files

sの部分列がwordDictにないか確認していく方法（その2）
```python
class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        words = set(wordDict)
        word_min_len = min([len(word) for word in words])
        word_max_len = max([len(word) for word in words])

        breakable = [False] * (len(s) + 1)
        breakable[0] = True
        for start in range(len(s)):
            if not breakable[start]:
                continue
            for stop in range(start + word_min_len, start + word_max_len + 1):
                if stop > len(s):
                    break
                if s[start:stop] in words:
                    breakable[stop] = True
        
        return breakable[-1]
```
思考ログ：
- stopの範囲は別途計算しても良かったが、これはこれで意図が分かりやすいと思ったのだがどうだろう

トライ木
```python
from typing import Iterator


class TrieNode:
    def __init__(self):
        self.children: dict[str, TrieNode] = {}
        self.is_word: bool = False


class Trie:
    def __init__(self):
        self.root = TrieNode()
    
    def insert(self, word: str) -> None:
        node = self.root
        for c in word:
            if c not in node.children:
                node.children[c] = TrieNode()
            node = node.children[c]
        node.is_word = True
                
    def build(self, words: list[str]) -> None:
        for word in words:
            self.insert(word)
    
    def find_prefix_words(self, s: str) -> Iterator[str]:
        node = self.root
        prefix_words = []
        for c in s:
            if not c in node.children:
                return
            
            prefix_words.append(c)
            if node.children[c].is_word:
                yield ''.join(prefix_words)
            node = node.children[c]


class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        trie = Trie()
        trie.build(wordDict)

        @cache
        def word_break_helper(s: str) -> bool:
            if not s:
                return True
            for word in trie.find_prefix_words(s):
                if word_break_helper(s[len(word):]):
                    return True
            
            return False
        
        return word_break_helper(s)
```
思考ログ：
- トライ木にトライ
  - ちなみのそのトライではない
  - https://trja.wikipedia.org/wiki/%E3%83%88%E3%83%A9%E3%82%A4_(%E3%83%87%E3%83%BC%E3%82%BF%E6%A7%8B%E9%80%A0)
    > trie という名称は "retrieval"（探索、検索）が語源

正規表現(TLE)
```python
class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        pattern = f"^({'|'.join(map(re.escape, wordDict))})*$"
        regex = re.compile(pattern)

        return True if regex.fullmatch(s) else False
```
思考ログ：


# Step3

かかった時間：3min

```python
class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        # s[:i]が分解可能か管理する配列
        breakable = [False] * (len(s) + 1)
        breakable[0] = True

        for start in range(len(s)):
            if not breakable[start]:
                continue
            for word in wordDict:
                if s.startswith(word, start):
                    breakable[start + len(word)] = True
        
        return breakable[-1]
```
思考ログ：
- DPの考え方ができていなかったので練習も兼ねて
- アプローチとしても自然な気もする

# Step4

```python
```
思考ログ：
