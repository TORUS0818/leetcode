# Step1

かかった時間：解けず

計算量：
ノード数をNとして、

時間計算量：O(N^2)

空間計算量：O(N)

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def buildTree(self, preorder: List[int], inorder: List[int]) -> Optional[TreeNode]:
        if not preorder or not inorder:
            return None

        val = preorder[0]
        inorder_split_index = inorder.index(val)
        
        left_tree_inorder = inorder[:inorder_split_index]
        right_tree_inorder = inorder[inorder_split_index + 1:]
        
        num_left_nodes = len(left_tree_inorder)
        left_tree_preorder = preorder[1:1 + num_left_nodes]
        right_tree_preorder = preorder[1 + num_left_nodes:]

        return TreeNode(
            val,
            self.buildTree(left_tree_preorder, left_tree_inorder),
            self.buildTree(right_tree_preorder,right_tree_inorder)
        )
```
思考ログ：
- 時間切れ、よく分かっていない事が分かった
- 部分木の小さな問題にしていくのは頭にあったが、pre/inorderの配列をどう分配すれば良いか分からなくなった
  - 普通に考えれば、以下の手順でやれば良いが、恐らくinorderの理解が緩かったため困惑したのだろう
    -  preorderとinorderから親探す（preorderは先頭、inorderは見つけた親ノードを検索する）
    -  inorderの親の左右のノード数が左/右部分木のノード数
    -  preorderは、親ノード、左部分木ノード達、右部分木ノード達、という構成なので、上記で求めたノード数情報をもとに切り取り
  - あと、この問題に限った事ではないが、思考をギブアップするのが早い気がする（時間というより、色々試す手間を惜しんでいるというか）
- L21の条件はなんかイケテナイ感じがある
  - ```preorder```と```inorder```は対応してるので、片方が空になることはないが、混乱しそう
  - 先に両者の長さが一致することを```assert```しといて片方だけ確認すればいいか？
- あと再帰上限要注意（3000）

# Step2

講師役目線でのセルフツッコミポイント：
- inorderのindexサーチに関して、ハッシュマップの選択肢は思い浮かんでおいた方がいいか
- スライスじゃなくてインデックスで管理しようというのも一個
- step1の解法、```num_left_nodes```なくても```inorder_split_index```使えばいいよね

参考にした過去ログなど：
- https://github.com/Yoshiki-Iwasa/Arai60/pull/33
- https://github.com/kazukiii/leetcode/pull/30
  - preorder&inorder情報を使って上から木を構築していく方法
- https://github.com/fhiyo/leetcode/pull/31
  > inorderでのsubrootのindexを求めるところは、先にkeyがノードのval, valueがinorderでのindexである辞書を作っておけばO(1)でlookupできるが、ノードの数が高々3Kなので二分探索で十分速いだろうと思いそのままにしている。
  - ここら辺抜けてた、うーん  
- https://github.com/YukiMichishita/LeetCode/pull/12
  - 個人的に整理されてて読みやすかった  
- https://github.com/sakupan102/arai60-practice/pull/30
- https://github.com/Mike0121/LeetCode/pull/12
- https://github.com/shining-ai/leetcode/pull/29#pullrequestreview-1948764345
- https://github.com/hayashi-ay/leetcode/pull/43

DFS-Loop
```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def buildTree(self, preorder: List[int], inorder: List[int]) -> Optional[TreeNode]:
        def split_inorder(inorder, split_index):
            left_inorder = inorder[:split_index]
            right_inorder = inorder[split_index + 1:]
            
            return left_inorder, right_inorder
        
        def split_preorder(preorder, num_left_nodes):
            left_preorder = preorder[1:1 + num_left_nodes]
            right_preorder = preorder[1 + num_left_nodes:]
            
            return left_preorder, right_preorder

        def append_next_candidates(node, preorder, inorder):
            inorder_split_index = inorder.index(node.val)
            left_inorder, right_inorder = split_inorder(inorder, inorder_split_index)
            left_preorder, right_preorder = split_preorder(preorder, len(left_inorder))
            
            assert len(left_preorder) == len(left_inorder)
            assert len(right_preorder) == len(right_inorder)

            if left_preorder: 
                node.left = TreeNode(left_preorder[0])
                node_and_order_pairs.append((node.left, left_preorder, left_inorder))
            if right_preorder:
                node.right = TreeNode(right_preorder[0])
                node_and_order_pairs.append((node.right, right_preorder, right_inorder))
            
        root = TreeNode(preorder[0])
        node_and_order_pairs = [(root, preorder, inorder)]
        
        while node_and_order_pairs:
            next_node, next_preorder, next_inorder = node_and_order_pairs.pop()
            if not next_node:
                continue
            
            append_next_candidates(next_node, next_preorder, next_inorder)
        
        return root
