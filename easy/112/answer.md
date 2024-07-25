# Step1

かかった時間：13min

計算量：
ノードの数をNとして

時間計算量：O(N)

空間計算量：O(N)

再帰
```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def hasPathSum(self, root: Optional[TreeNode], targetSum: int) -> bool:
        if not root:
            return False
        if not root.left and not root.right and root.val == targetSum:
            return True
        
        return self.hasPathSum(root.left, targetSum - root.val) or \
               self.hasPathSum(root.right, targetSum - root.val)
```
思考ログ：
- 葉を目指しながら```targetSum```を調整していけば良い
- 終端条件で手間取って時間を使う
  - 最初ノードがNoneになるまでまで見て、そこでtargetSumが0なら目的のパスを発見、のように実装していた
  - 目的のパスがあるかどうかは、葉で判定するように分離することにした
  - 葉かどうかの判定をする関数を作っても良かったかも
- ```The number of nodes in the tree is in the range [0, 5000].```なのでデフォルトでは厳しい条件

ループ
```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def hasPathSum(self, root: Optional[TreeNode], targetSum: int) -> bool:
        if not root:
            return False
        
        node_target_sum_pairs = [(root, targetSum)]
        while node_target_sum_pairs:
            node, target_sum = node_target_sum_pairs.pop()
            if not node:
                continue
            if not node.left and not node.right and node.val == target_sum:
                return True
            
            node_target_sum_pairs.append((node.left, target_sum - node.val))
            node_target_sum_pairs.append((node.right, target_sum - node.val))
        
        return False
```
思考ログ：
- ループ版、そんなに複雑でないので今回は再帰でなくてこっちでもいいかも

# Step2

講師役目線でのセルフツッコミポイント：
- 再帰実装は以下のようにした方が良さそう
  ```python
  if not root.left and not root.right:
      return root.val == targetSum
  ```

参考にした過去ログなど：
- https://github.com/kazukiii/leetcode/pull/26
  > 再帰で書く時、`return find_target_path_sum(node.left, path_sum) or find_target_path_sum(node.right, path_sum)` と書いても、left childからtrueが返ってくれば、短絡評価によってright childは呼び出されないのか -> 自分のコードは冗長な書き方をしていたので修正する必要
  - 実装上はこう書いているが、このメリットを意識していなかった   
- https://github.com/Yoshiki-Iwasa/Arai60/pull/29
- https://github.com/SuperHotDogCat/coding-interview/pull/37
  - orと|
    - https://github.com/SuperHotDogCat/coding-interview/pull/37/files#r1661771760
      > 論理演算には ブーリアン演算子 and, or および not を使ってください。2つの真偽値に対してビット演算子 &, |, ^ を適用した場合は、それぞれ論理演算 "and", "or", "xor" に相当するブール値を返します。しかしながら、論理演算子 and, or および != を使うほうが &, | や ^ を使うよりも好ましいです。
- https://github.com/fhiyo/leetcode/pull/27
- https://github.com/sakupan102/arai60-practice/pull/26
- https://github.com/Mike0121/LeetCode/pull/5
- https://github.com/rossy0213/leetcode/pull/14
- https://github.com/shining-ai/leetcode/pull/25
- https://github.com/hayashi-ay/leetcode/pull/30
- https://github.com/YukiMichishita/LeetCode/tree/main/112_path_sum

足していく方式
```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def hasPathSum(self, root: Optional[TreeNode], targetSum: int) -> bool:
        def has_path_sum_helper(root: Optional[TreeNode], total: int) -> bool:
            if not root:
                return False

            total += root.val
            if not root.left and not root.right:
                return total == targetSum
            
            return has_path_sum_helper(root.left, total) or has_path_sum_helper(root.right, total)
            
        return has_path_sum_helper(root, 0)
```
思考ログ：
- 再帰でやるなら引いていったほうが素直な感じ

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
    def hasPathSum(self, root: Optional[TreeNode], targetSum: int) -> bool:
        def is_leaf(node: Optional[TreeNode]) -> bool:
            assert node
            return not node.left and not node.right

        if not root:
            return False
        
        node_remain_pairs = [(root, targetSum)]
        while node_remain_pairs:
            node, remain = node_remain_pairs.pop()
            if not node:
                continue
            
            remain -= node.val
            if is_leaf(node) and remain == 0:
                return True
            
            node_remain_pairs.append((node.left, remain))
            node_remain_pairs.append((node.right, remain))
        
        return False
```
思考ログ：
- やっている事に対して、```target_sum```だと違和感があるので```remain```に変更
- ```is_leaf```関数を導入

# Step4

```python
```
思考ログ：

