# Step1

かかった時間：60min

計算量：
n: len(wordList), m: wordListに含まれるwordの長さ  
時間計算量：O(n^2*m)  
空間計算量：O(n)

```python
from collections import deque


class Solution:
    def ladderLength(self, beginWord: str, endWord: str, wordList: List[str]) -> int:
        def is_convertible(s1: str, s2: str) -> bool:
            num_diff = 0
            for c1, c2 in zip(s1, s2):
                if c1 != c2:
                    num_diff += 1
                if num_diff > 1:
                    return False
            return num_diff == 1

        candidates = deque([(beginWord, 1)])
        next_words = set(wordList)
        while candidates:
            word, num_transformations = candidates.popleft()
            if word == endWord:
                return num_transformations
            
            added_words = set()
            for next_word in next_words:
                if not is_convertible(word, next_word):
                    continue
                
                candidates.append((next_word, num_transformations + 1))
                added_words.add(next_word)
            next_words -= added_words
        
        return 0
```
思考ログ：
- 最短経路を求めたいのでBFSが良いか
- 今までの問題と違って次進む候補が決まっていない
  - 自分で候補を```wordList```から探す必要がある
  - 現在注目している文字と1文字違いの文字かどうか判定する関数を用意する
- 最初、毎回```wordList```から全ての文字を引っ張ってきて判定するようにしたらTLE
- よく考えると、候補として追加したものはその後もう確認しなくて良いのでは？
  - この改善でTLEは解消した

# Step2

講師役目線でのセルフツッコミポイント：
- 問題を解くのに集中してしまって、選択肢が狭まっている
  - 隣接リストくらい出てきてもいいんじゃないかと
- ```for c1, c2 in zip(s1, s2):```これ危ない
  - 問題文から```s1 == s2```が保証されているが、もし仮に長さが違ったとしたらどうなるか
    - これは長さが違ってもエラーが出ず普通に動いてしまうので結構問題がある
    - アサーションチェックくらいはした方がいいな、くらいの認識を持つ

参考にした過去ログなど：
- https://github.com/kazukiii/leetcode/pull/21
  - Dijkstra、最近書いてないから書いてみるか
  - 編集距離？
    - レーベンシュタイン距離で思い出した
      - https://ja.wikipedia.org/wiki/%E3%83%AC%E3%83%BC%E3%83%99%E3%83%B3%E3%82%B7%E3%83%A5%E3%82%BF%E3%82%A4%E3%83%B3%E8%B7%9D%E9%9B%A2
    - ハミング距離の一般化
      - https://ja.wikipedia.org/wiki/%E3%83%8F%E3%83%9F%E3%83%B3%E3%82%B0%E8%B7%9D%E9%9B%A2
        > リチャード・ハミング (Richard Wesley Hamming) にちなんで命名されたもので、鼻歌 (humming) ではない  
        > n 次元超立方体の 2 頂点間のマンハッタン距離
  - 参考
    - https://discord.com/channels/1084280443945353267/1183683738635346001/1199046289686548581
- https://github.com/fhiyo/leetcode/pull/22
  - ```for ch in ascii_lowercase:```の```ascii_lowercase```ってなんだ？
    - https://docs.python.org/ja/3/library/string.html
    - 文字列定数なるものがあるのか
      - ascii_letters = ascii_lowercase + ascii_uppercase
      - punctuation = !"#$%&'()*+,-./:;<=>?@[\]^_`{|}~.
      - printable =  digits + ascii_letters + punctuation + whitespace
  - 選択肢いろいろ
    - 一文字違いの候補をリストアップする方法
    - ハミング距離を計算する方法（step1でやったこともこれに相当する）
      - 隣接リストを作ってもいい
      - 計算量を減らす工夫
        - https://discord.com/channels/1084280443945353267/1200089668901937312/1216123084889788486
        - 比較する文字列の前半/後半パートをキーとして同値類を考える感じ
    - ワイルドカードを使って管理
      - ```'do*': ['dog', 'dot']```
- https://github.com/sakupan102/arai60-practice/pull/20
  - named tuple
    - https://docs.python.org/ja/3.12/library/collections.html#collections.namedtuple  
      > 名前付きタプルのインスタンスはインスタンスごとの辞書を持たないので、軽量で、普通のタプル以上のメモリを使用しません。
- https://github.com/hayashi-ay/leetcode/pull/42
  - lru_cache  
    - https://docs.python.org/ja/3/library/functools.html#functools.lru_cache
    - https://discord.com/channels/1084280443945353267/1200089668901937312/1206274586249793666
  - 双方向BFS
    - startとgoalの両者から探索していく、queueの中身が少ない方を優先して探索
    - 両側探索の速度面のメリットと実装コストの複雑さのデメリットを考える
