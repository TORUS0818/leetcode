# Step1

かかった時間：17min

計算量：
m == grid.length、n == grid[i].lengthとして

時間計算量：O(n*m)

空間計算量：O(n*m)

```python
class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        def is_valid_pos(pos: List[int]) -> bool:
            h_i, w_i = pos
            if (0 <= h_i < h_len) and (0 <= w_i < w_len):
                return True
            return False

        def check_island_area(pos: List[int]) -> None:
            candidates = [pos]
            while candidates:
                h_i, w_i = candidates.pop()
                grid[h_i][w_i] = '-1'
                for d_h, d_w in [[1, 0], [-1, 0], [0, -1], [0, 1]]:
                    new_h_i, new_w_i = h_i + d_h, w_i + d_w
                    if not is_valid_pos([new_h_i, new_w_i]):
                        continue
                    if grid[new_h_i][new_w_i] == '1':
                        candidates.append([new_h_i, new_w_i])
        
        h_len = len(grid)
        w_len = len(grid[0])
        num_of_islands = 0
        for h_i in range(h_len):
            for w_i in range(w_len):
                if grid[h_i][w_i] == '1':
                    check_island_area([h_i, w_i])
                    num_of_islands += 1

        return num_of_islands
```
思考ログ：
- 島に上陸次第BFSで島の領域を調査済み（"-1"）にしながら島が何個あるか数える方針
- 変数名とか関数切り出しとか工夫の余地がありそう

# Step2

講師役目線でのセルフツッコミポイント：
- DFSも一緒に考えたいところ
- この手の問題、見た瞬間に脳死でBFS（完）となるのは問題あり
  - 問題の解き方の選択肢だけでなく、実装の工夫も幅広く考慮できると良い  
    - '1', '0', '-1'はこのままでいいの？'LAND'と'WATER'とかわかりやすい形にして管理する選択肢は？
    - 関数の切り出し方はこれでいいの？
    - （これは考えたが）gridに直接書き込んじゃっても問題ない？
  - あとこの手の問題はどうせ計算量を多分気にしなくてもいいだろうという思考があり、とてもよくない（ACすることだけが目的だとそうなっていくのか）

参考にした過去ログなど：
- https://github.com/fhiyo/leetcode/pull/20
  - Enumの導入と速度面の注意点
    - ドキュメントに軽く目を通す
      - https://docs.python.org/ja/3/library/enum.html
        > class enum.Flag    
        > Flag is the same as Enum, but its members support the bitwise operators & (AND), | (OR), ^ (XOR), and ~ (INVERT); the results of those operations are (aliases of) members of the enumeration.
        
        > class enum.EnumCheck    
        > EnumCheck には verify() デコレータやさまざまな制約を保証するために使用されるオプションが含まれています。制約に違反すると ValueError が発生します。 
    - https://stackoverflow.com/questions/27735732/why-are-python-enums-slow
  - UnionFind
    - キーワードでdiscordを漁る
      - https://discord.com/channels/1084280443945353267/1183683738635346001/1197738650998415500
    - ここら辺も参考
      - https://note.nkmk.me/python-union-find/
    - ざっくりとまとめると
      - 要素が同じグループかどうかを効率よく判定したい
      - 一つのグループを木で表現して、同じグループかどうかは根が同じかどうかで判断
        - 一直線に要素が並ぶと判定に時間がかかる -> 効率化
          - 再帰的に調べる際に調べた辺を根に直接繋ぎ直す（経路圧縮）
          - 木の高さを持っておき、低い方を高い方に繋げる（ランク）
          - これらの効率化をして計算時間はO(a(n)):aは逆アッカーマン関数
            - 片方だけだと大体O(log(n))くらいらしい
- https://github.com/sakupan102/arai60-practice/pull/18
- https://github.com/rossy0213/leetcode/pull/8
- https://github.com/ryoooooory/LeetCode/pull/2
- https://github.com/hayashi-ay/leetcode/pull/33
- https://docs.google.com/document/d/1y0OX4kCmcvEm9chBdmZU2Mk9cAi7n9GA5zpJ32_nN50/edit

