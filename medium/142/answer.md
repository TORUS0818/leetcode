# Step1

かかった時間：3min

計算量：
Nをノード数として

時間計算量：O(N)
空間計算量：O(N)

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    def detectCycle(self, head: Optional[ListNode]) -> Optional[ListNode]:
        found = set()
        while head:
            if head in found:
                return head

            found.add(head)
            head = head.next
        
        return None
```
思考ログ：
- 基本的にやることは前回（141）と同じ
- fast/slowを使った方法も同様に試してみる（ただし今回は少し複雑）

# Step2

講師役目線でのセルフツッコミポイント：
- 141とほぼ同じなので省略

参考にした過去ログなど：
- https://discordapp.com/channels/1084280443945353267/1228896007279083653/1231270973513404500


```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    def detectCycle(self, head: Optional[ListNode]) -> Optional[ListNode]:
        fast = head
        slow = head
        has_cycle = False
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
            if fast == slow:
                has_cycle = True
                break
        
        if has_cycle:
            slow_1 = head
            slow_2 = fast
            while slow_1 != slow_2:
                slow_1 = slow_1.next
                slow_2 = slow_2.next
            return slow_1
        else:
            return None
```
思考ログ：
- fast/slowで解いてみる
- fast/slowを使ったやり方は自分の中ではそんなにtrivialでなかったので、一応どういう原理か確認しておく
  - 以下のような設定をする
    - スタートからサイクルの合流点までの距離をx
    - サイクルの合流点からfast&slowの出会う点までの距離をy
    - サイクルの長さをc
  - fast&slowが出会うまでにslowの進んだ長さを考える
    - x+y+m1*c (m1は自然数)
  - fast&slowが出会うまでにfastの進んだ長さを考える
    - x+y+m2*c (m2は自然数)
  - またfastの進んだ距離はslowの進んだ距離の2倍ということに注意する
    - x+y+m2*c = 2(x+y+m1*c)
  - 整理すると
    - x+y = m*c (mは正の整数)
  - 以上から、fast&slowが出会った点とスタート地点から同時にslow（1stepずつ進むポインタ）を置いて走らせると、その2つが出会う場所がサイクルの合流点となる、なぜならば
    - 一方がスタート地点からx進んだ時、もう片方もfast&slowが出会った点からxだけ進んでいる
    - ということは、もう片方はfast&slowが出会った点からスタートし、何周かした後、そこからyだけ戻った点にいる (x = m*c-y)
    - ということは、この2つはスタート地点からx、つまりサイクルの合流点にいる

# Step3

かかった時間： 2min

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    def detectCycle(self, head: Optional[ListNode]) -> Optional[ListNode]:
        current = head
        found = set()
        
        while current:
            if current in found:
                return current
            
            found.add(current)
            current = current.next
        
        return None
```
思考ログ：
- Step1でスピード自体は大丈夫そうだったので、この問題は1回だけやって終了
- 141でご指摘頂いた点を加味して書いてみた

# Step4

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    def detectCycle(self, head: Optional[ListNode]) -> Optional[ListNode]:
        fast = head
        slow = head
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
            if slow == fast:
                break
        else:
            return None
        
        slow_1 = head
        slow_2 = fast
        while slow_1 != slow_2:
            slow_1 = slow_1.next
            slow_2 = slow_2.next
        return slow_1
```
修正ポイント：
- ネストが深いと読みにくいので、適度にコードを区切る癖をつける
- while-elseを活用してシンプルなコードにする
