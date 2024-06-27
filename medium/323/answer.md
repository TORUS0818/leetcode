# Step1

かかった時間：18min

計算量：
node数をn、edge数をmとして

時間計算量：O(n+m)

空間計算量：O(n+m)

```python
from typing import (
    List,
)
from collections import (
    defaultdict,
    deque,
)


class Solution:
    """
    @param n: the number of vertices
    @param edges: the edges of undirected graph
    @return: the number of connected components
    """
    def count_components(self, n: int, edges: List[List[int]]) -> int:
        # write your code here
        def explore_glaph(i: int) -> None:
            candidates = deque([i])
            while candidates:
                i = candidates.popleft()
                if found[i]:
                    continue
                found[i] = True
                candidates += graph[i]

        graph = defaultdict(list)
        for edge in edges:
            graph[edge[0]].append(edge[1])
            graph[edge[1]].append(edge[0])
        
        found = [False] * n
        num_glaph = 0
        for i in range(n):
            if found[i]:
                continue
            explore_glaph(i)
            num_glaph += 1
        
        return num_glaph
```
思考ログ：
- BFSかDFSでまずは良さそうか
  - leetcodeが有料なのでlintcodeを見ているが、制約が載ってない？
  - DFSなら再帰の深さを考えておく、一直線に繋がった長いグラフだと困る
  - 与えられた形式のグラフ構造データだとやりにくいので各ノードから辿り着けるノード一覧を辞書で持つことにする
  - ノードを順に辿っていく、辿れなくなったら（繋がっているノードが全て発見済みになったら）次へ
    - このタイミングでグラフを1つカウントする
- これこそUnionFindの出番か
- ```i```で済ませている箇所が不親切かなと思いつつ、いい名前も思いつかないし、```node_i```みたいなのもくどいのかなと思いそのままに

UnionFind
```python
from typing import (
    List,
)


class UnionFind():
    def __init__(self, n: int):
        self.n = n
        self.parents = [-1] * n
    
    def find(self, i: int) -> int:
        if self.parents[i] < 0:
            return i
        
        self.parents[i] = self.find(self.parents[i])
        return self.parents[i]
    
    def union(self, i: int, j: int) -> None:
        root_i = self.find(i)
        root_j = self.find(j)
        if root_i == root_j:
            return
        
        if root_i > root_j:
            root_i, root_j = root_j, root_i
        self.parents[root_i] += self.parents[root_j]
        self.parents[root_j] = root_i

class Solution:
    """
    @param n: the number of vertices
    @param edges: the edges of undirected graph
    @return: the number of connected components
    """
    def count_components(self, n: int, edges: List[List[int]]) -> int:
        # write your code here
        uf = UnionFind(n)
        for edge in edges:
            uf.union(edge[0], edge[1])
        
        root_set = set()
        for i in range(n):
            root_set.add(uf.find(i))
        
        return len(root_set)
```
思考ログ：
- UnionFindとも大分仲良くなってきた
- 今回は前回調査したparentsを-1で初期化する方法で実装してみた
  - rank最適化も楽だし慣れればこちらの方がいいかも、という気持ちになってきた
  - ただ読み手がどう思うかの方が大事なのでそこは反応を見て決めたいところ
- 与えられたグラフ情報のデータをそのまま自然に使えるし、相性は良さそう

# Step2

講師役目線でのセルフツッコミポイント：
- graph
  - ```node_to_adjacent_nodes```とかもう少しあるよね
- BFSの解法でgraphにリストを使ってもいいよね
  - 今回ノードはn個順番に存在するのが分かっているので、リストで作っても無駄はない

参考にした過去ログなど：
- https://github.com/kazukiii/leetcode/pull/20
- https://github.com/hayashi-ay/leetcode/pull/37
  - UnionFindでの解法にて
    - ```for node1, node2 in edges:```の書き方が良さそう
    - unionで繋げた回数を管理しておいて、n（ノード数）から引くと、連結した部分がいくつあるか分かる
      - (0, 1, 2, 3)
      - ((0, 1), 2, 3)
      - ((0, 1), (2, 3))
      - 2回繋げたので、4 - 2 = 2つ
  - 命名
  > node だとオブジェクトっぽいときには、node_id にするかなあ、くらいです。
