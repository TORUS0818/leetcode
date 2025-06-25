# Step1

かかった時間：解けず

計算量：

Nをノード数として
- 時間計算量：O(N)
- 空間計算量：O(N)

```python
from lintcode import (
    TreeNode,
)

"""
Definition of TreeNode:
class TreeNode:
    def __init__(self, val):
        self.val = val
        self.left, self.right = None, None
"""

class Solution:
    """
    @param root: the given tree
    @param v: the target value
    @return: the root TreeNode after splitting
    """
    def split_b_s_t(self, root: TreeNode, v: int) -> TreeNode:
        # write your code here
        def count_nodes(root: TreeNode) -> int:
            if not root:
                return 0

            queue = [root]
            count = 0
            while queue:
                next_queue = []
                for node in queue:
                    count += 1
                    if node.left:
                        next_queue.append(node.left)
                    if node.right:
                        next_queue.append(node.right)
                queue = next_queue

            return count


        def split_b_s_t_helper(
            root: TreeNode,
            v: int
            ) -> tuple[TreeNode, TreeNode]:
            if not root:
                return None, None

            if root.val <= v:
                small_root, large_root = split_b_s_t_helper(root.right, v)
                root.right = small_root
                return root, large_root
            else:
                small_root, large_root = split_b_s_t_helper(root.left, v)
                root.left = large_root
                return small_root, root


        small_root, large_root = split_b_s_t_helper(root, v)

        small_tree_nodes_count = count_nodes(small_root)
        large_tree_nodes_count = count_nodes(large_root)
        if small_tree_nodes_count < large_tree_nodes_count:
            return large_root
        if small_tree_nodes_count > large_tree_nodes_count:
            return small_root

        if small_root.val < large_root.val:
            return large_root
        else:
            return small_root
```
思考ログ：
- leetcodeで有料の問題だったので、lintcodeの該当する問題を解きました
  - 本家：https://leetcode.com/problems/split-bst/description/  
  - 若干内容が異なっている模様
- 一つずつ問題を解決していけばいいものを一気に解決しようとしてよく分からなくなる、という悪い癖が出た
  - 条件を満たすように木を分ける、ノードの数を数える、ノードの数を比較する、根の値を比較する
- 木の分割について
  - ```split_b_s_t_helper```は、ノードを与えると、条件を満たすように2つの木（```small_tree```, ```large_tree```）に分けてそれらのrootを返す関数
  - 現在のノードの値を確認してv以下なら、そのノードと左の子以降は```small_tree```の部分木になる
  - ノードの右の子以降は```split_b_s_t_helper```を使って```sub_small_tree```と```sub_large_tree```に分解する
  - ```small_tree```の右の子に上記の```sub_small_tree```を繋ぐ
  - ```large_tree```は上記の```sub_large_tree```を採用する
  - vより大きい場合も同様の考え方

# Step2

講師役目線でのセルフツッコミポイント：

参考にした過去ログなど：
- https://github.com/ryosuketc/leetcode_arai60/pull/36
- https://github.com/olsen-blue/Arai60/pull/48
  - 根から作っていくタイプの解法
- https://github.com/Ryotaro25/leetcode_first60/pull/50
  - https://github.com/Ryotaro25/leetcode_first60/pull/50/files#r1912058276
- https://github.com/goto-untrapped/Arai60/pull/54
- https://github.com/Yoshiki-Iwasa/Arai60/pull/41
  - 型について
    - https://github.com/Yoshiki-Iwasa/Arai60/pull/41/files#r1712673452
- https://github.com/Mike0121/LeetCode/pull/16
- https://github.com/shining-ai/leetcode/pull/47
  - stackに```("go" or "back", root, None)```の形で積んでいく方法
- https://github.com/hayashi-ay/leetcode/pull/53

```python
from lintcode import (
    TreeNode,
)

"""
Definition of TreeNode:
class TreeNode:
    def __init__(self, val):
        self.val = val
        self.left, self.right = None, None
"""

class Solution:
    """
    @param root: the given tree
    @param v: the target value
    @return: the root TreeNode after splitting
    """
    def split_b_s_t(self, root: TreeNode, v: int) -> TreeNode:
        # write your code here
        def count_nodes(root: TreeNode) -> int:
            if not root:
                return 0

            queue = [root]
            count = 0
            while queue:
                next_queue = []
                for node in queue:
                    count += 1
                    if node.left:
                        next_queue.append(node.left)
                    if node.right:
                        next_queue.append(node.right)
                queue = next_queue

            return count


        def split_b_s_t_helper(
            root: TreeNode,
            v: int
            ) -> tuple[TreeNode]:
            if not root:
                return None, None

            if root.val == v:
                small_root = root
                large_root = root.right
                small_root.right = None
                return small_root, large_root

            if root.val < v:
                small_root, large_root = split_b_s_t_helper(root.right, v)
                root.right = small_root
                return root, large_root
            else:
                small_root, large_root = split_b_s_t_helper(root.left, v)
                root.left = large_root
                return small_root, root


        small_root, large_root = split_b_s_t_helper(root, v)

        small_tree_nodes_count = count_nodes(small_root)
        large_tree_nodes_count = count_nodes(large_root)
        if small_tree_nodes_count < large_tree_nodes_count:
            return large_root
        if small_tree_nodes_count > large_tree_nodes_count:
            return small_root

        if small_root.val < large_root.val:
            return large_root
        else:
            return small_root
```
思考ログ：
- `split_b_s_t_helper`にて、`v`が見つかった時点で打ち切って良い

