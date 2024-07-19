# Step1

かかった時間：17min

計算量：
2つの木の重なっている部分のノード数をNとすると

時間計算量：O(N)

空間計算量：O(N)

```python
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def mergeTrees(self, root1: Optional[TreeNode], root2: Optional[TreeNode]) -> Optional[TreeNode]:
        if not root1 and not root2:
            return None
        if not root1:
            return root2
        if not root2:
            return root1

        root = TreeNode(val=root1.val + root2.val)
        root.left = self.mergeTrees(root1.left, root2.left)
        root.right = self.mergeTrees(root1.right, root2.right)

        return root
```
思考ログ：
- 嵌ってしまって結構時間がかかった
- ループへの書き換えを考えたが、パッと思いつけなかった
  - 木の操作はまだ慣れてない感じ、もう少し数をこなすことか
- 再帰上限確認（2,000なのでデフォルト設定では危ない）
- これ、非破壊的な実装を意識して書いたが、停止条件でそのまま```root1```、```root2```を返しているので、中途半端なことになっている

ループ（DFS）
```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def mergeTrees(self, root1: Optional[TreeNode], root2: Optional[TreeNode]) -> Optional[TreeNode]:
        if not root1 and not root2:
            return None
        if not root1:
            return root2
        if not root2:
            return root1
        
        def merge_nodes(node1: Optional[TreeNode], node2: Optional[TreeNode]) -> Optional[TreeNode]:
            if node1 and node2:
                return TreeNode(node1.val + node2.val)
            if node1:
                return TreeNode(node1.val)
            if node2:
                return TreeNode(node2.val)
            return None

        merged_root = TreeNode(root1.val + root2.val)
        next_nodes = [(root1, root2, merged_root)]
        while next_nodes:
            node1, node2, merged_node = next_nodes.pop()

            if not node1 and not node2:
                continue
            if node1 and not node2:
                merged_node.left = node1.left
                merged_node.right = node1.right
                continue
            if not node1 and node2:
                merged_node.left = node2.left
                merged_node.right = node2.right
                continue
                
            merged_node.left = merge_nodes(node1.left, node2.left)
            merged_node.right = merge_nodes(node1.right, node2.right)
            next_nodes.append((node1.left, node2.left, merged_node.left))
            next_nodes.append((node1.right, node2.right, merged_node.right))
            
        return merged_root
```
思考ログ：
- 長い
- 条件分岐が込み入っている

# Step2

講師役目線でのセルフツッコミポイント：
- ループでも書いてみる
- root1/2どちらかをベースにマージする方法も頭にあったか

参考にした過去ログなど：
- https://github.com/kazukiii/leetcode/pull/24
- https://github.com/SuperHotDogCat/coding-interview/pull/35
- https://github.com/fhiyo/leetcode/pull/25
  - https://github.com/fhiyo/leetcode/pull/25/files#r1656461334
    > 新しいのを作るか作らないのか、古い入力を壊すのか壊さないのか、共有するのかしないのか(変更しない前提ならばメモリー使用量が減る)、などのオプションがあって、自分がどれを「選択」したかを意識しましょう。
  - deepcopy
    - https://docs.python.org/ja/3/library/copy.html
    - https://github.com/python/cpython/blob/main/Lib/copy.py#L118
    > A deep copy constructs a new compound object and then, recursively, inserts copies into it of the objects found in the original.   
- https://github.com/sakupan102/arai60-practice/pull/24
  - 下記のようにしてまとめて書ける
  ```python
  if not root1 or not root2:
      return root1 or root2
  ```
  - https://github.com/sakupan102/arai60-practice/pull/24/files#r1592577213
    > この関数自体は非破壊的かもしれないですが、この関数の引数のTreeを加工したら引数のTreeも意図せず変わってしまう場合があります。
- https://github.com/Mike0121/LeetCode/pull/9
  - BFSの実装あり  
- https://github.com/rossy0213/leetcode/pull/12
- https://github.com/hayashi-ay/leetcode/pull/12
  - https://github.com/hayashi-ay/leetcode/pull/12/files#r1478253882
    > 両方Noneのときは、```if not root1: return root2```で処理できると思います。  

再帰（非破壊的）
```python
from copy import deepcopy


# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def mergeTrees(self, root1: Optional[TreeNode], root2: Optional[TreeNode]) -> Optional[TreeNode]:
        if not root1 or not root2:
            return deepcopy(root1) or deepcopy(root2)
        
        merged_root = TreeNode(root1.val + root2.val)
        merged_root.left = self.mergeTrees(root1.left, root2.left)
        merged_root.right = self.mergeTrees(root1.right, root2.right)

        return merged_root
```
思考ログ：
- 再帰の停止条件（Line:136）がスッキリした
  - が、個別に書いた方が読む人には優しいのかもしれない

再帰（破壊的）
```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def mergeTrees(self, root1: Optional[TreeNode], root2: Optional[TreeNode]) -> Optional[TreeNode]:
        if not root1 or not root2:
            return root1 or root2

        root1.val += root2.val
        root1.left = self.mergeTrees(root1.left, root2.left)
        root1.right = self.mergeTrees(root1.right, root2.right)

        return root1
```
思考ログ：
- 個人的には破壊しない方が好みではある

# Step3

かかった時間：2min

```python
from copy import deepcopy


# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def mergeTrees(self, root1: Optional[TreeNode], root2: Optional[TreeNode]) -> Optional[TreeNode]:
        if not root1 or not root2:
            return deepcopy(root2) or deepcopy(root1)
        
        merged_root = TreeNode(root1.val + root2.val)
        merged_root.left = self.mergeTrees(root1.left, root2.left)
        merged_root.right = self.mergeTrees(root1.right, root2.right)

        return merged_root
```
思考ログ：
- ```return deepcopy(root2) or deepcopy(root1)```と逆順にしてみたがこっちの方が意図が伝わりやすかったりするだろうか？
  - ```not root1``` -> ```return deepcopy(root2)```の対応が読み取りやすいかなと思ったのだが

# Step4

```python
```
思考ログ：

