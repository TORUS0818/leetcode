# Step1

かかった時間：13min

計算量：
ノード数をNとして
時間計算量：O(N) 空間計算量：O(1)

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def reverseList(self, head: Optional[ListNode]) -> Optional[ListNode]:
        node = head
        prev_node = None
        while node:
            next_node = node.next
            node.next = prev_node
            prev_node = node
            node = next_node
        return prev_node
```
思考ログ：
- 繋ぎかえだなあ、となる
- 例によって繋ぎかえに手間取る
  - 初め```node```1つでやろうとして失敗
  - ノードを2つに増やすも、remove_dupをやり込んだ名残で```prev_node```を```ListNode```で初期化し辺なことに
  - ```node.next```に```prev_node```を設定して、、あれ？```head```が切れてませんか、、
    - 一時的に```node.next```は退避しておかないといけないな、、
  - 上記を解消して意気揚々と```head```を```return```して先頭ノードがポロッと出てくる、、ん？？
- 空間計算量```O(N)```でよければ、普通にノードを順にスタックに積んで、出した順にLinkedList作ればいいので、個人的にやりやすいんだけどなあ
- これ、再帰実装も当然あるよね
  - 苦手だからって、脳死で"pythonなので再帰は〜"じゃなくて書けるようにしましょう -> step2

もう一度、ダメ押しでポインタの推移とかをトレースしてみる
- [0] None(prev) 1(head, node) -> 2 -> 3 -> None
- [1] None(prev) 1(head, node) -> 2(next_node) -> 3 -> None
- [1] None(prev) <- 1(head, node) 2(next_node) -> 3 -> None
- [1] None <- 1(head, node, prev) 2(next_node) -> 3 -> None
- [1] None <- 1(head, prev) 2(next_node, node) -> 3 -> None
- [2] None <- 1(head, prev) 2(node) -> 3(next_node) -> None
- [2] None <- 1(head, prev) <- 2(node) 3(next_node) -> None
- [2] None <- 1(head) <- 2(node, prev) 3(next_node) -> None
- [2] None <- 1(head) <- 2(prev) 3(next_node, node) -> None
- [3] None <- 1(head) <- 2(prev) 3(node) -> None(next_node)
- [3] None <- 1(head) <- 2(prev) <- 3(node) None(next_node)
- [3] None <- 1(head) <- 2 <- 3(node, prev) None(next_node)
- [3] None <- 1(head) <- 2 <- 3(prev) None(next_node, node)
  
# Step2

講師役目線でのセルフツッコミポイント：
- 他の選択肢は？
  - 想定解？のスタックを使ったやり方
  - 再帰

参考にした過去ログなど：
- https://github.com/hayashi-ay/leetcode/pull/13#discussion_r1596729242
  - ListNodeのループができた場合の挙動について議論している
  - ```__str__```を定義して中身を確認している、関連で以下を確認しておく
    - https://docs.python.org/3/reference/datamodel.html#object.__str
- https://github.com/sakupan102/arai60-practice/compare/ef5ce652027a...b433dfcb7b27
  - このinplaceの方法は思いつかなかった、少し動かして遊んでみる
- https://github.com/cheeseNA/leetcode/pull/11#discussion_r1537433607
  - 末尾再帰最適化について書いてある
- https://discordapp.com/channels/1084280443945353267/1195700948786491403/1198145399416946758  

まずはStep1と同様の方法で
```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def reverseList(self, head: Optional[ListNode]) -> Optional[ListNode]:
        node = head
        prev_node = None
        while node:
            next_node = node.next
            node.next = prev_node
            prev_node = node
            node = next_node
        return prev_node
```
思考ログ：
- 特になし

スタックで
```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def reverseList(self, head: Optional[ListNode]) -> Optional[ListNode]:
        node = head
        value_stack = []
        while node:
            value_stack.append(node.val)
            node = node.next
        
        dummy_node = ListNode(-1)
        reversed_node = dummy_node
        while value_stack:
            reversed_node.next = ListNode(value_stack.pop())
            reversed_node = reversed_node.next
        return dummy_node.next
```
思考ログ：
- 個人的にはこちらの方が繋ぎかえよりは描きやすい
- ただ空間計算量が増えてしまうし、コードも長くなるので繋ぎかえの方が良さそう

再帰で
```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def reverseList(self, head: Optional[ListNode]) -> Optional[ListNode]:
        if not (head and head.next):
            return head

        reversed_head = self.reverseList(head.next)
        head.next.next = head
        head.next = None
        return reversed_head
```
思考ログ：
- 再帰はやっぱり苦手
- 他の人はどのような思考をして再帰を組み立てているのか気になる
- 普段通り再帰を組み立てると私は以下のような頭の使い方をしている（1->2->3を例に）
  - 1と2->3に分けて考える
  - 2->3が2<-3と変換できる関数があるとして、1とどう繋げたら目的のものができるか考える
  - 1.nextが2を指しているのでこれを使って1.next.next=1とする（1<->2<-3）
  - 1.nextを外す（1<-2<-3）

以下の実装も試してみる
https://github.com/sakupan102/arai60-practice/compare/ef5ce652027a...b433dfcb7b27
```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def reverseList(self, head: Optional[ListNode]) -> Optional[ListNode]:
        node = head
        reversed_head = None
        while node:
            reversed_head = ListNode(node.val, reversed_head)
            node = node.next
        return reversed_head
```
思考ログ：
- これも割と自然な感じはする
- ただListNodeを毎回作っていくのは欠点か

# Step3

かかった時間：2min

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def reverseList(self, head: Optional[ListNode]) -> Optional[ListNode]:
        node = head
        reversed_head = None
        while node:
            next_node = node.next
            node.next = reversed_head
            reversed_head = node
            node = next_node
        return reversed_head
```
思考ログ：
- 最後```prev_node```を返すことになるのが分かりにくいかと思い、変数名を```prev_node```->```reversed_head```としたがどうだろうか

# Step4

```node```と```reversed_node_head```のコンタミの解消
```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def reverseList(self, head: Optional[ListNode]) -> Optional[ListNode]:
        node = head
        reversed_node_head = None
        while node:
            node_next = node.next
            reversed_node_next = reversed_node_head
            node.next = reversed_node_next
            reversed_node_head = node
            node = node_next
        return reversed_node_head
```
思考ログ：
- 少し冗長にして処理を追いやすくする選択肢も一考

再帰の実装、ヘルパー関数を用意して返り値を2つにする方法
```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def reverseList(self, head: Optional[ListNode]) -> Optional[ListNode]:
        def reverse_list_helper(node: Optional[ListNode]):
            if not (node and node.next):
                return node, node

            reversed_tail, reversed_head = reverse_list_helper(node.next)
            reversed_tail.next = node
            node.next = None
            return node, reversed_head
        
        _, reversed_head = reverse_list_helper(head)
        return reversed_head
```
思考ログ：
- こちらの方が自然に感じるし、実装もスムーズだった
