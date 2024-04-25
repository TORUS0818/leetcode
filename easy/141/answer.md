# Step1

かかった時間：2min

計算量：

Nをノード数として
- 時間計算量：O(N)
- 空間計算量：O(N)

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    def hasCycle(self, head: Optional[ListNode]) -> bool:
        found = set()
        while head and head.next:
            if head in found:
                return True
            
            found.add(head)
            head = head.next
        
        return False
```
思考ログ：
- 一度解いたことがあり、すぐ思い出せた
- fast/slowを使ったもう一つの解法（というかこちらが想定解法？）も書いてみよう

# Step2

講師役目線でのセルフツッコミポイント：
- fast/slowの解法も追ってみよう
  - Step2で実装します 
- set()はどういう実装？任意のオブジェクトを受け付ける？
  - とりあえずドキュメントに目を通しておく
  - イミュータブルなfrozenset型なるものがあるのを知った
  - 集合の要素はハッシュ可能なもの限定
    - リストや辞書のようなミュータブルなコンテナはハッシュ不可能
      - ```set(['a', 'b', 'foo'])```はコンストラクタを使った初期化である点に注意
      - ミュータブルなリストを要素に持てるわけではない
    - ハッシュ値はid()で取得できる
    - 部分集合と等価性の比較は全順序付けを行う関数へと一般化することができない
      - 互いに素である二つの非空集合は、等しくなく、他方の部分集合でもないので、```a<b, a==b, a>b```にFalseを返す
    - 集合は半順序しか定義しないので、集合のリストにおける list.sort() メソッドの出力は未定義

参考にした過去ログなど：
- setの公式ドキュメント
  - https://docs.python.org/ja/3/library/stdtypes.html#set
- あまり多くなかったので、他過去ログは全般的に軽く眺めました

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    def hasCycle(self, head: Optional[ListNode]) -> bool:
        # 22:14 >> 22:17
        fast = head
        slow = head
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
            if slow == fast:
                return True

        return False
```
思考ログ：
- このfast/slowの方法は最初すぐ受け入れられず、しばらくノートで実験していた記憶があります

# Step3

かかった時間：2min

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    def hasCycle(self, head: Optional[ListNode]) -> bool:
        found = set()
        while head:
            if head in found:
                return True
            
            found.add(head)
            head = head.next
        
        return False
```
思考ログ：
- Step1でスピード自体は大丈夫そうだったので、この問題は1回だけやって終了
- そういえば、```set```に```ListNode```を放り込むのを躊躇ったことがあったが、以下の通り、ユーザー定義のクラスのインスタンスであるようなオブジェクトはデフォルトでハッシュ可能だということも覚えておこう
  - https://docs.python.org/ja/3/glossary.html#term-hashable
