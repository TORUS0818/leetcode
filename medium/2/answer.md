# Step1

かかった時間：8min

計算量： l1とl2の多い方のノード数をNとして

時間計算量：O(N) 空間計算量：O(N)

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def addTwoNumbers(self, l1: Optional[ListNode], l2: Optional[ListNode]) -> Optional[ListNode]:
        carry = 0
        dummy_node = ListNode()
        l = dummy_node
        while l1 or l2 or carry:
            l1_val = l1.val if l1 else 0
            l2_val = l2.val if l2 else 0

            sum_val = l1_val + l2_val + carry
            val = sum_val % 10
            carry = sum_val // 10
            l.next = ListNode(val, None)
            
            l = l.next
            l1 = l1.next if l1 else None
            l2 = l2.next if l2 else None
        
        return dummy_node.next
```
思考ログ：
- これも解いたことがあったのでcarryの考え方はすぐ思い出せた
  - l1, l2のvalと繰り上がりがある場合はそれも足す
    - l1かl2の片方がない場合は、valが0のノードがあると見做し計算する
  - 再度繰り上がりがあるかどうか計算
  - lのvalをセットする
  - l, l1, l2を一つずつ進める
- 変数名はStep2で何とかしよう

# Step2

講師役目線でのセルフツッコミポイント：
- 流石にvalは他の名前にした方が良さそう（見た感じdigitとかが良さそう？）

参考にした過去ログなど：
- https://github.com/SuperHotDogCat/coding-interview/pull/2#pullrequestreview-2026855904
- https://github.com/sakupan102/arai60-practice/pull/6
- https://github.com/Exzrgs/LeetCode/pull/1
- https://github.com/nittoco/leetcode/pull/2
- https://github.com/cheeseNA/leetcode/pull/2#discussion_r1527177448
  - intに代入可能な範囲の話
  - この仕様は多言語では一般的でないので注意
- https://github.com/hayashi-ay/leetcode/tree/hayashi-ay-patch-13
  - 再帰の実装例
- https://discord.com/channels/1084280443945353267/1196472827457589338/1197166381146329208
  - これも再帰の話

Step1の書き直し
```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def addTwoNumbers(self, l1: Optional[ListNode], l2: Optional[ListNode]) -> Optional[ListNode]:
        dummy_node = ListNode(-1)
        node = dummy_node
        carry = 0
        while l1 or l2 or carry:
            l1_val = l1.val if l1 else 0
            l2_val = l2.val if l2 else 0

            sum_val = l1_val + l2_val + carry
            digit = sum_val % 10
            carry = sum_val // 10
            node.next = ListNode(digit)

            node = node.next
            l1 = l1.next if l1 else None
            l2 = l2.next if l2 else None
        
        return dummy_node.next
```
思考ログ：
- 変数名を変更

再帰も試してみる
```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def addTwoNumbers(self, l1: Optional[ListNode], l2: Optional[ListNode]) -> Optional[ListNode]:
        def add_two_numbers_helper(l1, l2, carry):
            if not (l1 or l2 or carry):
                return None

            l1_val = l1.val if l1 else 0
            l2_val = l2.val if l2 else 0
            sum_val = l1_val + l2_val + carry
            digit = sum_val % 10
            carry = sum_val // 10
            
            l1_next = l1.next if l1 else None
            l2_next = l2.next if l2 else None
            node = ListNode(digit)
            node.next = add_two_numbers_helper(l1_next, l2_next, carry)

　　　　　　　return node

        return add_two_numbers_helper(l1, l2, 0)
```
思考ログ：
- ```carry```を引数に取る関数を作るのがポイント
- まだ再帰は慣れない感じがあるので時間をおいてまた書いてみる

# Step3

かかった時間：3min

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def addTwoNumbers(self, l1: Optional[ListNode], l2: Optional[ListNode]) -> Optional[ListNode]:
        # 22:32
        dummy_node = ListNode(-1)
        node = dummy_node
        carry = 0
        while l1 or l2 or carry:
            l1_val = l1.val if l1 else 0 
            l2_val = l2.val if l2 else 0
            sum_val = l1_val + l2_val + carry

            carry = sum_val // 10
            node.next = ListNode(sum_val % 10)
            
            node = node.next
            l1 = l1.next if l1 else None 
            l2 = l2.next if l2 else None
        
        return dummy_node.next
```
思考ログ：
- 可読性が劣るが、digitを定義しなくても良いかも

# Step4

```python
```
思考ログ：