- https://docs.google.com/document/d/13SeRSOE78orbkFV_ytlal-wWZK_7niM5cFLGaNO_qDA/edit
- https://github.com/sakupan102/arai60-practice/pull/22
  - 上でも書いたが、graphに採用するデータ構造について
    > ノード数が決まっており、 0 から順番にノード番号が割り当てられているのであれば、node_to_adjacent_nodes = [[] for _ in range(n)]としてしまう方法もあります。
    > これにより定数倍高速化できる場合があります。理由は、 defaultdict の実装が CPython だとハッシュテーブルなのに対し、 list は可変長変数のためです。
    > これにより、要素に参照する際のランダムメモリアクセスの回数が減ります。
    > ただし、定数倍の高速化が必要な場合に Python を使用するのは本末転倒な気もします。そういった場合は、 C++ 等、より高速な言語で書くほうが良いと思います。
  - 再帰の注意点
    > 訪れているかの判定を次の再帰に任せた場合、やや処理速度が落ちる点は気にしたほうが良いと思います。
    > 理由は、関数の呼び出しが、一般的に、コストがかかる処理だからです。
    > 個人的には再帰関数呼び出しの前に判定したほうが良いと思います。
- https://github.com/shining-ai/leetcode/pull/19

DFS
```python
from typing import (
    List,
)


class Solution:
    """
    @param n: the number of vertices
    @param edges: the edges of undirected graph
    @return: the number of connected components
    """
    def count_components(self, n: int, edges: List[List[int]]) -> int:
        # write your code here
        def explore_graph(i: int) -> None:
            if i in found_nodes:
                return
            found_nodes.add(i)
            for next_i in node_to_adjucent_nodes[i]:
                explore_graph(next_i)

        node_to_adjacent_nodes = [[] for _ in range(n)]
        for node1, node2 in edges:
            node_to_adjucent_nodes[node1].append(node2)
            node_to_adjucent_nodes[node2].append(node1)
        
        found_nodes = set()
        num_connected_components = 0
        for node_i in range(n):
            if node_i in found_nodes:
                continue
            explore_graph(node_i)
            num_connected_components += 1
        
        return num_connected_components
```
思考ログ：
- ```explore_graph```
  - もう少し情報入れられないかしら、、
  - ```explore_connected_nodes```とかか？

# Step3

かかった時間：7min

```python
from typing import (
    List,
)


class UnionFind():
    def __init__(self, n: int):
        self.parents = [-1] * n
    
    def find(self, i: int) -> int:
        if self.parents[i] < 0:
            return i

        self.parents[i] = self.find(self.parents[i])
        return self.parents[i]

    def union(self, i: int, j: int) -> bool:
        root_i = self.find(i)
        root_j = self.find(j)
        if root_i == root_j:
            return False
        
        if root_i > root_j:
            root_i, root_j = root_j, root_i
        
        self.parents[root_i] += self.parents[root_j]
        self.parents[root_j] = root_i

        return True

class Solution:
    """
    @param n: the number of vertices
    @param edges: the edges of undirected graph
    @return: the number of connected components
    """
    def count_components(self, n: int, edges: List[List[int]]) -> int:
        # write your code here
        # 20:21
        uf = UnionFind(n)
        num_connected = 0
        for node1, node2 in edges:
            if uf.union(node1, node2):
                num_connected += 1
        
        return n - num_connected
```
思考ログ：
- UnionFindを使った解法が自然に感じたのでUnionFindで
- ところで、UnionFindの時間計算量は```O(a(n)*m)```、B/DFSが```O(n+m)```だった
  - 逆アッカーマン間数```a(n)```はnが大分大きくても5未満で抑えられることを考えると、mが小さくnが大きいようなケースでは、UnionFindが良さそう？
    - https://ja.wikipedia.org/wiki/%E7%B4%A0%E9%9B%86%E5%90%88%E3%83%87%E3%83%BC%E3%82%BF%E6%A7%8B%E9%80%A0
  - つまりノードはたくさんあるけど、あんまり繋がってない、みたいなケースか

# Step4

```python
```
思考ログ：