まずはstep1を綺麗にする
```python
class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        WATER = '0'
        UNVISITED = '1'
        VISITED = '2'
        
        def is_valid_position(i: int, j: int) -> bool:
            if (0 <= i < height) and (0 <= j < width):
                return True
            return False

        def check_island_area(i: int, j: int) -> None:
            check_positions = [(i, j)]
            while check_positions:
                i, j = check_positions.pop()
                grid[i][j] = VISITED
                for d_i, d_j in [(1, 0), (-1, 0), (0, 1), (0, -1)]:
                    new_i, new_j = i + d_i, j + d_j
                    if not is_valid_position(new_i, new_j):
                        continue
                    if grid[new_i][new_j] in [VISITED, WATER]:
                        continue

                    grid[new_i][new_j] = VISITED
                    check_positions.append((new_i, new_j))
        
        height = len(grid)
        width = len(grid[0])
        num_islands = 0
        for i in range(height):
            for j in range(width):
                if grid[i][j] == UNVISITED:
                    check_island_area(i, j)
                    num_islands += 1
        
        return num_islands
```
思考ログ：
- ```is_valid_position```にスルー条件は押し込めた方がいい気がする
  - 以降の実装ではそのようにしてみる

DFS
```python
class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        UNVISITED = '1'
        VISITED = '2'

        def is_valid_position(i: int, j: int) -> bool:
            if (0 <= i < height) and (0 <= j < width) and grid[i][j] == UNVISITED:
                return True
            return False

        def check_island_area(i: int, j: int) -> None:
            if not is_valid_position(i, j):
                return

            grid[i][j] = VISITED
            for d_i, d_j in [(1, 0), (-1, 0), (0, 1), (0, -1)]:
                new_i, new_j = i + d_i, j + d_j
                check_island_area(new_i, new_j)

        height = len(grid)
        width = len(grid[0])
        num_islands = 0
        for i in range(height):
            for j in range(width):
                if grid[i][j] == UNVISITED:
                    check_island_area(i, j)
                    num_islands += 1
        
        return num_islands
```
思考ログ：
- 再帰の深さ
  > 1 <= m, n <= 300 なので再帰の回数は高々90Kで、leetcodeの環境で `sys.getrecursionlimit()` で上限を調べてみると `550000` だったので耐えられはしそう。
- 再帰の部分は```for```で回さないで高々4つなので書き下した方が分かりやすいかもしれない

UnionFind(効率化なし)
```python
class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        WATER = '0'
        LAND = '1'

        class UnionFind:
            def __init__(self, n: int):
                self.n = n
                self.parents = list(range(n))

            def find(self, i: int) -> int:
                if self.parents[i] == i:
                    return i
                return self.find(self.parents[i])

            def union(self, i: int, j: int) -> bool:
                root_i = self.find(i)
                root_j = self.find(j)
                if root_i == root_j:
                    return False

                self.parents[root_i] = root_j
                return True

        def grid_pos_to_index(i, j) -> int:
            return i * width + j

        def is_valid_position(i, j) -> bool:
            if (0 <= i < height) and (0 <= j < width) and grid[i][j] == LAND:
                return True
            return False

        height = len(grid)
        width = len(grid[0])
        num_islands = height * width
        uf = UnionFind(num_islands)
        for i in range(height):
            for j in range(width):
                if grid[i][j] == WATER:
                    num_islands -= 1
                    continue

                if is_valid_position(i + 1, j) and uf.union(
                    grid_pos_to_index(i, j), grid_pos_to_index(i + 1, j)
                ):
                    num_islands -= 1
                if is_valid_position(i, j + 1) and uf.union(
                    grid_pos_to_index(i, j), grid_pos_to_index(i, j + 1)
                ):
                    num_islands -= 1

        return num_islands
```
思考ログ：
- 名前は聞いたことがあったが、難しそうなので敬遠していた
- とりあえずrank効率化のないシンプルな実装で試してみる
- ほとんどliquo-riceさんの実装をpython化したもの（ありがたや）
  - https://github.com/fhiyo/leetcode/pull/20/files
  - シンプルだが結構学べるポイントが多い
    - unionで```bool```を返すことで実装がシンプルになってる
    - 島の数の最大値（height * width）を持っておいてWATERやLANDを新しく島の一部として取り込んだ時に1つずつ減らすことで島の数を求めている
    - ネストが深くなりそうな雰囲気だがうまく回避している