```
思考ログ：
- left/rightのコピペミスでデバッグにやや時間がかかった
- 関数の切り出しを意識してみた

preorderとinorderの情報を見てrootから順に構築する方法
```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def buildTree(self, preorder: List[int], inorder: List[int]) -> Optional[TreeNode]:
        inorder_value_to_index = {value: index for index, value in enumerate(inorder)}
        def insert_next_node(root: TreeNode, value: int) -> None:
            node = root
            while True:
                assert value != node.val
                if inorder_value_to_index[value] < inorder_value_to_index[node.val]:
                    if node.left:
                        node = node.left
                        continue
                    else:
                        node.left =  TreeNode(value)
                        break

                if inorder_value_to_index[node.val] < inorder_value_to_index[value]:
                    if node.right:
                        node = node.right
                        continue
                    else:
                        node.right =  TreeNode(value)
                        break

        root = TreeNode(preorder[0])
        for value in preorder[1:]:
            insert_next_node(root, value)
        
        return root
```
思考ログ：
- https://github.com/YukiMichishita/LeetCode/pull/12/files
  > 任意のiとjについて、i < j ⇒ preorder[i]はpreorder[j]と同じ階層かより上の階層にある  
  > 任意のiとjについて、i < j ⇒ inorder[i]はinorder[j]よりも左側にある  
  - preorderが挿入の順番、inorderが木の形の情報を教えてくれると思えばいい
    - preorderの順に挿入していく
      - 現在作成済みの木をrootから辿っていく
      - 挿入するnodeは現在地（node）より左に挿入するのか右に挿入するのか、inorderでチェック
      - 挿入すべき方向に子供がいなかったら挿入して終わり、子供がいたらポインタを進めて、inorderチェック（以下繰り返し）
- https://github.com/fhiyo/leetcode/pull/31/files
  > ここでアサーションが上手くいかなかったときにinorder_indexやnode.valを出力してあげてもよい気がしました。

スライスじゃなくインデックスで管理
```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def buildTree(self, preorder: List[int], inorder: List[int]) -> Optional[TreeNode]:
        inorder_value_to_index = {value: index for index, value in enumerate(inorder)}

        preorder_index = 0
        def build_tree_helper(inorder_left_bound: int, inorder_right_bound: int) -> TreeNode:
            nonlocal preorder_index
            if inorder_left_bound >= inorder_right_bound:
                return None

            root_val = preorder[preorder_index]
            root = TreeNode(root_val)
            preorder_index += 1
            root.left = build_tree_helper(
                inorder_left_bound, inorder_value_to_index[root_val]
            )
            root.right = build_tree_helper(
                inorder_value_to_index[root_val] + 1, inorder_right_bound
            )

            return root

        return build_tree_helper(0, len(preorder))
```
思考ログ：
- preorder_indexを内部で1ずつ増やしていくのがやや分かりにくいか

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
    def buildTree(self, preorder: List[int], inorder: List[int]) -> Optional[TreeNode]:
        if not preorder or not inorder:
            return None
        
        root_val = preorder[0]
        inorder_split_index = inorder.index(root_val)
        root = TreeNode(root_val)
        root.left = self.buildTree(preorder[1:1+inorder_split_index], inorder[:inorder_split_index])
        root.right = self.buildTree(preorder[1+inorder_split_index:], inorder[inorder_split_index + 1:])

        return root
```
思考ログ：
- ```.index```で捜索したり、スライスしたりと効率的ではないのだけど、シンプルで分かりやすい気はする
- ```preorder```のスライスに使ってる```inorder_split_index```は不親切かも、冗長でも```num_left_subtree_node```とかに入れておいた方がいいかもしれない

# Step4

```python
```
思考ログ：

