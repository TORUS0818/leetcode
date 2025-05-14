# Step1

かかった時間：15min

計算量：

時間計算量：O(n)

空間計算量：O(n)

```python
class Solution:
    def kthGrammar(self, n: int, k: int) -> int:
        row_len = 2 ** (n - 1)
        assert k <= row_len
        if row_len == 1:
            return 0
        
        if k <= row_len // 2:
            return self.kthGrammar(n-1, k)
        else:
            return 1 - self.kthGrammar(n-1, k - row_len // 2)
```
思考ログ：
- row(t)とrow(t+1)の関係性について
  - row(t+1) = row(t) + row(t)^c
    - s^cはsを反転させたものとする
- 上記を踏まえると、len(row(t+1)) = 2*len(row(t))に注意して
  - k <= len(row(t))なら、row(t+1)のkは、row(t)のkと同じ
  - k > len(row(t))なら、row(t+1)のkは、row(t)のk-len(row(t))を反転させたものと同じ
- ところでこの関係性ってtrivialなものなのか？？
  - 不安になって帰納法で確認したが、なんか考え方が不自然なのかしら

# Step2

講師役目線でのセルフツッコミポイント：

参考にした過去ログなど：
- https://github.com/hroc135/leetcode/pull/44
  - 二分木は頭にあってもよかった
  - bits.OnesCountの内部実装
  - popcnt
  - SWAR Algorithm
- https://github.com/olsen-blue/Arai60/pull/47
  - 2分木を考えてkの偶奇で親の値と同じになるかを判定
  - https://github.com/olsen-blue/Arai60/pull/47/files#r2002307405
    - これは思いつかないが知っておこう
- https://github.com/Ryotaro25/leetcode_first60/pull/49
  - https://github.com/Ryotaro25/leetcode_first60/pull/49/files#r1894794397
  - https://github.com/Ryotaro25/leetcode_first60/pull/49/files#r1894838721
    - Brian Kernighan's Algorithmあたりの話？
- https://github.com/Yoshiki-Iwasa/Arai60/pull/39
- https://github.com/yukik8/leetcode/commit/a0e14f3fbe34720d13fcb61916c99d2a424b7bc6
- https://github.com/nittoco/leetcode/pull/29
  - bitcount
    - 初手コレの人もいるのか
  - https://github.com/nittoco/leetcode/pull/29/files#r1685749723
- https://github.com/fhiyo/leetcode/pull/47
  - nではなくkを削っていってもいいのか
- https://github.com/goto-untrapped/Arai60/pull/26
- https://github.com/SuperHotDogCat/coding-interview/pull/26
- https://github.com/Mike0121/LeetCode/pull/18
  - https://github.com/Mike0121/LeetCode/pull/18/files#r1613819411
    - nではなくkを追う
    - https://docs.python.org/ja/3/library/stdtypes.html#int.bit_count
- https://github.com/shining-ai/leetcode/pull/46
  - https://github.com/shining-ai/leetcode/pull/46/files#r1557731428
    - int.bit_count()
      > 整数の絶対値の二進数表現における 1 の数
- https://github.com/hayashi-ay/leetcode/pull/46

2分木（再帰）
```python
class Solution:
    def kthGrammar(self, n: int, k: int) -> int:
        if k == 1:
            return 0
            
        if k % 2:
            return self.kthGrammar(n - 1, (k + 1) // 2)
        else:
            return 1 - self.kthGrammar(n - 1, (k + 1) // 2)
```
思考ログ：
- 各symbolを二分木のノードとして捉えて
  - 0
  - 01
  - 0110
  - 01101001
- kの偶奇と子ノードの対応は
  - kが奇数:左の子
  - kが偶数:右の子
- 右の子からその親に辿る時にsymbolが反転（0<->1）することに注意すると
  - kが偶数 => 反転、kが奇数 => そのまま
  - k -> (k + 1) // 2として一つ前の行のインデックスに更新
  - これを繰り返していけばいい
- 終了条件をnではなくkに変更してみた

2分木（loop）
```python
class Solution:
    def kthGrammar(self, n: int, k: int) -> int:
        symbol = 0
        while k > 1:
            if (k + 1) & 1:
                symbol ^= 1 
            k = (k + 1) >> 1
        
        return symbol
```
思考ログ：
- 上の再帰をループに
- kを0-indexedにした方が綺麗に書けるが、kの意味を変えたくなかったのでこのままで実装

pop count
```python
class Solution:
    def kthGrammar(self, n: int, k: int) -> int:
        return int.bit_count(k-1) % 2
```
思考ログ：
- 2分木をもう少し考察すると、k-1のbit-countをすればいいことが分かる
  - kを0-indexedにする（k = k-1と置き直す）
  - 最下位ビットの偶奇チェック、0-indexedの場合、奇数なら反転することに注意
  - 1右にシフトさせて（前の行のインデックスに変換して）同様のチェックを根まで繰り返す
  - ビットの数=反転の数
  - 反転が偶数回なら0、奇数回なら1を返せばいい

# Step3

かかった時間：2min

```python
class Solution:
    def kthGrammar(self, n: int, k: int) -> int:
        if k == 1:
            return 0

        if k % 2:
            return self.kthGrammar(n - 1, (k + 1) // 2)
        else:
            return 1 - self.kthGrammar(n - 1, (k + 1) // 2)
```
思考ログ：
- 2分木を想起出来なかったので反省として
- ループか再帰かは個人的な好みで再帰の方が分かりやすかったため

# Step4

```python
```
思考ログ：

