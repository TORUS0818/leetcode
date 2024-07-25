# Step1

かかった時間：5min

計算量：nums.length=Nとして、

時間計算量：O(N)

空間計算量：O(NlogN)

recursive-DFS
```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def sortedArrayToBST(self, nums: List[int]) -> Optional[TreeNode]:
        if not nums:
            return None

        root_value_index = len(nums) // 2
        root = TreeNode(val = nums[root_value_index])
        root.left = self.sortedArrayToBST(nums[:root_value_index])
        root.right = self.sortedArrayToBST(nums[root_value_index + 1:])
        
        return root
```
思考ログ：
- 直近で解き方を見ている
- 配列がソートされているので真ん中で割って根から順に木を生やせば良い
- 例によって再帰なのでスタックを気にしておく
  - 今回はlog(10^4)なので問題なさそう
- 配列のスライスがうまく機能しているか（漏れたりしてないか）少し脳内テスト  

loop-DFS
```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def sortedArrayToBST(self, nums: List[int]) -> Optional[TreeNode]:
        root = TreeNode()
        node_nums_pairs = [(root, nums)]
        while node_nums_pairs:
            node, subset_nums = node_nums_pairs.pop()
            mid_index = len(subset_nums) // 2
            node.val = subset_nums[mid_index]

            left_nums = subset_nums[:mid_index]
            if left_nums:
                node.left = TreeNode()
                node_nums_pairs.append((node.left, left_nums))
            right_nums = subset_nums[mid_index + 1:]
            if right_nums:
                node.right = TreeNode()
                node_nums_pairs.append((node.right, right_nums))
        
        return root
```
思考ログ：
- 配列を連れ回しているのは良くないか
  - インデックスで管理すればいいよねという話

# Step2

講師役目線でのセルフツッコミポイント：
- 特に思いつかなかった

参考にした過去ログなど：
- https://github.com/Yoshiki-Iwasa/Arai60/pull/28
  - midの取り方（left or right）に気が回っていなかった  
- https://github.com/kazukiii/leetcode/pull/25
  - 確かに先に木を作る方法もあった
    > dummyの値を持ったcomplete binary treeを構築して、inorder traversalしながら値をセットしても良さそう  
  - ボックス化
    - https://ja.wikipedia.org/wiki/%E3%83%9C%E3%83%83%E3%82%AF%E3%82%B9%E5%8C%96
- https://github.com/fhiyo/leetcode/pull/26
- https://github.com/Mike0121/LeetCode/pull/13
- https://github.com/hayashi-ay/leetcode/pull/29
  - ２分木でもsentinel
    - https://github.com/hayashi-ay/leetcode/pull/29/files#r1593455812    
- https://github.com/sakupan102/arai60-practice/pull/25
- https://github.com/rossy0213/leetcode/pull/13
- https://discord.com/channels/1084280443945353267/1183683738635346001/1209195844511858688

recursive-DFS（インデックス処理, 右半開区間ver）
```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def sortedArrayToBST(self, nums: List[int]) -> Optional[TreeNode]:
        def build_bst(left: int, right: int):
            if left >= right:
                return None

            mid = (left + right) // 2
            node = TreeNode(nums[mid])
            node.left = build_bst(left, mid)
            node.right = build_bst(mid + 1, right)
            return node
        
        return build_bst(0, len(nums))
```
思考ログ：
- 一つ関数を定義する必要があるが、インデックスで処理できるからエコな実装になる

recursive-DFS（インデックス処理, 閉区間ver）
```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def sortedArrayToBST(self, nums: List[int]) -> Optional[TreeNode]:
        def build_bst(left: int, right: int):
            if left > right:
                return None

            mid = (left + right) // 2
            node = TreeNode(nums[mid])
            node.left = build_bst(left, mid - 1)
            node.right = build_bst(mid + 1, right)
            return node
        
        return build_bst(0, len(nums) - 1)
```
思考ログ：
- 右半開区間の方が分かりやすいか

親と子の情報を積んでいくやり方
```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def sortedArrayToBST(self, nums: List[int]) -> Optional[TreeNode]:
        left, mid, right = 0, len(nums) // 2, len(nums)
        root = TreeNode(nums[mid])
        # [(parent_node, index_range_for_child, is_left_child)]
        parent_with_child_infos = [
            (root, (left, mid), True),
            (root, (mid + 1, right), False)
        ]
        while parent_with_child_infos:
            parent_node, (left, right), is_left = parent_with_child_infos.pop()
            if left >= right:
                continue
            mid = (left + right) // 2
            child_node = TreeNode(nums[mid])
            if is_left:
                parent_node.left = child_node
            else:
                parent_node.right = child_node
            parent_with_child_infos.append((child_node, (left, mid), True))
            parent_with_child_infos.append((child_node, (mid + 1, right), False))
        
        return root
```
思考ログ：
- 積む情報が多い、スタックの問題もあるが、再帰は簡潔に書けて良い

先に木を作ってin-orderで値を埋めていく方法
```python
from collections import deque


# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def sortedArrayToBST(self, nums: List[int]) -> Optional[TreeNode]:
        # complete binary tree
        def build_cbt(index: int):
            if index >= len(nums):
                return None
            root = TreeNode()
            root.left = build_cbt(2 * index + 1)
            root.right = build_cbt(2 * index + 2)
            return root

        nums_queue = deque(nums)
        def set_values(cbt_root: Optional[TreeNode]) -> None:
            nodes = [cbt_root]
            while nodes:
                node = nodes.pop()
                if not node:
                    continue
                set_values(node.left)
                node.val = nums_queue.popleft()
                set_values(node.right)

        cbt_root = build_cbt(0)
        set_values(cbt_root)

        return cbt_root
```
思考ログ：
- in-orderを書く機会が少ないのでせっかくなので書いておく
  - https://github.com/kazukiii/leetcode/pull/25/files
- nonlocalを使わない形に少しアレンジ

# Step3

かかった時間：3min

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def sortedArrayToBST(self, nums: List[int]) -> Optional[TreeNode]:
        if not nums:
            return None
        
        root = TreeNode()
        node_range_pairs = [(root, (0, len(nums)))]
        while node_range_pairs:
            node, (left, right) = node_range_pairs.pop()
            mid = (left + right) // 2
            node.val = nums[mid]

            if left < mid:
                node.left = TreeNode()
                node_range_pairs.append((node.left, (left, mid)))
            if mid + 1 < right:
                node.right = TreeNode()
                node_range_pairs.append((node.right, (mid + 1, right)))
        
        return root
```
思考ログ：
- 再帰でいい気がするが、スタック版を練習も兼ねて

# Step4

```python
```
思考ログ：

