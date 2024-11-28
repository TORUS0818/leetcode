# Step1

かかった時間：9min

計算量：
obstacleGrid.length = M
obstacleGrid[i].length = Nとして、

時間計算量：O(M*N)

空間計算量：O(M*N)

DP
```python
class Solution:
    def uniquePathsWithObstacles(self, obstacleGrid: List[List[int]]) -> int:
        m = len(obstacleGrid)
        n = len(obstacleGrid[0])
        num_ways = [[0] * n for _ in range(m)]
        
        if obstacleGrid[0][0] == 1\
            or obstacleGrid[-1][-1] == 1: 
            return 0
        
        num_ways[0][0] = 1
        for row in range(m):
            for col in range(n):
                if row == 0 and col == 0:
                    continue
                if obstacleGrid[row][col] == 1:
                    continue
                if row == 0:
                    num_ways[row][col] = num_ways[row][col - 1]
                    continue
                if col == 0:
                    num_ways[row][col] = num_ways[row - 1][col]
                    continue
                num_ways[row][col] = (
                    num_ways[row][col - 1] + num_ways[row - 1][col]
                )
        
        return num_ways[-1][-1]
```
思考ログ：
- 前回のUniquePathsIのやり方を継承しつつ、obstacleGridを参照しながら行けない場所を考慮できるようにする
- ばーっと書いてしまったが、for文の中の処理を切り出して見通しをよくしたい（初手これを書いたので一応載せている）

上を少しまとめた
```python
class Solution:
    def uniquePathsWithObstacles(self, obstacleGrid: List[List[int]]) -> int:
        def _count_unique_paths(row: int, col: int) -> int:
            # gridのセル(row, col)までのパス数を数える
            if row == 0:
                return num_ways[row][col - 1]
            if col == 0:
                return num_ways[row - 1][col]
            
            return num_ways[row][col - 1] + num_ways[row - 1][col]

        m = len(obstacleGrid)
        n = len(obstacleGrid[0])
        num_ways = [[0] * n for _ in range(m)]
        
        if obstacleGrid[0][0] == 1\
            or obstacleGrid[-1][-1] == 1: 
            return 0
        
        num_ways[0][0] = 1
        for row in range(m):
            for col in range(n):
                if row == 0 and col == 0:
                    continue
                if obstacleGrid[row][col] == 1:
                    continue
                num_ways[row][col] = _count_unique_paths(row, col)
        
        return num_ways[-1][-1]
```
思考ログ：

再帰
```python
class Solution:
    def uniquePathsWithObstacles(self, obstacleGrid: List[List[int]]) -> int:
        @cache
        def _count_unique_paths(row: int, col: int) -> int:
            assert row >= 0 or col >= 0, 'row and col must be positive integer values.'

            if obstacleGrid[row][col] == 1:
                return 0
            if row == 0 and col == 0:
                return 1
            if row == 0:
                return _count_unique_paths(row, col - 1)
            if col == 0:
                return _count_unique_paths(row - 1, col)

            return _count_unique_paths(row - 1, col)\
                + _count_unique_paths(row, col - 1)
        
        return _count_unique_paths(
            len(obstacleGrid) - 1,
            len(obstacleGrid[0]) - 1
```
思考ログ：

1-DP
```python
class Solution:
    def uniquePathsWithObstacles(self, obstacleGrid: List[List[int]]) -> int:
        # 11:59
        def _check_if_valid_cell(row: int, col: int, is_transposed: bool) -> bool:
            if is_transposed:
                return obstacleGrid[col][row] == 0
            else:
                return obstacleGrid[row][col] == 0

        if obstacleGrid[0][0] == 1\
            or obstacleGrid[-1][-1] == 1: 
            return 0

        m = len(obstacleGrid)
        n = len(obstacleGrid[0])
        is_transposed = False
        if m < n:
            n, m = m, n
            is_transposed = True
        
        num_ways = [0] * n
        num_ways[0] = 1
        for row in range(m):
            for col in range(n):
                if _check_if_valid_cell(row, col, is_transposed):
                    if col == 0:
                        continue
                    num_ways[col] += num_ways[col - 1]
                else:
                    num_ways[col] = 0
        
        return num_ways[-1]
```
思考ログ：
- メモリ節約ver
- 今回は、n, mを入れ替えた時に参照するobstacleGridにも影響が及ぶ
  - これも転置しちゃえば良いのだが、破壊的な変更をしたくないし、転置した情報を別に用意したらこれはこれで本末転倒なことに
  - 転置フラグで管理するようにしたが、どうだろうか
