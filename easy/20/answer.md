# Step1

かかった時間：32min

計算量：
sの長さをNとして
時間計算量：O(N) 空間計算量：O(N)

```python
from collections import deque

class Solution:
    def isValid(self, s: str) -> bool:
        parentheses_pairs = {
            '(': ')',
            '[': ']',
            '{': '}'
        }
        parentheses_stack = deque()
        for s_i in s:
            if s_i in ['(', '[', '{']:
                parentheses_stack.append(s_i)
                continue
            if not parentheses_stack:
                return False
            s_j = parentheses_stack.pop()
            if parentheses_pairs[s_j] != s_i:
                return False
    
        return False if parentheses_stack else True
```
思考ログ：
- まず問題文を読み間違える
  - ```([)]```こういうのもOKだと思って実装を始める、、
- 脱線から復帰（15分ほどロス、、）
- 方針としては、
  - 左括弧を貯めておくスタックを用意して、sを順にチェックしていく
    - 左括弧が来たらスタックに入れる
    - 右括弧が来たらスタックから取り出したものと比較する
      - そもそも取り出すものがなかったらアウト
      - 正しいペアでなかったらアウト
      - 正しいペアだったらさらにsの検査を進める
    - 最後、スタックに何か残ってたらアウト、そうでなければ合格
- あんまり思考の整理ができてない状態でコードを書いてる、、
  - ”そもそも取り出すものがなかったらアウト”を忘れていて、モグラ叩きのようにデバッグしながら追加、、
- 命名も適当になってしまった、s_iとかs_j

# Step2

講師役目線でのセルフツッコミポイント：
- 変数名について（s_i, s_j）
- dequeについて調べよう
  - https://docs.python.org/3/library/collections.html#collections.deque  
  - https://github.com/python/cpython/blob/1b293b60067f6f4a95984d064ce0f6b6d34c1216/Modules/_collectionsmodule.c
    - 実装をチラ見
    - linked-listで固定長配列が繋がったもの

参考にした過去ログなど：
- https://github.com/sakzk/leetcode/pull/6#discussion_r1595460679
  - dictionary, dequeの調査など
- https://github.com/sakupan102/arai60-practice/pull/7#discussion_r1584540368
  - 変数命名の参考
- https://github.com/Mike0121/LeetCode/pull/2
  - close-bracketがスタックに入った時点での早期終了について
- https://github.com/tayzarnw/LeetCode/pull/7
  - 変数命名の参考
- https://github.com/rm3as/code_interview/pull/4
- https://github.com/rossy0213/leetcode/pull/7
- https://github.com/sakupan102/arai60-practice/pull/7#pullrequestreview-1999411452
  - プッシュダウンオートマトンの話
- https://github.com/cheeseNA/leetcode/pull/10#pullrequestreview-1965188602
  - プッシュダウンオートマトン、チョムスキー階層の話がある
- https://github.com/Kitaken0107/GrindEasy/pull/5
- https://github.com/colorbox/leetcode/pull/4#discussion_r1523522386
  - チョムスキー階層、type-2、文脈自由文法、プッシュダウンオートマトン
  - - https://ja.wikipedia.org/wiki/%E3%83%81%E3%83%A7%E3%83%A0%E3%82%B9%E3%82%AD%E3%83%BC%E9%9A%8E%E5%B1%A4
- https://github.com/hayashi-ay/leetcode/pull/16

```python
class Solution:
    def isValid(self, s: str) -> bool:
        brackets_stack = []
        close_to_open = {
            ')': '(',
            ']': '[',
            '}': '{'
        }
        open_brackets = list(close_to_open.values())
        for char in s:
            if char in open_brackets:
                brackets_stack.append(char)
                continue
            if not brackets_stack:
                return False
            if close_to_open[char] != brackets_stack.pop(-1):
                return False
        return not brackets_stack
```
思考ログ：
- まず変数名から直していく
  - ```parentheses_stack``` -> ```brackets_stack```
  - ```parentheses_pairs``` -> ```close_to_open```
  - ```s_i``` -> ```char```
    - また、Step1の```s_j```のように無理に変数に入れなくても直接比較してしまった方がわかりやすい時もあるかも
- データ構造について
  - https://github.com/sakzk/leetcode/pull/6#discussion_r1595460679
  - 上記でも議論があったが、なぜ様々な選択肢から```deque```を選んだか？を考えること
  - 脳死で、stack->dequeのような判断は止める

# Step3

かかった時間：3min

```python
class Solution:
    def isValid(self, s: str) -> bool:
        brackets_stack = list()
        close_to_open = dict((')(', '}{', ']['))
        for char in s:
            if char in close_to_open.values():
                brackets_stack.append(char)
                continue
            if not brackets_stack:
                return False
            if close_to_open[char] != brackets_stack.pop(-1):
                return False
        return not brackets_stack
```
思考ログ：
- 普通に書いた方が分かりやすいと思うが、```dict((')(', '}{', ']['))```みたいな書き方もできる
- Step2のように一旦```open_brackets```に入れた方が分かりやすい気もするが

# Step4

```python
```
思考ログ：
