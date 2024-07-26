# Step1

かかった時間：7min

計算量：
ノード数をNとして

時間計算量：O(NlogN)

空間計算量：O(N)

```python
from collections import deque, defaultdict


# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def levelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        node_level_pairs = deque([(root, 1)])
        level_to_values = defaultdict(list)
        while node_level_pairs:
            node, level = node_level_pairs.popleft()
            if not node:
                continue

            level_to_values[level].append(node.val)
            node_level_pairs.append((node.left, level + 1))
            node_level_pairs.append((node.right, level + 1))

        return [
            values for _, values in sorted(level_to_values.items(), key=lambda x: x[0])
        ]
```
思考ログ：
- BFSで辿りながらlevelごとにノードの値を記録していく方針を思いついた
- 辞書で管理したがlevel毎にループを回して直接リストを作って行った方が素直な感じがする
  - あと最後のソートで計算量が悪くなっている（NlogN）
  - python3.7以降の前提なら順序の保証があるのでこの余計なソートは省けるが

dictを使わないで書き直してみた
```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def levelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        if not root:
            return []

        nodes = [root]
        level_ordered_values = []
        while nodes:
            next_nodes = []
            same_level_values = []
            for node in nodes:
                same_level_values.append(node.val)
                if node.left:
                    next_nodes.append(node.left)
                if node.right:
                    next_nodes.append(node.right)

            level_ordered_values.append(same_level_values)
            nodes = next_nodes 
        
        return level_ordered_values
```
思考ログ：
- 時間計算量をO(N)へ
- 命名が難しい
  - ```level_order_values```は二重リスト、```same_level_values```は単一のリストなんだけど同じ```hoge_values```というのがなんか気に入らない

# Step2

講師役目線でのセルフツッコミポイント：

参考にした過去ログなど：
- https://github.com/kazukiii/leetcode/pull/27
  - https://github.com/kazukiii/leetcode/pull/27/files#r1676087524
    > 1段だけしか拡張しなくても例外が投げられることがないことに気がつくパズルを解かせる必要ないですよね。そうすると、下にするならば、コメント1行付けておいて、くらいの感覚です。   
    - この議論、理解は出来るが、まだ瞬時にこの感覚（違和感）を持つのは難しいと感じた
    - 意識して感覚を矯正する必要がありそう
  - ```level```と配列の長さを見比べて、足りなくなったら空配列を追加する方法
- https://github.com/Yoshiki-Iwasa/Arai60/pull/30
- https://github.com/fhiyo/leetcode/pull/28
  - 再帰の実装あり
  - DFSの実装あり
- https://github.com/sakupan102/arai60-practice/pull/27
  - 配列の長さでlevelを管理する方法
    - https://github.com/sakupan102/arai60-practice/pull/27/files#r1597353212  
- https://github.com/Mike0121/LeetCode/pull/7
- https://github.com/shining-ai/leetcode/pull/26
  - DFSの実装あり
- https://github.com/hayashi-ay/leetcode/pull/32

BFS（1-deque ver）
```python
from collections import deque


# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def levelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        if not root:
            return []
        
        nodes = deque([root])
        level_ordered_values = []
        while nodes:
            level_ordered_values.append([])
            level = len(level_ordered_values) - 1
            num_nodes = len(nodes)
            for _ in range(num_nodes):
                node = nodes.popleft()
                level_ordered_values[level].append(node.val)
                
                if node.left: nodes.append(node.left)
                if node.right: nodes.append(node.right)
        
        return level_ordered_values
```
思考ログ：
- levelの情報を配列の要素数で管理する方法
- 一番後ろの配列に追加していけば```level```は不要
- 参考
  - https://github.com/sakupan102/arai60-practice/pull/27/files#r1597353212  

DFS（練習）
```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def levelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        level_ordered_values = []
        def level_order_heelper(root: Optional[TreeNode], level: int) -> None:
            if not root:
                return None

            while len(level_ordered_values) - 1 < level:
                level_ordered_values.append([])
            level_ordered_values[level].append(root.val)
            
            if root.left: level_order_heelper(root.left, level + 1)
            if root.right: level_order_heelper(root.right, level + 1)
        
        level_order_heelper(root, 0)

        return level_ordered_values
```
思考ログ：
- 練習のため書いてみた
- L154~155のwhileについて、以下の議論があった
  - https://github.com/kazukiii/leetcode/pull/27/files#r1676087524
- 参考
  - https://github.com/shining-ai/leetcode/pull/26

# Step3

かかった時間：3min

```python
from collections import deque


# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def levelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        if not root:
            return []

        nodes = deque([root])
        level_ordered_values = []
        while nodes:
            level_ordered_values.append([])
            for _ in range(len(nodes)):
                node = nodes.popleft()
                level_ordered_values[-1].append(node.val)
                if node.left: nodes.append(node.left)
                if node.right: nodes.append(node.right)
        
        return level_ordered_values
```
思考ログ：
- 1-deque-BFSにした
- ```level```を使わないように修正

# Step4

```python
```
思考ログ：

