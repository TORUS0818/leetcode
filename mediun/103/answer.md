# Step1

かかった時間：11min

計算量：
ノード数をNとして

時間計算量：O(N)

空間計算量：O(N)


```python
from collections import deque


# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def zigzagLevelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        if not root:
            return []

        nodes = deque([root])
        zigzag_level_ordered_values = []
        from_right = False
        while nodes:
            zigzag_level_ordered_values.append([])
            for _ in range(len(nodes)):
                node = nodes.popleft()
                zigzag_level_ordered_values[-1].append(node.val)

                if node.left: nodes.append(node.left)
                if node.right: nodes.append(node.right)

            if from_right:
                zigzag_level_ordered_values[-1] = zigzag_level_ordered_values[-1][::-1]
            from_right = not from_right

        return zigzag_level_ordered_values
```
思考ログ：
- 102とほぼ同じかと思いきや、効率よく反転させる方法が思いつかず苦戦
- フラグを立てて都度逆順にする方法はすぐ思いつくが、無駄なコピーコストを払うのでやりたくない気持ち
- dequeを使って、逆順の時appendじゃなくてappendleftで値を突っ込む方法もあるが、あとでlistに直す必要があり、微妙
- listのままでコストの高い```insert```をする手もあるが、、うーん
    ```python
    for _ in range(len(nodes)):
        node = nodes.popleft()
        if from_left:    
            zigzag_level_ordered_values[-1].append(node.val)
        else:
            zigzag_level_ordered_values[-1].insert(0, node.val)
        
        if node.left: nodes.append(node.left)
        if node.right: nodes.append(node.right)

    from_left = not from_left
    ```

# Step2

講師役目線でのセルフツッコミポイント：
- 効率面でのコメントがあるかも
- deque使わずにループで良いのでは

参考にした過去ログなど：
- https://github.com/kazukiii/leetcode/pull/28
  - levelの偶奇と方向をリンクさせる方法
- https://github.com/Yoshiki-Iwasa/Arai60/pull/31
- https://github.com/fhiyo/leetcode/pull/29
  - ```.reverse()```
    - 実装はこれかな
      - https://github.com/python/cpython/blob/main/Objects/listobject.c#L1491
      - インプレースで両端から要素をswapしていく
    - 序でにlistのメソッドを復習しよう
      - https://docs.python.org/ja/3/tutorial/datastructures.html
  - 再帰、DFS実装あり
  - ```yield from```とは
    - https://docs.python.org/ja/3.12/whatsnew/3.3.html#pep-380
      > これは ジェネレータ に、その操作の一部をほかのジェネレータに委譲するための式です。
      > 単純なイテレータに対して、 yield from iterable は本質的には for item in iterable: yield item への単なる速記法です
- https://github.com/YukiMichishita/LeetCode/pull/11
- https://github.com/sakupan102/arai60-practice/pull/28
- https://github.com/Mike0121/LeetCode/pull/10
- https://github.com/shining-ai/leetcode/pull/27
- https://github.com/hayashi-ay/leetcode/pull/35
  > あとPythonのreverseはCのネイティブコードが走るので変にPythonでごちゃごちゃやるよりかは早そう。  

levelの偶奇で管理する方法
```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def zigzagLevelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        if not root:
            return []

        zigzag_level_ordered_values = []
        node_level_pairs = [(root, 0)]
        while node_level_pairs:
            node, level = node_level_pairs.pop()
            while level >= len(zigzag_level_ordered_values):
                zigzag_level_ordered_values.append([])
            
            if level % 2 == 0:
                zigzag_level_ordered_values[level] \
                = zigzag_level_ordered_values[level] + [node.val]
            else:
                zigzag_level_ordered_values[level] \
                 = [node.val] + zigzag_level_ordered_values[level]
            
            if node.right: node_level_pairs.append((node.right, level + 1))
            if node.left: node_level_pairs.append((node.left, level + 1))
        
        return zigzag_level_ordered_values
```
思考ログ：
- そんなに変わり映えがないか

再帰
```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def zigzagLevelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        if not root:
            return []

        zigzag_level_ordered_values = []
        def zigzag_level_order_helper(nodes: list[Optional[TreeNode]], level: int) -> None:
            if not nodes:
                return

            while level >= len(zigzag_level_ordered_values):
                zigzag_level_ordered_values.append([])
            
            next_nodes = []
            while nodes:
                node = nodes.pop()
                zigzag_level_ordered_values[level].append(node.val)
                
                if level % 2 == 0:
                    if node.left:
                        next_nodes.append(node.left)
                    if node.right:
                        next_nodes.append(node.right)
                else:
                    if node.right:
                        next_nodes.append(node.right)
                    if node.left:
                        next_nodes.append(node.left)
            
            zigzag_level_order_helper(next_nodes, level + 1)
        
        zigzag_level_order_helper([root], 0)
        return zigzag_level_ordered_values
```
思考ログ：
- 参考
  - https://github.com/fhiyo/leetcode/pull/29/files
- 少しアレンジ
- https://github.com/fhiyo/leetcode/pull/29/files#r1659477743
  - こちらの選択肢もあるが、今回はこれで

zigzagに探索してみる
```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def zigzagLevelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        def append_node_if_exist(node):
            if node:
                new_nodes.append(node)

        if not root:
            return []
        
        nodes = [root]
        zigzag_level_ordered_values = []
        from_right = False
        while nodes:
            zigzag_level_ordered_values.append([])
            new_nodes = []
            num_same_level_nodes = len(nodes)
            for _ in range(num_same_level_nodes):
                node = nodes.pop()
                if from_right:
                    zigzag_level_ordered_values[-1].append(node.val)
                    append_node_if_exist(node.right)
                    append_node_if_exist(node.left)
                else:
                    zigzag_level_ordered_values[-1].append(node.val)
                    append_node_if_exist(node.left)
                    append_node_if_exist(node.right)
            
            nodes = new_nodes
            from_right = not from_right
        
        return zigzag_level_ordered_values
```
思考ログ：
- 取り出す方向(pop/pop_left)、子を追加する順序(right/left)があって混乱しやすいかもしれない

# Step3

かかった時間：4min

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def zigzagLevelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        if not root:
            return []

        from_right = False
        nodes = [root]
        zigzag_level_ordered_values = []
        while nodes:
            values = []
            new_nodes = []
            for node in nodes:
                values.append(node.val)
                if node.left: new_nodes.append(node.left)
                if node.right: new_nodes.append(node.right)
            
            if from_right:
                values.reverse()

            zigzag_level_ordered_values.append(values)
            from_right = not from_right
            nodes = new_nodes

        return zigzag_level_ordered_values
```
思考ログ：
- BFSが自然な感じ、```reverse```は許容して分かりやすさを取る

# Step4

```python
```
思考ログ：