- https://docs.google.com/document/d/16jyTV-AELKyMUu2_28027uR0JvapBBX1kCDD6rrmGOs/edit
- https://github.com/Yoshiki-Iwasa/Arai60/pull/22#pullrequestreview-2157900096
  
まずは隣接リストをやってみる
```python
from collections import defaultdict


class Solution:
    def ladderLength(self, beginWord: str, endWord: str, wordList: List[str]) -> int:
        def is_convertible(word1: str, word2: str) -> bool:
            assert len(word1) == len(word2)
            
            num_diff = 0
            for c1, c2 in zip(word1, word2):
                if c1 != c2:
                    num_diff += 1
                if num_diff > 1:
                    return False
            return num_diff == 1

        word_to_adjacent_words = defaultdict(list)
        for i in range(len(wordList) - 1):
            for j in range(i + 1, len(wordList)):
                word1, word2 = wordList[i], wordList[j]
                if is_convertible(word1, word2):
                    word_to_adjacent_words[word1].append(word2)
                    word_to_adjacent_words[word2].append(word1)
        
        words = [word for word in wordList if is_convertible(beginWord, word)]
        num_transformations = 1
        found_words = set()

        while words:
            next_words = []
            num_transformations += 1
            for word in words:
                found_words.add(word)
                if word == endWord:
                    return num_transformations
                next_words += [
                    next_word for next_word in word_to_adjacent_words[word] 
                    if next_word not in found_words
                ]
            words = next_words
        
        return 0
```
思考ログ：
- ギリギリ（Runtime 9242ms）
- 最近人のコードを見て、腹落ちした後に何も見ないで書いてみると、割と再現できるような感覚がある
  - ここは自分なりに書きたいなあ、というところは適宜変えているが、それも”ここはこう書いてなかったなあ”という意識がある

パターンと文字列の対応（同値類）を作る方法
```python
from collections import defaultdict


class Solution:
    def ladderLength(self, beginWord: str, endWord: str, wordList: List[str]) -> int:
        pattern_to_words = defaultdict(list)
        for word in wordList:
            for i in range(len(word)):
                pattern = word[:i] + '*' + word[i + 1:]
                pattern_to_words[pattern].append(word)
        
        found_words = set()

        def get_next_words(word: str) -> list[str]:
            next_words = []
            for i in range(len(word)):
                pattern = word[:i] + '*' + word[i + 1:]
                next_words += [
                    word for word in pattern_to_words[pattern]
                    if word not in found_words
                ]
            return next_words
        
        words = get_next_words(beginWord)
        num_transformations = 1

        while words:
            next_words = []
            num_transformations += 1
            for word in words:
                if word == endWord:
                    return num_transformations
                found_words.add(word)
                next_words += get_next_words(word)
            
            words = next_words
        
        return 0
```
思考ログ：
- fstring使えばよかったかな

両側探索
```python
class Solution:
    def ladderLength(self, beginWord: str, endWord: str, wordList: List[str]) -> int:
        def is_convertible(word1: str, word2: str):
            assert len(word1) == len(word2)
            
            num_diff = 0
            for c1, c2 in zip(word1, word2):
                if c1 != c2:
                    num_diff += 1
                if num_diff > 1:
                    return False
            return num_diff == 1

        def get_next_words(word: str) -> list[str]:
            return [
                next_word for next_word in wordList 
                if is_convertible(word, next_word)
            ]

        if endWord not in wordList:
            return 0
        if endWord in get_next_words(beginWord):
            return 2

        words_from_begin = get_next_words(beginWord)
        words_from_end = get_next_words(endWord)
        found_words_from_begin = set([beginWord])
        found_words_from_end = set([endWord])
        num_transformations = 1
        is_from_begin = True
        
        while words_from_begin and words_from_end:
            num_transformations += 1
            next_words = []
            if len(words_from_begin) < len(words_from_end):
                is_from_begin = True
            else:
                is_from_begin = False
            # is_from_begin = True

            if is_from_begin:    
                words = words_from_begin
                found_words = found_words_from_begin
                end_words = found_words_from_end
            else:
                words = words_from_end
                found_words = found_words_from_end
                end_words = found_words_from_begin

            for word in words:
                found_words.add(word)
                if word in end_words:
                    return num_transformations
                
                next_words += [
                    next_word for next_word in get_next_words(word)
                    if next_word not in found_words
                ]
            
            if is_from_begin:
                words_from_begin = next_words
                found_words_from_begin = found_words
            else:
                words_from_end = next_words
                found_words_from_end = found_words
        
        return 0
```
思考ログ：
- beginとend双方から辿っていく方法、次の文字の候補数が少ないものを優先して採用する
- 確かに片側よりは早い（Runtime 5951ms）
  - 見直してみたら、step1の方が早い（Runtime 4813ms）
