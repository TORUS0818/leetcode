# Step1

かかった時間：11min

計算量： ポストの数をNとして、

時間計算量：O(N)

空間計算量：O(N)

```python
class Solution:
    """
    @param n: non-negative integer, n posts
    @param k: non-negative integer, k colors
    @return: an integer, the total number of ways
    """
    def num_ways(self, n: int, k: int) -> int:
        # write your code here
        if n == 1:
            return k

        # num_ways_so_far[0, i]: i-1, iが同色となるi番目のポストまでの塗り方
        # num_ways_so_far[1, i]: i-1, iが異色となるi番目のポストまでの塗り方
        num_ways_so_far = [[0] * n for _ in range(2)]

        num_ways_so_far[0][1] = k
        num_ways_so_far[1][1] = k * (k - 1)
        for post_index in range(2, n):
            num_ways_so_far[0][post_index] = num_ways_so_far[1][post_index - 1]
            num_ways_so_far[1][post_index] = num_ways_so_far[0][post_index - 1] * (k - 1) \
                                           + num_ways_so_far[1][post_index - 1] * (k - 1)
        
        return num_ways_so_far[0][-1] + num_ways_so_far[1][-1]
```
思考ログ：
- 以下を漸化式に落として実装すれば良さそう
  - i番目のpostの時点での塗り方を次の2通りに分けて考える
    - パターン１：i-1とi番目が同色
    - パターン2：i-1とi番目が異色
  - パターン1が作りたい場合は、i-1番目のpostの時点でパターン2の塗り方になっていないといけない（パターン１では3連続同色になってしまう）
  - パターン2が作りたい場合は、i-1番目のpostの時点の塗り方のパターンはどちらでもよく、i-1番目のポストで塗った色以外の色を塗れる（k-1通り）

# Step2

講師役目線でのセルフツッコミポイント：
- ネーミング（num_ways_so_far）

参考にした過去ログなど：
- https://github.com/Yoshiki-Iwasa/Arai60/pull/44
  > あー、あと、この問題、行列の n 乗なので、そういう方法でも計算できるはずです。
  - これやってみよう  
- https://github.com/goto-untrapped/Arai60/pull/44
  > フィボナッチ数列に関連して、 時間計算量 O(log n) の解法は思いつきますか？
    - これは知らなかった
    - https://ja.wikipedia.org/wiki/%E3%83%95%E3%82%A3%E3%83%9C%E3%83%8A%E3%83%83%E3%83%81%E6%95%B0
      - 行列表現の結果をもとに漸化式を作る
      - q = n//2として、q-1, q, q+1番目のフィボナッチ数を用いてn番目のフィボナッチ数を表すことができる（lognに落とせる）
      - 漸化式に2nと2n-1を代入して確かめれば、そうなってるなあとなるが、、結構テクい式変形に感じた
  > 前と違う色を塗り分ける時は、1個前までの組み合わせ数×(k-1)通りで、前と同じ色を塗る時は2個前までの組み合わせ数×(k-1)×1通り、の和
    - これでよかった、step1のように余分に配列を用意しなくても良い
    - 最後が2連続同職の1個前までの組み合わせ数は、2個前までの組み合わせ数と同じ数になる性質を利用すればいいのか
  - あと再帰の実装を忘れていた
- https://github.com/sakupan102/arai60-practice/pull/31
  - メモ再帰
  - LRUキャッシュの実装
- https://github.com/shining-ai/leetcode/pull/30
  > 1つ前と2つ前のパターンだけ記憶しておく
    - 確かに配列で保持しておかなくてもよかった
- https://github.com/hayashi-ay/leetcode/pull/17

buttom-up DP
```python
class Solution:
    """
    @param n: non-negative integer, n posts
    @param k: non-negative integer, k colors
    @return: an integer, the total number of ways
    """
    def num_ways(self, n: int, k: int) -> int:
        # write your code here
        if n == 1:
            return k

        num_ways_so_far = [0] * n
        num_ways_so_far[0] = k
        num_ways_so_far[1] = k * k
        for i in range(2, n):
            num_ways_so_far[i] = (k - 1) * (num_ways_so_far[i　-　1] + num_ways_so_far[i　-　2])
        
        return num_ways_so_far[-1]
```
思考ログ：
- step1の改修
- 以下のように考える（前回と同じ、前回と異なるの2つに分ける）
  - 同異：前回のパターン*(k-1)
  - 同同：前前回のパターン*(k-1)*1

buttom-up DP（空間計算量O(1)）
```python
class Solution:
    """
    @param n: non-negative integer, n posts
    @param k: non-negative integer, k colors
    @return: an integer, the total number of ways
    """
    def num_ways(self, n: int, k: int) -> int:
        # write your code here
        if n == 1:
            return k
        if n == 2:
            return k * k
        
        num_ways = 0
        num_ways_two_steps_back = k
        num_ways_one_step_back = k * k
        for _ in range(2, n):
            num_ways = (k - 1) * (num_ways_one_step_back + num_ways_two_steps_back)
            num_ways_two_steps_back = num_ways_one_step_back
            num_ways_one_step_back = num_ways
        
        return num_ways
```
思考ログ：
- 配列にしないで変数を更新していく方法
- 名前が難しい
- ```n == 2```を特別扱いしなくても良いか

