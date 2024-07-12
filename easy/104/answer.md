# Step1

かかった時間：9min

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
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        def max_depth_helper(node: Optional[TreeNode], depth: int) -> int:
            if not node:
                return depth
            return max(
                max_depth_helper(node.left, depth + 1),
                max_depth_helper(node.right, depth + 1)
            )

        return max_depth_helper(root, 0)
```
思考ログ：
- 再帰が分かりやすいが、スタックが心配

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        if not root:
            return 0
            
        candidates = [(root, 1)]
        max_depth = 0
        while candidates:
            node, depth = candidates.pop()
            max_depth = max(max_depth, depth)
            if node.left:
                candidates.append((node.left, depth + 1))
            if node.right:
                candidates.append((node.right, depth + 1))
        
        return max_depth
```
思考ログ：
- ループ版

# Step2

講師役目線でのセルフツッコミポイント：
- Step1の再帰DFSはhelper関数を用意する必要はない

参考にした過去ログなど：
- https://github.com/kazukiii/leetcode/pull/22
  > むしろ、私の感覚は逆で、「この環境では大丈夫」なコードは可能ならば避けたいです。いつ、自分の足を撃ち抜くか分からないからです。
  - BFSはDFSスタックと同じような感じになるだろうと省略したが、depthを積まない実装もあった
- https://github.com/Yoshiki-Iwasa/Arai60/pull/23
- https://github.com/NobukiFukui/Grind75-ProgrammingTraining/pull/38  
- https://github.com/SuperHotDogCat/coding-interview/pull/34
  - Noneでも入れてしまってpop後に有効かどうかを判定するというのも一考  
- https://github.com/fhiyo/leetcode/pull/23
  > 帰りがけの処理を行う再帰をスタックで実装したバージョン。
    - うーん処理を追うのが結構大変に感じる、苦手なのかしら
  - 再帰でnonlocalでdepthを弄っていく方法もある
- https://github.com/nittoco/leetcode/pull/14
- https://github.com/sakupan102/arai60-practice/pull/21
  - stack + loopでの解法
- https://github.com/Mike0121/LeetCode/pull/6
  - https://github.com/Mike0121/LeetCode/pull/6/files#r1589881795
    - 確かに右から入れていかないと辿る順が綺麗じゃないのか  
- https://github.com/rossy0213/leetcode/pull/10
- https://github.com/hayashi-ay/leetcode/pull/22
- https://docs.google.com/document/d/1waOk82HA9I2FHYpeYZBV55trmGeaJ9EaJTGQukDiJF8/edit

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
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        if not root:
            return 0

        candidates = deque([(root)])
        depth = 0
        while candidates:
            depth += 1
            for _ in range(len(candidates)):    
                node = candidates.popleft()
                if node.left:
                    candidates.append(node.left)
                if node.right:
                    candidates.append(node.right)

        return depth
```
思考ログ：
- candidatesを2つ用意して入れ替えていく実装の方が素直かなと思ったが敢えてこうしてみた

帰りがけに処理していく方法
```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        max_depth = [None]
        candidates = [(True, root, max_depth, [None], [None])] # direction, node, depth, left_depth, right_depth
        while candidates:
            is_preorder, node, depth, left_depth, right_depth = candidates.pop()
            if is_preorder:
                if not node:
                    depth[0] = 0
                    continue
                candidates.append((False, node, depth, left_depth, right_depth))
                candidates.append((True, node.left, left_depth, [None], [None]))
                candidates.append((True, node.right, right_depth, [None], [None]))
                continue
            depth[0] = max(left_depth[0], right_depth[0]) + 1
        
        return max_depth[0]
```
思考ログ：
- いくつかdiscord上に実装があり参考にさせて頂いたが、腹落ちするのに割と時間がかかった。。
  - 最初なんでdepth関連が全部リストになっているのか分からず困っていた
    - こういうリストの使い方はまああるのだろうか、個人的には少し気になってしまう  
  - あとはdepthを共有して引き継ぎしていくところがうまく脳内トレースできなかったので紙に書いて理解した
- 理解したあと、手を動かしてみたら再現はすぐ出来たので、やはり復元できるかは理解度で決まるのだろう
- 一応やってみたが、これを選ぶかというと、、自分が苦手なのを差し引いても分かりにくい実装だと思う

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
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        if not root:
            return 0

        candidates = deque([(root, 1)])
        while candidates:
            node, depth = candidates.popleft()
            if node.left:
                candidates.append((node.left, depth + 1))
            if node.right:
                candidates.append((node.right, depth + 1))
        
        return depth
```
思考ログ：
- 個人的にはこれがバランスがいい気がする
- 再帰dfsは完結で分かりやすいが、スタックが不安

# Step4

```python
```
思考ログ：