loop版
```python
from lintcode import (
    TreeNode,
)

"""
Definition of TreeNode:
class TreeNode:
    def __init__(self, val):
        self.val = val
        self.left, self.right = None, None
"""

class Solution:
    """
    @param root: the given tree
    @param v: the target value
    @return: the root TreeNode after splitting
    """
    def split_b_s_t(self, root: TreeNode, v: int) -> TreeNode:
        # write your code here
        def count_nodes(root: TreeNode) -> int:
            if not root:
                return 0

            queue = [root]
            count = 0
            while queue:
                next_queue = []
                for node in queue:
                    count += 1
                    if node.left:
                        next_queue.append(node.left)
                    if node.right:
                        next_queue.append(node.right)
                queue = next_queue

            return count


        def split_b_s_t_into_two_subtrees(
            root: TreeNode,
            v: int
        ) -> tuple[TreeNode]:
            node = root
            stack = []
            while node:
                stack.append(node)
                if node.val > v:
                    node = node.left
                else:
                    node = node.right

            small_subtree_root = None
            large_subtree_root = None
            while stack:
                node = stack.pop()
                if node.val > v:
                    node.left = large_subtree_root
                    large_subtree_root = node
                else:
                    node.right = small_subtree_root
                    small_subtree_root = node

            return small_subtree_root, large_subtree_root


        small_root, large_root = split_b_s_t_into_two_subtrees(root, v)

        small_tree_nodes_count = count_nodes(small_root)
        large_tree_nodes_count = count_nodes(large_root)
        if small_tree_nodes_count < large_tree_nodes_count:
            return large_root
        if small_tree_nodes_count > large_tree_nodes_count:
            return small_root

        if small_root.val < large_root.val:
            return large_root
        else:
            return small_root
```
- 同じことをやっているのにまだ再帰の方が考えやすい、という感覚がある
  - 引継ぎの考え方がまだ甘いのとloopをトップダウンで考える癖があるのが原因な気がする

# Step3

かかった時間：11min

```python
from lintcode import (
    TreeNode,
)

"""
Definition of TreeNode:
class TreeNode:
    def __init__(self, val):
        self.val = val
        self.left, self.right = None, None
"""

class Solution:
    """
    @param root: the given tree
    @param v: the target value
    @return: the root TreeNode after splitting
    """
    def split_b_s_t(self, root: TreeNode, v: int) -> TreeNode:
        # write your code here
        def count_nodes(root: TreeNode) -> int:
            if not root:
                return 0

            stack = [root]
            count = 0
            while stack:
                node = stack.pop()
                count += 1
                if node.right:
                    stack.append(node.right)
                if node.left:
                    stack.append(node.left)

            return count


        def split_b_s_t(
            root: TreeNode,
             v: int
            ) -> tuple[TreeNode]:
                if not root:
                    return None, None

                if root.val == v:
                    large_root = root.right
                    small_root = root
                    small_root.right = None
                    return small_root, large_root

                if root.val < v:
                    sub_small_root, sub_large_root = split_b_s_t(
                        root.right,
                        v
                    )
                    large_root = sub_large_root
                    small_root = root
                    small_root.right = sub_small_root
                    return small_root, large_root
                else:
                    sub_small_root, sub_large_root = split_b_s_t(
                        root.left,
                        v
                    )
                    large_root = root
                    large_root.left = sub_large_root
                    small_root = sub_small_root
                    return small_root, large_root


        small_root, large_root = split_b_s_t(root, v)
        small_tree_size = count_nodes(small_root)
        large_tree_size = count_nodes(large_root)
        if small_tree_size == large_tree_size:
            if small_root.val < large_root.val:
                return large_root
            else:
                return small_root

        if small_tree_size < large_tree_size:
            return large_root
        else:
            return small_root
```
思考ログ：
- 再帰が自然に感じるので再帰で
- 木の深さ的にも問題なさそう（<<1000）

# Step4

```python
```
思考ログ：
