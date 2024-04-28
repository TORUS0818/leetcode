# Step1

かかった時間：11min

計算量：
計算量： Nをノード数として

時間計算量：O(N) 空間計算量：O(1)

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def deleteDuplicates(self, head: Optional[ListNode]) -> Optional[ListNode]:
        dummy = ListNode(-999)
        prev = dummy
        current = head
        while current:
            if prev.val == current.val:
                current = current.next
            else:
                prev.next = current
                prev = prev.next
                current = current.next

        prev.next = current
        return dummy.next
```
思考ログ：
- スムーズに書けず、ノートで思考の整理をした
- 考え方としては、現在地を指すノードと同じ値かどうかを先読みするノードを用意して、
  - 2つのノードの値が同じ値だったら、先読みノードを1つ進める
  - 2つのノードの値が違う値だったら、現在地ノードの次のノードに先読みノードを入れて、現在地ノードを先読みノードに置き換え、先読みノードは1つ進める
- 基本的にLinkedListの問題に苦手意識がある、、
- ```dummy```の初期化の仕方が適当すぎる、、範囲を確認したとはいえ、これは読む人に優しくない

# Step2

講師役目線でのセルフツッコミポイント：
- ```dummy```の初期化の仕方が適当、なんとかする
- if&elseの書き方がもう少し整理できるはず
```python
if prev.val != current.val:
    prev.next = current
    prev = prev.next
current = current.next
```
- 最後の後始末（prev.next = current）をしているのがなんか下手な感じ
- 昇順が保証されているのだから```<```で比較するのが自然な気がする

参考にした過去ログなど：
- https://discordapp.com/channels/1084280443945353267/1228896007279083653/1231816299718381649
- https://leetcode.com/problems/remove-duplicates-from-sorted-list/solutions/2892275/python3-o-n-beats-95-80-41-ms-simple-with-explanation/

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def deleteDuplicates(self, head: Optional[ListNode]) -> Optional[ListNode]:
        if not head:
            return head

        current = head
        while current.next:    
            if current.val < current.next.val:
                current = current.next
            else:
                current.next = current.next.next
        
        return head
```
思考ログ：
- そもそもポインタ2つ用意しなくて良いことに気づく
- もう一度、試行のプロセスを記録しておくと、
  - ポインタを一つ用意して、自分自身と次のノードの```val```を比べる
    - 同じだったら、次のノードを次の次のノードで置き換える
    - 異なる値だったらポインタを一つ進める
- 端は大丈夫か？
  - 最後のノードと一個前のノードの値が等しい場合は、次のノードに```None```が入る
  - 最後のノードと一個前のノードの値が異なる場合は、何もしない
  - ので、問題ない

# Step3

かかった時間：2min

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def deleteDuplicates(self, head: Optional[ListNode]) -> Optional[ListNode]:
        if not head:
            return head
        
        current = head
        while current.next:
            if current.val == current.next.val:
                current.next = current.next.next
            else:
                current = current.next
        
        return head
```
思考ログ：
- LinkedListの繋ぎ換えのイメージをしっかり持つようにする
  - 同じ値だったら一個飛ばして引っかけ直す
    - 次の次のノードへのアクセスはどうやるか？
      - ```current.next.next```

# Step4

```python
```
思考ログ：
