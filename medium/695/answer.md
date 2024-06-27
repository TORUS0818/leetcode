# Step1

かかった時間：11min

計算量：
m == grid.length, n == grid[i].lengthとして

時間計算量：O(n*m)

空間計算量：O(n*m)

```python
from collections import deque


class Solution:
    def maxAreaOfIsland(self, grid: List[List[int]]) -> int:
        UNVISITED = 1
        VISITED = 2

        def is_valid_position(h: int, w: int) -> bool:
            return 0 <= h < height and 0 <= w < width and grid[h][w] == UNVISITED

        def explore_island_area(h: int, w: int) -> int:
            candidates = deque([(h, w)])
            grid[h][w] = VISITED
            area = 1
            while candidates:
                h, w = candidates.popleft()
                for d_h, d_w in [(-1, 0), (1, 0), (0, -1), (0, 1)]:
                    new_h, new_w = h + d_h, w + d_w
                    if not is_valid_position(new_h, new_w):
                        continue
                    candidates.append((new_h, new_w))
                    grid[new_h][new_w] = VISITED
                    area += 1

            return area

        height = len(grid)
        width = len(grid[0])
        max_area = 0
        for h in range(height):
            for w in range(width):
                if grid[h][w] == UNVISITED:
                    max_area = max(max_area, explore_island_area(h, w))

        return max_area
```
思考ログ：
- Number of Islandsの類題か
  - と思って油断していたら、島がstrじゃなくてintで入っていて怒られる
  - 前回Step3でDFSで書いたので今回はBFSで書いてみる
  - テンプレ覚えてしまっているが、条件判定の場所とか書き方のバリエーションはあることを忘れずに
- BFS, DFS, UnionFindあたりが選択肢か
- 前回同様grid直接書き換えていいのか問題もある

UnionFind
```python
class Solution:
    def maxAreaOfIsland(self, grid: List[List[int]]) -> int:
        WATER = 0
        
        class UnionFind():
            def __init__(self, n: int):
                self.n = n
                self.parents = list(range(n))
                self.depth = [1] * n
            
            def find(self, i: int) -> int:
                if self.parents[i] == i:
                    return i
                self.parents[i] = self.find(self.parents[i])
                return self.parents[i]

            def union(self, i: int, j: int):
                root_i = self.find(i)
                root_j = self.find(j)
                if root_i == root_j:
                    return
                self.parents[root_i] = root_j
                self.depth[root_j] += self.depth[root_i]

            def size(self, i: int) -> int:
                return self.depth[self.find(i)]

        def convert_grid_position_to_index(h: int, w: int) -> int:
            return h * width + w

        def is_valid_position(h: int, w: int) -> bool:
            return 0 <= h < height and 0 <= w < width and grid[h][w] == 1

        height = len(grid)
        width = len(grid[0])
        uf = UnionFind(height * width)
        max_area = 0
        for h in range(height):
            for w in range(width):
                if grid[h][w] == WATER:
                    continue
                if is_valid_position(h, w + 1):    
                    uf.union(
                        convert_grid_position_to_index(h, w),
                        convert_grid_position_to_index(h, w + 1)
                    )
                if is_valid_position(h + 1, w):    
                    uf.union(
                        convert_grid_position_to_index(h, w),
                        convert_grid_position_to_index(h + 1, w)
                    )
                max_area = max(
                    max_area, 
                    uf.size(convert_grid_position_to_index(h, w))
                )
        
        return max_area
```
思考ログ：
- 復讐も兼ねて思い出しながらUnionFind書いてみる
- 今回効率化は経路圧縮だけ実装
- こっちはUnionFindで島の状態を管理していくのでgridは弄らないで済む
- ```convert_grid_position_to_index```を作ったが、UnionFindのparentsを二次元リストにするのも一考

# Step2

講師役目線でのセルフツッコミポイント：
- 命名、探索するだけでなく島の面積を求める関数なので、```explore_island_area```はよろしくない
- Step1の道中で割と考えられるようになってきたかもしれない

参考にした過去ログなど：
- https://github.com/kazukiii/leetcode/pull/19
  - UnionFindをparentsを-1で初期化する方式で実装してあった
    - この実装よく見かけるが、よく分かってないのでちょっと調べてみる
      - parentsは負なら根、正なら親のインデックスが入っている
      - parentsは負なら深さの情報も兼ねる
      - 慣れていれば便利だが、あまり素直な実装とは言えないので使う場合は注意が必要かも
- https://github.com/goto-untrapped/Arai60/pull/31
- https://github.com/fhiyo/leetcode/pull/21
- https://github.com/sakupan102/arai60-practice/pull/19
- https://github.com/YukiMichishita/LeetCode/pull/6
  - 4方向のarea + 1（自分自身）を返す、成程
  ```python
  def calc_area(x, y):
      if not (0 <= x < width) or not (0 <= y < height) or visited[y][x] or grid[y][x] == WATER:
          return 0
      visited[y][x] = True
      return calc_area(x + 1, y) + calc_area(x, y + 1) + calc_area(x - 1, y) + calc_area(x, y - 1) + 1
  ```
- https://github.com/rossy0213/leetcode/pull/9
- https://github.com/hayashi-ay/leetcode/pull/34
  - UnionFindの実装にて
    > さらに、
    > 
    ```python
    for h in range(height - 1):
        for w in range(width - 1):
    ```
    > とすれば、この if 文を消せると思います。  
