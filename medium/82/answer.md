# Step1

かかった時間：13min

計算量： 計算量： Nをノード数として

時間計算量：O(N) 空間計算量：O(1)

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def deleteDuplicates(self, head: Optional[ListNode]) -> Optional[ListNode]:
        dummy = ListNode(val=-1, next=head)
        prev = dummy
        current = head
        while current:
            while current.next and current.val == current.next.val:
                current = current.next
            if prev.next == current:
                prev = prev.next
                current = current.next
            else:
                prev.next = current.next
                current = current.next

        return dummy.next
```
思考ログ：
- 83と同じような方針で進めるが、ポインタが一つだと足りなそう
- そういえばdummyポインタを置いて、currentと2つを動かす方法があったなと朧げながら思い出す
- あとは（自分の中で）自然な流れをコードに落とし込む
  - ```current```を使って、次のノード以降でどこまで値の重複があるか確認していく（最大最終ノードまで）
  - 重複がないパターン（現在のノードと次のノードの値が異なる）
    - この場合は```prev```と```current```を一つずつ進める
  - 重複ありのパターン（現在のノードと次以降のノードが一つ以上重複している）
    - ```current```を重複がなくなる一歩手前のノードまで進める
    - ```prev.next```を```current.next```に繋ぐ
    - ```current```を一つ進める
- 最初、重複ありのパターンで```prev```を進めるつもりで```prev = current```としてハマった
  - LinkedListの逐次処理で何が起きているのか、よく分かっていない（のでこういうことが起きる）
  - 上記のように書くと何が起きるのか、一度図示してみる（discordにあげる）

# Step2

講師役目線でのセルフツッコミポイント：
- 変数名をもう少しなんとかしようという件
  - ```prev```だけ略語なのもバランス悪い
- 再帰で書ける？

参考にした過去ログなど：
- https://discordapp.com/channels/1084280443945353267/1228700203327164487/1228789691881623704
- https://discordapp.com/channels/1084280443945353267/1227073733844406343/1228549047451910284
  - 再帰の実装例
  - 重複検知処理を関数として切り出す
- https://discordapp.com/channels/1084280443945353267/1221030192609493053/1225103398962204686
- https://discordapp.com/channels/1084280443945353267/1192736784354918470/1222902863148093502
- https://discordapp.com/channels/1084280443945353267/1200089668901937312/1206627151969787985
  - while1つで回す実装

上記を踏まえて、まず変数名を少し変えてみたバージョン
```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def deleteDuplicates(self, head: Optional[ListNode]) -> Optional[ListNode]:
        dummy_node = ListNode(-1, head)
        previous_node = dummy_node
        current_node = head
        while current_node:
            while current_node.next and current_node.val == current_node.next.val:
                current_node = current_node.next
            
            if previous_node.next == current_node:
                previous_node = previous_node.next
                current_node = current_node.next
            else:
                previous_node.next = current_node.next
                current_node = current_node.next
        
        return dummy_node.next
```
思考ログ：
- 変数名が長いので```prev_node```、```curr_node```のように省略形で揃えてもいいかも

漁った過去ログでもあった、whileを二重にしない方式の解法を再現してみる
```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def deleteDuplicates(self, head: Optional[ListNode]) -> Optional[ListNode]:
        dummy = ListNode(-1)
        previous = dummy
        current = head
        duplicated_value = None
        while current:
            if current.val == duplicated_value:
                current = current.next
                continue
            if current.next and current.val == current.next.val:
                duplicated_value = current.val
                continue
            
            previous.next = ListNode(current.val)
            previous = previous.next
            current = current.next

        return dummy.next
```
思考ログ：
- 変数名はサボった
- ポイントは重複検知した際にその値を持っておくところ

再帰で書いてみる
```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def deleteDuplicates(self, head: Optional[ListNode]) -> Optional[ListNode]:
        def delete_duplicates_helper(
            head: Optional[ListNode], 
            duplicated_val: int
            ) -> Optional[ListNode]:
            if not head:
                return head

            if head.val == duplicated_val:
                head = delete_duplicates_helper(head.next, head.val)
            elif head.next and head.val == head.next.val:
                head = delete_duplicates_helper(head.next, head.val)
            else:
                head.next = delete_duplicates_helper(head.next, None)
            
            return head
        
        unique_head = delete_duplicates_helper(head, None)
        return unique_head
```
思考ログ：
- 再帰する時にどうやって重複する情報を持たせようか悩んだ
  - 問題は、```head```と重複削除した```head.next```を接続する部分
  - 83と違って、今回の重複削除では重複したノードは全て削除される（よって、重複処理済みのLinkedListを見ただけでは、重複したものが何だったか伝わらない）
    - 例えば、```1>1>1>2```のようなLinkedListを考えて、```1```と```1>1>2```を重複処理したものの接続を考えてみる
    - この場合、後者の重複処理をすると、```2```となるので、結果は```1>2```となってしまう（期待してるのは```2```）
    - なので、```1```が重複してるという情報を引き継がないとうまくいかない
- まだ再帰に苦手意識がある、もう少し数を熟そう

# Step3

かかった時間：3min

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def deleteDuplicates(self, head: Optional[ListNode]) -> Optional[ListNode]:
        dummy_node = ListNode(-1, head)
        prev_node = dummy_node
        curr_node = head
        while curr_node:
            while curr_node.next and curr_node.val == curr_node.next.val:
                curr_node = curr_node.next
            
            if prev_node.next == curr_node:
                prev_node = prev_node.next
            else:
                prev_node.next = curr_node.next
            curr_node = curr_node.next
        
        return dummy_node.next
```
思考ログ：
- 最後の分岐の```curr_node = curr_node.next```の部分は共通なので外に出した

# Step4

変数名を修正
```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def deleteDuplicates(self, head: Optional[ListNode]) -> Optional[ListNode]:
        # 10:41
        dummy_node = ListNode(-1, head)
        prev_node = dummy_node
        node = head
        while node and node.next:
            if node.val != node.next.val:
                prev_node = prev_node.next
                node = node.next
                continue
            
            while node.next and node.val == node.next.val:
                node = node.next
            
            prev_node.next = node.next
            node = node.next
        
        return dummy_node.next
```
思考ログ：
- なるべくシンプルに意味のある名前をつけることを心がける
