# 追加課題
再帰で書き直してみる

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def deleteDuplicates(self, head: Optional[ListNode]) -> Optional[ListNode]:
        if not head or not head.next:
            return head

        if head.val != head.next.val:
            head.next = self.deleteDuplicates(head.next)
        else:
            head = self.deleteDuplicates(head.next)

        return head
```
試行ログ:
- 重複を除いてユニークLinkedListを返す関数（```f```とする）を考える
- 問題を分割するできないか考える
  - ```head```と```head.next```以降に分割する
  - ```head.next```は```f```に入れれば目的のものが得られる（```sorted_head_next```とする）
  - あとは、```head```と```head.next```の繋ぎ目をどうするか考えれば良い
- 繋ぎ目の考え方は
  - もし、```head```と```head.next```の```val```が同じなら```head```を```sorted_head_next```で置き換える
  - もし、```head```と```head.next```の```val```が異なるなら```head.next```に```sorted_head_next```を入れる
- 片付け
  - 最後の一つのノードまでたどり着いたら、それはそのまま返せば良い
  - ```head.next```が```None```かを確認すれば良い