- ```num_ways```の0番目の要素の扱いで少しハマった
  - ```num_ways```がどんな要素を持つ配列なのか、それは意図したものなのか、確認がまだ甘い
  - colのループの前で```col = 0```だけ処理してしまってもいいが、うーん

# Step2

講師役目線でのセルフツッコミポイント：
- マジックナンバー（obstacle）

参考にした過去ログなど：
- https://github.com/Ryotaro25/leetcode_first60/pull/40
- https://github.com/seal-azarashi/leetcode/pull/32
- https://github.com/Yoshiki-Iwasa/Arai60/pull/49
  - BFS
- https://github.com/nittoco/leetcode/pull/27
- https://github.com/fhiyo/leetcode/pull/35
- https://github.com/goto-untrapped/Arai60/pull/33
  - 以下の条件のまとめ方が頭になかった  
    - https://github.com/goto-untrapped/Arai60/pull/33/files#r1665011732
- https://github.com/YukiMichishita/LeetCode/pull/15
- https://github.com/sakupan102/arai60-practice/pull/35
  - シンプルな1DP
- https://github.com/SuperHotDogCat/coding-interview/pull/17
- https://github.com/shining-ai/leetcode/pull/34
- https://github.com/hayashi-ay/leetcode/pull/44
  - 値を受け取る、渡す  

DP
```python
class Solution:
    def uniquePathsWithObstacles(self, obstacleGrid: List[List[int]]) -> int:
        OBSTACLE = 1
        if obstacleGrid[0][0] == OBSTACLE \
            or obstacleGrid[-1][-1] == OBSTACLE:
            return 0
        
        m = len(obstacleGrid)
        n = len(obstacleGrid[0])
        num_ways = [[0] * n for _ in range(m)]
        num_ways[0][0] = 1
        for row in range(m):
            for col in range(n):
                if row == 0 and col == 0:
                    continue
                if obstacleGrid[row][col] == OBSTACLE:
                    continue
                if row > 0:
                    num_ways[row][col] += num_ways[row - 1][col]
                if col > 0:
                    num_ways[row][col] += num_ways[row][col - 1]
        
        return num_ways[-1][-1]
```
思考ログ：
- 少し処理の順序を変えた
- マジックナンバーの解消
- DP受け取りの分岐を変更

DP、事前に初期化
```python
class Solution:
    def uniquePathsWithObstacles(self, obstacleGrid: List[List[int]]) -> int:
        OBSTACLE = 1
        if obstacleGrid[0][0] == OBSTACLE \
            or obstacleGrid[-1][-1] == OBSTACLE:
            return 0
        
        m = len(obstacleGrid)
        n = len(obstacleGrid[0])
        def initialize_num_ways() -> list[list[int]]:
            num_ways = [[0] * n for _ in range(m)]
            for row in range(m):
                if obstacleGrid[row][0] == OBSTACLE:
                    break
                num_ways[row][0] = 1
            for col in range(n):
                if obstacleGrid[0][col] == OBSTACLE:
                    break
                num_ways[0][col] = 1
            
            return num_ways

        num_ways = initialize_num_ways()
        for row in range(1, m):
            for col in range(1, n):
                if obstacleGrid[row][col] == OBSTACLE:
                    continue
                num_ways[row][col] = num_ways[row - 1][col] + num_ways[row][col - 1]
        
        return num_ways[-1][-1]
```
思考ログ：
- 後半がシンプルに書けて良い

# Step3

かかった時間：6min

```python
class Solution:
    def uniquePathsWithObstacles(self, obstacleGrid: List[List[int]]) -> int:
        OBSTACLE = 1
        if obstacleGrid[0][0] == OBSTACLE \
            or obstacleGrid[-1][-1] == OBSTACLE:
            return 0
        
        m = len(obstacleGrid)
        n = len(obstacleGrid[0])
        def initialize_num_ways():
            num_ways = [[0] * n for _ in range(m)]
            for row in range(m):
                if obstacleGrid[row][0] == OBSTACLE:
                    break
                num_ways[row][0] = 1
            for col in range(n):
                if obstacleGrid[0][col] == OBSTACLE:
                    break
                num_ways[0][col] = 1
            
            return num_ways

        num_ways = initialize_num_ways()
        for row in range(1, m):
            for col in range(1, n):
                if obstacleGrid[row][col] == OBSTACLE:
                    continue
                num_ways[row][col] = num_ways[row - 1][col] + num_ways[row][col - 1]
        
        return num_ways[-1][-1]
```
思考ログ：
- 個人的には後半の処理がすっきりするのが良かった
- 1DPは```min(n, m)```の最適化までやりたくなるが、コードが込み入ってくるので2DPでも良い気がする

# Step4

```python
```
思考ログ：

