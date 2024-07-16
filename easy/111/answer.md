# Step1

かかった時間：7min

計算量：
nodeの数をNとして

時間計算量：O(N)

空間計算量：O(N)

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def minDepth(self, root: Optional[TreeNode]) -> int:
        if not root:
            return 0
        
        if not root.left:
            return self.minDepth(root.right) + 1
        if not root.right:
            return self.minDepth(root.left) + 1    

        return min(self.minDepth(root.left), self.minDepth(root.right)) + 1
```
思考ログ：
- 前回（104）とほぼ同じだなあと再帰を組んでエラー
  ```python
  if not root:
      return 0 
  
  return min(self.minDepth(root.left), self.minDepth(root.right)) + 1
  ```
- 問題がある箇所にはすぐ気付いたが、修正に手間取る
- 片方しかない場合は飛ばして先に進めば良かった
- maxとminなので対称性があると思ったのに、、
  - と勝手に騙された気分になっていたが、maxの時は子が1つの時、depthが0のものを採用することが無い、というだけの話
  - minの場合は逆に必ず0が採用されてしまうので今回のようなことになった

BFS
```python
from collections import deque
import math


# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def minDepth(self, root: Optional[TreeNode]) -> int:
        if not root:
            return 0

        next_node_with_depth = deque([(root, 1)])
        min_depth = math.inf
        while next_node_with_depth:
            node, depth = next_node_with_depth.popleft()
            if not (node.left or node.right):
                min_depth = min(min_depth, depth)
                continue

            if node.left:
                next_node_with_depth.append((node.left, depth + 1))
            if node.right:
                next_node_with_depth.append((node.right, depth + 1))
        
        return int(min_depth)
```
思考ログ：
- 幅優先で探索して葉に辿り着いたら最小の深さを更新

# Step2

講師役目線でのセルフツッコミポイント：
- せっかくのBFSの性質を活かせてない、、葉に辿り着いたらdepthをreturnすれば良い

参考にした過去ログなど：
- https://github.com/Yoshiki-Iwasa/Arai60/pull/25
- https://github.com/kazukiii/leetcode/pull/23
  - is と == の違い  
    - https://docs.python.org/ja/3/reference/expressions.html#is-not
- https://github.com/SuperHotDogCat/coding-interview/pull/36
- https://github.com/fhiyo/leetcode/pull/24
  - depthの初期値（十分大きな値）をどうするかの議論  
- https://github.com/Mike0121/LeetCode/pull/11
  - DFSの枝刈り
- https://github.com/sakupan102/arai60-practice/pull/23
  - ループ時のdepthのカウントアップの場所について  
- https://github.com/rossy0213/leetcode/pull/11
- https://github.com/shining-ai/leetcode/pull/22
- https://github.com/hayashi-ay/leetcode/pull/26
- https://discord.com/channels/1084280443945353267/1183683738635346001/1202680261108826184

DFS（再帰）
```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def minDepth(self, root: Optional[TreeNode]) -> int:
        if not root:
            return 0
        if not root.left and not root.right:
            return 1

        if not root.left:
            return self.minDepth(root.right) + 1
        if not root.right:
            return self.minDepth(root.left) + 1

        return min(self.minDepth(root.left), self.minDepth(root.right)) + 1
```
思考ログ：
- ```if not root.left and not root.right:```で1を返してしまう選択肢もある
- ```if not (root.left or root.right):```と書いていたのを```if not root.left and not root.right:```へ変更

DFS（stack）
```python
import math


# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def minDepth(self, root: Optional[TreeNode]) -> int:
        if not root:
            return 0

        node_depth_pairs = [(root, 1)]
        min_depth = math.inf
        while node_depth_pairs:
            node, depth = node_depth_pairs.pop()
            if depth >= min_depth:
                continue

            if not node.left and not node.right:
                min_depth = min(min_depth, depth)
        
            if node.right:
                node_depth_pairs.append((node.right, depth + 1))
            if node.left:
                node_depth_pairs.append((node.left, depth + 1))
        
        return min_depth
```
思考ログ：
- 過去ログにあった枝刈りを追加
  ```python
  if depth >= min_depth:
      continue
  ```
- ```node.right```, ```node.left```の順に入れて自然な探索順になるようにした

BFS
```python
from collections import deque


# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def minDepth(self, root: Optional[TreeNode]) -> int:
        if not root:
            return 0
        
        node_depth_pairs = deque([(root, 1)])
        while node_depth_pairs:
            node, depth = node_depth_pairs.popleft()
            if not node.left and not node.right:
                return depth
            
            if node.left:
                node_depth_pairs.append((node.left, depth + 1))
            if node.right:
                node_depth_pairs.append((node.right, depth + 1))
        
        raise Exception('unreachable')
```
思考ログ：
- step1でやっていた無駄な探索を改良
```python
if not node.left and not node.right:
    return depth
```
- 最終行に例外を追加

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
    def minDepth(self, root: Optional[TreeNode]) -> int:
        if not root:
            return 0

        node_depth_pairs = deque([(root, 1)])
        while node_depth_pairs:
            node, depth = node_depth_pairs.popleft()
            if not node.left and not node.right:
                return depth
            
            if node.left:
                node_depth_pairs.append((node.left, depth + 1))
            if node.right:
                node_depth_pairs.append((node.right, depth + 1))
        
        raise Exception('unreachable')
```
思考ログ：
- BFSが素直な気がするのでこれで

# Step4

```python
```
思考ログ：