- https://docs.google.com/document/d/1Ak9oSrtFW_L5w0ZeLNDjl9wm32zzQsMnEJI8cS8h7qM/edit

BFS、判定を一箇所に押し込めてみた、また訪問したか否かの情報は別途visitedで管理しgridを保護
```python
from collections import deque


class Solution:
    def maxAreaOfIsland(self, grid: List[List[int]]) -> int:
        WATER = 0

        def is_valid_position(h: int, w: int) -> bool:
            return (
                0 <= h < height and 
                0 <= w < width and 
                grid[h][w] != WATER and
                not visited[h][w] 
            )

        def calculate_island_area(h: int, w: int) -> int:
            candidates = deque([(h, w)])
            area = 0
            while candidates:
                h, w = candidates.popleft()
                if not is_valid_position(h, w):
                    continue

                visited[h][w] = True
                area += 1
                candidates.append((h - 1, w))
                candidates.append((h + 1, w))
                candidates.append((h, w - 1))
                candidates.append((h, w + 1))
            
            return area      

        height = len(grid)
        width = len(grid[0])
        visited = [[False] * width for _ in range(height)]
        max_area = 0
        for h in range(height):
            for w in range(width):
                max_area = max(max_area, calculate_island_area(h, w))
        
        return max_area
```
思考ログ：
- コード全体がさっぱりはした気はする
- 一方でfor文の中で毎回関数を呼んでチェックしているのは非効率な感じ

BFSをDFSへ
```python
class Solution:
    def maxAreaOfIsland(self, grid: List[List[int]]) -> int:
        WATER = 0

        def is_valid_position(h: int, w: int) -> bool:
            return (
                0 <= h < height and 
                0 <= w < width and 
                grid[h][w] != WATER and
                not visited[h][w] 
            )

        def calculate_island_area(h: int, w: int) -> int:
            if not is_valid_position(h, w):
                return 0

            visited[h][w] = True
            return (
                calculate_island_area(h - 1, w) +
                calculate_island_area(h + 1, w) + 
                calculate_island_area(h, w - 1) +
                calculate_island_area(h, w + 1) + 
                1
            )

        height = len(grid)
        width = len(grid[0])
        visited = [[False] * width for _ in range(height)]
        max_area = 0
        for h in range(height):
            for w in range(width):
                max_area = max(max_area, calculate_island_area(h, w))
        
        return max_area
```
思考ログ：
- 再帰の深さを確認
  - m * n <= 2500
  - （pythonの）デフォルトの再帰上限は1000ということもあり、わざわざ選択することもないか

# Step3

かかった時間：7min

```python
from collections import deque


class Solution:
    def maxAreaOfIsland(self, grid: List[List[int]]) -> int:
        UNVISITED = 1
        VISITED = 2
        def is_valid_position(h: int, w: int) -> bool:
            return 0 <= h < height and 0 <= w < width and grid[h][w] == UNVISITED

        def calculate_island_area(h: int, w: int) -> int:
            candidates = deque([(h, w)])
            area = 0
            while candidates:
                h, w = candidates.popleft()
                if not is_valid_position(h, w):
                    continue

                grid[h][w] = VISITED
                area += 1
                candidates.append((h - 1, w))
                candidates.append((h + 1, w))
                candidates.append((h, w - 1))
                candidates.append((h, w + 1))

            return area

        height = len(grid)
        width = len(grid[0])
        max_area = 0
        for h in range(height):
            for w in range(width):
                if grid[h][w] == UNVISITED:
                    max_area = max(max_area, calculate_island_area(h, w))
        
        return max_area
```
思考ログ：
- 途中で詰まる感じもなく同じものを再現できるが、書くのにそこそこ時間はかかってしまう
  - 皆さん、5分くらいで書いてるイメージだが

コメントを受けて、UnionFindの実装を整理
# Step4

```python
class Solution:
    def maxAreaOfIsland(self, grid: List[List[int]]) -> int:
        WATER = 0
        LAND = 1
        
        class UnionFind():
            def __init__(self, n: int):
                self.n = n
                self.parents = list(range(n))
                self.sizes = [1] * n
            
            def find(self, i: int) -> int:
                if self.parents[i] == i:
                    return i
                self.parents[i] = self.find(self.parents[i])
                return self.parents[i]

            def union(self, i: int, j: int):
                root_i = self.find(i)
                root_j = self.find(j)
                if root_i == root_j:
                    return
                self.parents[root_i] = root_j
                self.sizes[root_j] += self.sizes[root_i]

            def size(self, i: int) -> int:
                return self.sizes[self.find(i)]

        def convert_grid_position_to_index(h: int, w: int) -> int:
            return h * width + w

        def is_valid_position(h: int, w: int) -> bool:
            return 0 <= h < height and 0 <= w < width and grid[h][w] == LAND

        height = len(grid)
        width = len(grid[0])
        uf = UnionFind(height * width)
        max_area = 0
        for h in range(height):
            for w in range(width):
                if grid[h][w] == WATER:
                    continue
                if is_valid_position(h, w + 1):    
                    uf.union(
                        convert_grid_position_to_index(h, w),
                        convert_grid_position_to_index(h, w + 1)
                    )
                if is_valid_position(h + 1, w):    
                    uf.union(
                        convert_grid_position_to_index(h, w),
                        convert_grid_position_to_index(h + 1, w)
                    )
                max_area = max(
                    max_area, 
                    uf.size(convert_grid_position_to_index(h, w))
                )
        
        return max_area
```
思考ログ：
- 特になし

# Step5

```python
```
思考ログ：