top-down DP
```python
from functools import cache


class Solution:
    """
    @param n: non-negative integer, n posts
    @param k: non-negative integer, k colors
    @return: an integer, the total number of ways
    """
    @cache
    def num_ways(self, n: int, k: int) -> int:
        # write your code here
        if n == 1:
            return k
        if n == 2:
            return k * k
        
        return (k - 1) * (self.num_ways(n - 1, k) + self.num_ways(n - 2, k))
```
思考ログ：
- cacheを使った方法
- lintcodeだと再帰上限に引っかかる（デフォルトの1000）
- 以前デコレータやLRU-Cacheの勉強のために書いた実装を見直しておこう

top-down DP （配列メモ）
```python
class Solution:
    """
    @param n: non-negative integer, n posts
    @param k: non-negative integer, k colors
    @return: an integer, the total number of ways
    """
        
    def num_ways(self, n: int, k: int) -> int:
        # write your code here
        memo = [[None] * (k + 1) for _ in range(n + 1)]

        def num_ways_helper(n: int, k: int):
            nonlocal memo

            if n == 1:
                return k
            if n == 2:
                return k * k
            if memo[n][k]:
                return memo[n][k]

            memo[n][k] =  (k - 1) * (num_ways_helper(n - 1, k) + num_ways_helper(n - 2, k))
            
            return memo[n][k]

        return num_ways_helper(n, k)
```
思考ログ：
- cacheを配列で取った版
- これもlintcodeだとメモリ使い過ぎと怒られる
  > Your code cost too much memory than we expected. Check your space complexity. Memory limit exceeded usually caused by you create a 2D-array which is unnecessary.  

行列の累乗 (おまけ)
```python
k = 2
n = 3

# 2 * 2の行列専用の行列積を取る関数
def multiply_2d_matrix(m1: list[list[float]], m2: list[list[float]]) -> list[list[float]]:
    assert len(m1) == len(m2) == 2
    assert len(m1[0]) == len(m2[0]) == 2

    return [
        [m1[0][0] * m2[0][0] + m1[0][1] * m2[1][0], m1[0][0] * m2[0][1] + m1[0][1] * m2[1][1]], 
        [m1[1][0] * m2[0][0] + m1[1][1] * m2[1][0], m1[1][0] * m2[0][1] + m1[1][1] * m2[1][1]]
    ]

assert n >= 3
assert k != 1
# nの時の塗り方をベクトルw_nとして纏めると（w_n = (前回と同色の塗り方, 前回と異色の塗り方)）
# w_n = T^(n-2) * w_2と表せる（T = [[0, 1], [k - 1, k - 1]]）
# 普通に累乗するとコストがかかるので対角化する

# 文字を置き直す
t = n - 2
a = k - 1
b = a * a + 4 * a
# 固有値計算
lam_1 = (a - b ** 0.5) / 2
lam_2 = (a + b ** 0.5) / 2
# 固有値を成分とする対角行列のt（=n-2）乗を計算
D_t = [[lam_1 ** t, 0], [0, lam_2 ** t]]
# 対角化行列の計算
P = [[-(b ** 0.5 + a) / (2 * a), (b ** 0.5 - a) / (2 * a)], [1, 1]]
# 対角化行列の逆行列の計算
c = -a / (b ** 0.5)
P_inv = [[c, c * (a - b ** 0.5) / (2 * a)], [-c, -c * (a + b ** 0.5) / (2 * a)]]
# E = multiply_2d_matrix(P, P_inv)
# print(E)

# 推移行列Tのt乗を計算
T_t = multiply_2d_matrix(P, D_t)
T_t = multiply_2d_matrix(T_t, P_inv)

# n = 2の場合の塗り方をt回推移させて最終的な塗り方の総数を計算する
# 無理やり2 * 2の行列演算に持っていく
num_ways_matrix = multiply_2d_matrix(
    T_t,
    [[0, k], [0, k * (k - 1)]]
)

# 0列目はダミー
print(num_ways_matrix[0][1] + num_ways_matrix[1][1])
```
思考ログ：
- https://discord.com/channels/1084280443945353267/1251052599294296114/1272143005851058229
  > あー、あと、この問題、行列の n 乗なので、そういう方法でも計算できるはずです。
- 漸化式から行列表記にして、n乗を計算する
- 普通に累乗するとコストが大きそうなので、対角化して累乗計算する
  - 計算ミスって時間が溶けた
- 小数丸めの問題で通らないサンプルもあるが、計算自体は良さそう
- floatが返ってくるのはご愛嬌、他にも変数名とか色々端折っている

# Step3

かかった時間：3min

```python
class Solution:
    """
    @param n: non-negative integer, n posts
    @param k: non-negative integer, k colors
    @return: an integer, the total number of ways
    """
    def num_ways(self, n: int, k: int) -> int:
        # write your code here
        if n == 1:
            return k
        
        num_ways_2steps_back = k
        num_ways_1step_back = k * k
        num_ways = num_ways_1step_back
        for _ in range(2, n):
            num_ways = (k - 1) * (num_ways_1step_back + num_ways_2steps_back)
            num_ways_2steps_back = num_ways_1step_back
            num_ways_1step_back = num_ways
        
        return num_ways
```
思考ログ：
- 無駄なメモリを使わないのが良いかなと
- 一方で、式の更新```num_ways = (k - 1) * (num_ways_1step_back + num_ways_2step_back)```は、個人的にはパッと思いつく感じではなかったなと思う
  - 特に”前回と同色となる塗り方”が前々回の塗り方に依存している、というところがポイントだったように思う
  - 前回同色を塗るためには => 前々回と前回が異色である必要がある => 前々回の塗り方 * (k-1)通り、というのがパッと思い浮かべば良いのだが 

# Step4

```python
```
思考ログ：