- 一方で実装に手間取ったのと、読む側にもあまり親切なものではなさそう
  - 特殊ケースの排除が汚い（これは実装方法の問題だろう）
    ```python
    if endWord not in wordList:
        return 0
    if endWord in get_next_words(beginWord):
        return 2
    ```    
久々なのでダイクストラも書いておく
```python
import heapq
from collections import defaultdict


class Solution:
    def ladderLength(self, beginWord: str, endWord: str, wordList: List[str]) -> int:
        def is_convertible(word1: str, word2: str) -> bool:
            assert len(word1) == len(word2)

            num_diff = 0
            for c1, c2 in zip(word1, word2):
                if c1 != c2:
                    num_diff += 1
                if num_diff > 1:
                    return False
            return num_diff == 1

        word_to_dist_and_adjacent_words = defaultdict(list)
        for i in range(len(wordList) - 1):
            for j in range(i + 1, len(wordList)):
                if is_convertible(wordList[i], wordList[j]):
                    # (dist, adjacent_word) を追加
                    word_to_dist_and_adjacent_words[wordList[i]].append((1, wordList[j]))
                    word_to_dist_and_adjacent_words[wordList[j]].append((1, wordList[i]))
        
        if beginWord not in word_to_dist_and_adjacent_words:
            for i in range(len(wordList)):
                if is_convertible(beginWord, wordList[i]):
                    word_to_dist_and_adjacent_words[beginWord].append((1, wordList[i]))
                    word_to_dist_and_adjacent_words[wordList[i]].append((1, beginWord))

        candidates = [(1, beginWord)]
        found_words = set([beginWord])
        while candidates:
            total_dist, word = heapq.heappop(candidates)
            found_words.add(word)
            if word == endWord:
                return total_dist

            for next_dist, next_word in word_to_dist_and_adjacent_words[word]:
                if next_word in found_words:
                    continue
                heapq.heappush(candidates, (total_dist + next_dist, next_word))
        
        return 0
```
思考ログ：
- 久々のダイクストラ、普通にBFSすればいいと思うけどせっかくなので復習がてら記憶を辿って書いてみる
- ダイクストラ、隣接リストで処理していくことが多い気がするので今回も合わせて敢えて隣接リストを全てのエッジの重み1で登録
  - word間の距離は全部1なので今回この情報は無駄なのだが、ダイクストラっぽくするためにこうしている
- 考え方は確か、今まで探索した最短距離をfixして、そこから辿れるノードを調査、また最短距離をfixして、、、を繰り返す感じ
  - エッジの重みが非負という前提があるので上記のように最短距離を確定しながら進めることができる
  - 負のエッジがある場合はワーシャルフロイドとかあった（ここら辺から記憶が朧げ）
    - https://ja.wikipedia.org/wiki/%E3%83%AF%E3%83%BC%E3%82%B7%E3%83%A3%E3%83%AB%E2%80%93%E3%83%95%E3%83%AD%E3%82%A4%E3%83%89%E6%B3%95
- ちなみに（Runtime 9224ms）

# Step3

かかった時間：7min

```python
from collections import deque


class Solution:
    def ladderLength(self, beginWord: str, endWord: str, wordList: List[str]) -> int:
        def is_convertible(word1: str, word2: str) -> bool:
            assert len(word1) == len(word2)

            num_diff = 0
            for c1, c2 in zip(word1, word2):
                if c1 != c2:
                    num_diff += 1
                if num_diff > 1:
                    return False
            return num_diff == 1

        word_and_num_transformations = deque([(beginWord, 1)])
        candidate_words = set(wordList)
        while word_and_num_transformations:
            word, num_transformations = word_and_num_transformations.popleft()
            if word == endWord:
                return num_transformations

            added_words = set()
            for candidate_word in candidate_words:
                if not is_convertible(word, candidate_word):
                    continue
                added_words.add(candidate_word)
                word_and_num_transformations.append(
                    (candidate_word, num_transformations + 1)
                )

            candidate_words -= added_words

        return 0
```
思考ログ：
- なんだかんだで最後はstep1の解法で
- 疲れた

# Step4

```python
```
思考ログ：

