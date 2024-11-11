# Step1

かかった時間：9min

計算量：gridのマスの数をN（m * n）として

時間計算量：O(N)

空間計算量：O(N)

DP
```python
class Solution:
    def uniquePaths(self, m: int, n: int) -> int:
        # 壁際のマスへの行き方は1通りしかないのでgridの初期化の際に1で埋める
        grid = []
        for row_i in range(m):
            if row_i == 0:
                grid.append([1] * n)
                continue
            grid.append([1] + [0] * (n - 1))

        for row_i in range(1, m):
            for col_i in range(1, n):
                grid[row_i][col_i] = grid[row_i - 1][col_i] + grid[row_i][col_i - 1]

        return grid[m - 1][n - 1]
```
思考ログ：
- 小学生の時にやったマスの上と左に1を書いて足していくやつ
- 問題文につられて```grid```にしたが、```num_ways```とかにした方が情報があって良かったか
  - ある座標[i, j]への行き方の総数を記録した二次元配列
- ```grid```の作成と壁際の初期化を同時にやってしまったが、冗長でも分けても良かったかもしれない

組み合わせを計算
```python
from scipy.special import comb


class Solution:
    def uniquePaths(self, m: int, n: int) -> int:
        return comb(m - 1 + n - 1, n - 1, exact=True)
```
思考ログ：
- 組み合わせを計算する
  - 例えば、m = 3, n = 3の場合
    - (↓, ↓, →, →)の並べ方を考えればいい（=4C2）
- scipy.comb
  - https://docs.scipy.org/doc/scipy/reference/generated/scipy.special.comb.html
  - https://github.com/scipy/scipy/blob/v1.14.1/scipy/special/_basic.py#L2681-L2753
  - n-1とm-1の小さい方を入れた方が効率がいいかなと思ったが、実装を見たら（_comb_intの中で）やってるようなので気にしないことに
    - https://github.com/scipy/scipy/blob/70455fcdc8c0737ac24b592c74083837ca6fbf72/scipy/special/_comb.pyx#L5

# Step2

講師役目線でのセルフツッコミポイント：

参考にした過去ログなど：
- https://github.com/Ryotaro25/leetcode_first60/pull/39
- https://github.com/seal-azarashi/leetcode/pull/31
- https://github.com/Yoshiki-Iwasa/Arai60/pull/47
- https://github.com/nittoco/leetcode/pull/26
  - 再帰
  - mathモジュールの話題
    - https://github.com/nittoco/leetcode/pull/26/files#r1677082520
- https://github.com/fhiyo/leetcode/pull/34
  - 任意精度演算の話
    - https://zenn.dev/yukinarit/articles/afb263bf68fff2
- https://github.com/goto-untrapped/Arai60/pull/32
- https://github.com/YukiMichishita/LeetCode/pull/14
- https://github.com/sakupan102/arai60-practice/pull/34
  - combinationsの再帰（というか分解）
- https://github.com/SuperHotDogCat/coding-interview/pull/16
- https://github.com/shining-ai/leetcode/pull/32
- https://github.com/hayashi-ay/leetcode/pull/39
  - 末尾再帰

DPの配列を圧縮（空間計算量：O(min(n, m))）
```python
class Solution:
    def uniquePaths(self, m: int, n: int) -> int:
        if n < m:
            n, m = m, n

        num_ways = [1] * n
        for row_i in range(1, m):
            for col_i in range(1, n):
                num_ways[col_i] = num_ways[col_i] + num_ways[col_i - 1]
        
        return num_ways[-1]
```
思考ログ：
- 確かに上書きしていけば二次元配列は必要ない
- n, mの小さい方で配列を確保すればよりエコ
  - そこまで気にする必要はないかもしれないが、追加の処理も少ないのでやっておいてもいいのではという気持ち

再帰
```python
from functools import cache


class Solution:
    @ cache
    def uniquePaths(self, m: int, n: int) -> int:
        if m == 1 or n == 1:
            return 1
            
        return self.uniquePaths(m - 1, n) + self.uniquePaths(m, n - 1)
```
- cacheを取っておかないと厳しい
- ただcacheを取れば万事解決というわけでもないのでちゃんと考えて選択する
  - https://github.com/fhiyo/leetcode/pull/34/files

# Step3

かかった時間：2min

```python
class Solution:
    def uniquePaths(self, m: int, n: int) -> int:
        # メモリ節約のため、uniqe path数を管理する配列の長さをmin(n, m)で確保する
        if m < n:
            n, m = m, n

        num_ways = [1] * n
        for _ in range(1, m):
            for i in range(1, n):
                num_ways[i] += num_ways[i - 1]
        
        return num_ways[-1]
```
思考ログ：
- 変数名などを修正
- いきなりn, mのスワップが始まるとアレかと思いコメントを入れてみたが、もう少しいい説明があったか

# Step4

```python
```
思考ログ：