UnionFind(効率化、経路圧縮とランク)
```python
class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        WATER = '0'
        LAND = '1'

        class UnionFind:
            def __init__(self, n: int):
                self.n = n
                self.parents = list(range(n))
                self.rank = [0] * n

            def find(self, i: int) -> int:
                if self.parents[i] == i:
                    return i

                # path compression
                self.parents[i] = self.find(self.parents[i])
                return self.parents[i]

            def union(self, i: int, j: int) -> bool:
                root_i = self.find(i)
                root_j = self.find(j)
                if root_i == root_j:
                    return False

                # union by rank
                if self.rank[root_i] > self.rank[root_j]:
                    root_i, root_j = root_j, root_i
                self.parents[root_i] = root_j
                if self.rank[root_i] == self.rank[root_j]:
                    self.rank[root_j] += 1
                return True

        def grid_pos_to_index(i, j) -> int:
            return i * width + j

        def is_valid_position(i, j) -> bool:
            if (0 <= i < height) and (0 <= j < width) and grid[i][j] == LAND:
                return True
            return False

        height = len(grid)
        width = len(grid[0])
        num_islands = height * width
        uf = UnionFind(num_islands)
        for i in range(height):
            for j in range(width):
                if grid[i][j] == WATER:
                    num_islands -= 1
                    continue

                if is_valid_position(i + 1, j) and uf.union(
                    grid_pos_to_index(i, j), grid_pos_to_index(i + 1, j)
                ):
                    num_islands -= 1
                if is_valid_position(i, j + 1) and uf.union(
                    grid_pos_to_index(i, j), grid_pos_to_index(i, j + 1)
                ):
                    num_islands -= 1

        return num_islands
```
思考ログ：
- 効率化
  - path compression
  - union by rank
  - union by size
- 今回の問題のケースでいうと効率化前と実行速度はそんなに変わらなかったが、簡単な実験をしてみたところ割と差が出た
```python
import timeit

import numpy as np


class UnionFindBasic():
    def __init__(self, n: int):
        self.n = n
        self.parents = list(range(n))

    def find(self, i: int) -> int:
        if self.parents[i] == i:
            return i
        return self.find(self.parents[i])

    def union(self, i: int, j: int) -> None:
        root_i = self.find(i)
        root_j = self.find(j)
        if root_i == root_j:
            return
        self.parents[root_i] = root_j

class UnionFindPro():
    def __init__(self, n: int):
        self.n = n
        self.parents = list(range(n))
        self.rank = [0] * n

    def find(self, i: int) -> int:
        if self.parents[i] == i:
            return i
        self.parents[i] = self.find(self.parents[i])
        return self.parents[i]

    def union(self, i: int, j: int) -> None:
        root_i = self.find(i)
        root_j = self.find(j)
        if root_i == root_j:
            return
        
        if self.rank[root_i] > self.rank[root_j]:
            root_i, root_j = root_j, root_i
        self.parents[root_i] = root_j
        if self.rank[root_i] == self.rank[root_j]:
            self.rank[root_j] += 1

def test1(n: int = 1000):
    uf = UnionFindBasic(n)
    for _ in range(n):
        i, j = np.random.choice(n, 2)
        uf.union(i, j)
        uf.find(i)
        uf.find(j)

def test2(n: int = 1000):
    uf = UnionFindPro(n)
    for _ in range(n):
        i, j = np.random.choice(n, 2)
        uf.union(i, j)
        uf.find(i)
        uf.find(j)

if __name__ == '__main__':
    repeat_n = 1000
    print(timeit.timeit(test1, number=repeat_n) / repeat_n) # 0.012343118333
    print(timeit.timeit(test2, number=repeat_n) / repeat_n) # 0.0070453356660000015
```

# Step3

かかった時間：8min

```python
class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        UNVISITED = '1'
        VISITED = '2'
        
        def is_valid_position(i: int, j: int) -> bool:
            if (0 <= i < height) and (0 <= j < width) and grid[i][j] == UNVISITED:
                return True
            return False

        def search_island_area(i: int, j: int) -> None:
            if not is_valid_position(i, j):
                return
            grid[i][j] = VISITED
            search_island_area(i - 1, j)
            search_island_area(i + 1, j)
            search_island_area(i, j - 1)
            search_island_area(i, j + 1)

        height = len(grid)
        width = len(grid[0])
        num_islands = 0
        for i in range(height):
            for j in range(width):
                if grid[i][j] == UNVISITED:
                    search_island_area(i, j)
                    num_islands += 1
        
        return num_islands
```
思考ログ：
- 苦手？なDFSで書いてみた
  - ListNodeあたりで色々教えて頂きながら再帰と結構戯れたおかげで、苦手意識は無くなってきた模様
- コードは同じものに収束するが、少し書くのに時間がかかる

# Step4

```python
```
思考ログ：

