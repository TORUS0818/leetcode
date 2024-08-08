# Step1

かかった時間：解けず

計算量：  
ノード数をNとして

時間計算量：O(N)

空間計算量：O(N)


```python
import math


# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        def is_valid_bst_helper(
            root: Optional[TreeNode], lower_val: float, upper_val: float
        ):
            if not root:
                return True

            if root.left:
                if not (lower_val < root.left.val < root.val):
                    return False
            if root.right:
                if not (root.val < root.right.val < upper_val):
                    return False
            
            return is_valid_bst_helper(root.left, lower_val, root.val) \
            and is_valid_bst_helper(root.right, root.val, upper_val)
        
        return is_valid_bst_helper(root, -math.inf, math.inf)
```
思考ログ：
- 最初の取り組みでタイムアップし、次の日もう一度解いてみたら10分程で解決した
- 初手では、親と子の関係（左の子<親<右の子）しか考えておらず、祖先との関係性を引き継ぐ必要があるのに気づかなかった  
  - 各ノードが収まる範囲として、```lower_val```と```upper_val```を導入して、親との関係に加えて、この区間に収まっているかを確認する（というか親との関係も```lower_val```と```upper_val```の一部なのだが）
  - 左に行ったら```upper_val```が```root.val```で更新され、右に行ったら```lower_val```が```root.val```で更新される
- 再帰で10**4なのでデフォルト設定だと厳しいことは念頭においておく

ループに直しておく（DFS）
```python
import math


# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        node_valid_range_pairs = [(root, -math.inf, math.inf)]
        while node_valid_range_pairs:
            node, lower_val, upper_val = node_valid_range_pairs.pop()
            if not (lower_val < node.val < upper_val):
                return False
            if node.right: 
                node_valid_range_pairs.append((node.right, node.val, upper_val))
            if node.left: 
                node_valid_range_pairs.append((node.left, lower_val, node.val))
        
        return True
```
思考ログ：
- 親から有効な範囲を渡されてチェックしていく、有効なノードでなかったら```False```を返す
- 最後まで走り切れたら```True```

一応BFSも
```python
import math
from collections import deque


# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        node_valid_range_pairs = deque([(root, -math.inf, math.inf)])
        while node_valid_range_pairs:
            node, lower_val, upper_val = node_valid_range_pairs.popleft()
            if not (lower_val < node.val < upper_val):
                return False
            if node.left: 
                node_valid_range_pairs.append((node.left, lower_val, node.val))
            if node.right: 
                node_valid_range_pairs.append((node.right, node.val, upper_val))
        
        return True
```
思考ログ：
- 特になし

# Step2

講師役目線でのセルフツッコミポイント：
- in-orderの探索は頭にあった方が良い

参考にした過去ログなど：
- https://github.com/kazukiii/leetcode/pull/29
- https://github.com/Yoshiki-Iwasa/Arai60/pull/32
- https://github.com/sakupan102/arai60-practice/pull/29
  - in-orderのループ実装あり
- https://github.com/Mike0121/LeetCode/pull/8
  - この書き方、頭になかった
  - 値の取りうる範囲を管理しなくても、左優先で探索していけば上手くいく（つまりはin-orderということなんだけど）
  ```python
  class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        if not root:
            return True

        if root.left:
            if root.left.val >= root.val:
                return False
            if not self.isValidBST(root.left):
                return False

        if root.right:
            if root.right.val <= root.val:
                return False
            if not self.isValidBST(root.right):
                return False

        return True
  ```  
- https://github.com/YukiMichishita/LeetCode/pull/8
  - inorder traversal(stack)の実装
  - inorder traversalをyieldを使って実装（これやってみる）
    ```python
    def inorder_sort(node):
        if not node:
            return
        if node.left:
            yield from inorder_sort(node.left)
        yield node
        if node.right:
            yield from inorder_sort(node.right)
    ```
    - ```return```と```yield```が混在している場合どうなるんだっけ、、
      - https://peps.python.org/pep-0380/
        > return expr in a generator causes StopIteration(expr) to be raised upon exit from the generator.
- https://github.com/shining-ai/leetcode/pull/28
- https://github.com/hayashi-ay/leetcode/pull/38
  - Morris in-order traversal
    > （基本方針） .rightでin-orderなLinked Listを作る。  
    > あるノードに注目したときにin-orderで探索するには、左 -> 自分 -> 右という順番で見ていけば良い。  
    > 左の部分木を探索する際にあとで自分に戻ってこれるようにpredecessorから自分に対してrightでつなぐ。  
    > 左が存在しない場合は自分を頂点とする部分木において、自分が先頭になる。
    - 結構複雑、、単純な木でトレースしてみた

in-order（再帰）
```python
import math


# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        prev_val = -math.inf
        is_bst = True
        def is_valid_bst_helper(node):
            nonlocal prev_val, is_bst
            
            nodes = [node]
            while nodes:
                node = nodes.pop()
                if not node: continue

                is_valid_bst_helper(node.left)
                if prev_val >= node.val:
                    is_bst = False
                    return
                prev_val = node.val
                is_valid_bst_helper(node.right)
        
        is_valid_bst_helper(root)
        return is_bst
```
思考ログ：
- 二分探索木といえば、in-orderの実装を失念していた

in-order（ループ）
```python
import math


# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        def append_all_left_nodes(node: TreeNode, nodes: list[TreeNode]) -> None:
            while node:
                nodes.append(node)
                node = node.left

        nodes = []
        append_all_left_nodes(root, nodes)
        prev_val = -math.inf
        while nodes:
            node = nodes.pop()
            if prev_val >= node.val:
                return False
            prev_val = node.val
            if node.right:
                append_all_left_nodes(node.right, nodes)
        
        return True
```
思考ログ：
- ループに直す

in-order（yield）
```python
import math


# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        def traverse_inorder(node: TreeNode) -> int:
            if node.left: yield from traverse_inorder(node.left)
            yield node.val
            if node.right: yield from traverse_inorder(node.right)
        
        prev_val = -math.inf
        for val in traverse_inorder(root):
            if prev_val >= val:
                return False
            prev_val = val
        
        return True
```
思考ログ：
- ```yield```は何か処理の流れを追いにくくなるイメージを持っていたが、これは分かりやすくシンプルに書けて良さそう

# Step3

かかった時間：3min

```python
import math


# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        def traverse_inorder(node: TreeNode) -> int:
            if node.left: yield from traverse_inorder(node.left)
            yield node.val
            if node.right: yield from traverse_inorder(node.right)
        
        prev_val = -math.inf
        for val in traverse_inorder(root):
            if prev_val >= val:
                return False
            prev_val = val
        
        return True
```
思考ログ：
- 選択肢を増やすという意味でも、あまり使い慣れない```yield```の実装で
- 今持ってるもので勝負しがちなので、面倒くさがらずに色々なやり方を見ておくのは大事(定期)

# Step4

```python
```
思考ログ：

