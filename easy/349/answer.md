# Step1

かかった時間：1min

計算量：
nums1.length = N1, nums2.length = N2, num1+num2のuniqueな要素数をMとして

時間計算量：O(N1+N2)

空間計算量；0(M)

```python
class Solution:
    def intersection(self, nums1: List[int], nums2: List[int]) -> List[int]:
        return list(set(nums1) & set(nums2))
```
思考ログ：
- ```set```と```&```で共通部分をとってリスト化、というのは思いつく
- 一方で計算量のところで```set```について理解が浅いことに気づく
  - https://wiki.python.org/moin/TimeComplexity#set

    > he complexities for set difference s-t or s.difference(t) (set_difference()) and in-place set difference s.difference_update(t) (set_difference_update_internal()) are different!  
  - https://docs.python.org/ja/3/library/stdtypes.html#set
    
    > 集合の集合を表現するためには、内側の集合は frozenset オブジェクトでなくてはなりません。<br>
    > メソッドは、任意のイテラブルを引数として受け付けます。対して、演算子を使う版では、引数は集合でなくてはなりません。<br>
    > set のインスタンスは、 frozenset のインスタンスと、要素に基づいて比較されます。<br>
    > 集合は半順序（部分集合関係）しか定義しないので、集合のリストにおける list.sort() メソッドの出力は未定義です。<br>
    > set インスタンスと frozenset インスタンスを取り混ぜての二項演算は、第一被演算子の型を返します。


```python
class Solution:
    def intersection(self, nums1: List[int], nums2: List[int]) -> List[int]:
        found  = set()
        for num1 in nums1:
            found.add(num1)

        intersection_values = []
        for num2 in nums2:
            if num2 in found and num2 not in intersection_values:
                intersection_values.append(num2)
        
        return intersection_values
```
思考ログ：
- setのドキュメント

# Step2

講師役目線でのセルフツッコミポイント：

参考にした過去ログなど：
- https://github.com/fhiyo/leetcode/pull/16
  - ここら辺は頭にあっていいかも
      
    > ループ内部の処理が平均O(1)でできるので、nums1で回す (O(n1)) より要素数の少ない方で回す (O(min(n1, n2))) の方がコストを抑えられるメリットがあると思ってやりました。
  - 特殊ケースについて

    > nums1 = [nan] nums2 = [nan]だと何が起きますか?
- https://github.com/nittoco/leetcode/pull/15#pullrequestreview-2105624320
- https://github.com/sakupan102/arai60-practice/pull/14
- https://github.com/tshimosake/arai60/pull/3
- https://github.com/shining-ai/leetcode/pull/13

  > （nanの挙動について）IEEE754 を確認しておいてください。
  - https://pystyle.info/floating-point-numbers/
  - https://zehnpaard.hatenablog.com/entry/2022/05/30/084212
    - これは知らなかった

    > Pythonではリスト（というかcontainer）の比較は、要素の値を比較する前にポインタを比較する

    ```python
    x = float('nan')
    y = float('nan')
    
    print(x == x)
    print(x == y)
    print([x] == [x]) # これだけtrue
    print([x] == [y])
    ```
- https://github.com/hayashi-ay/leetcode/pull/21
  - num1/2を見たかどうかのon/offスイッチをリストで管理する方法
- https://github.com/YukiMichishita/LeetCode/blob/main/349_intersection_of_two_arrays/step2_sort.py

num1, num2のユニークな要素の少ない方で要素チェックする方法
```python
class Solution:
    def intersection(self, nums1: List[int], nums2: List[int]) -> List[int]:
        smaller_unique_nums, larger_unique_nums = set(nums1), set(nums2)
        if len(smaller_unique_nums) > len(larger_unique_nums):
            smaller_unique_nums, larger_unique_nums = larger_unique_nums, smaller_unique_nums
        
        intersrection_of_two_arrays = []
        for num in smaller_unique_nums:
            if num in larger_unique_nums:
                intersrection_of_two_arrays.append(num)

        return intersrection_of_two_arrays
```
思考ログ：
- こうしたちょっとした工夫でも結構早くなったりする

片方だけsetにしてチェックする方針
```python
class Solution:
    def intersection(self, nums1: List[int], nums2: List[int]) -> List[int]:
        if len(nums1) <= len(nums2):
            check_target_nums = nums1
            candidate_nums = set(nums2)
        else:
            check_target_nums = nums2
            candidate_nums = set(nums1)
        
        intersection_of_two_arrays = []
        for num in candidate_nums:
            if num in check_target_nums:
                intersection_of_two_arrays.append(num)

        return intersection_of_two_arrays
```
思考ログ：
- ```in```で確認していくので、なるべく小さい方を```list```にしたい
- それか、同じく小さい方を```list```にしておいて、そちらでループを回して最後に```set```で重複排除して返すとか

setを使わずmerge sortっぽくやっていく方法
```python
from math import isnan


class Solution:
    def intersection(self, nums1: List[int], nums2: List[int]) -> List[int]:
        for num in nums1 + nums2:
            if isnan(num):
                raise RuntimeError('nan value is included in the lists')

        index1, index2 = 0, 0
        sorted_nums1, sorted_nums2 = sorted(nums1), sorted(nums2)
        intersection_of_two_arrays = []
        while index1 < len(sorted_nums1) and index2 < len(sorted_nums2):
            if sorted_nums1[index1] < sorted_nums2[index2]:
                index1 += 1
                continue
            if sorted_nums1[index1] > sorted_nums2[index2]:
                index2 += 1
                continue

            common_num = sorted_nums1[index1]
            intersection_of_two_arrays.append(common_num)

            while index1 < len(sorted_nums1) and sorted_nums1[index1] == common_num:
                index1 += 1
            while index2 < len(sorted_nums2) and common_num == sorted_nums2[index2]:
                index2 += 1
        
        return intersection_of_two_arrays
```
思考ログ：
- nanが入っていた場合のチェックを前半に入れる
  
  > ゴミが返ってくるのと、停止しないのだと、ちょっと扱いは違うかなあと感じた

# Step3

かかった時間：1min

```python
class Solution:
    def intersection(self, nums1: List[int], nums2: List[int]) -> List[int]:
        return list(set(nums1) & set(nums2))
```
思考ログ：
- 特になし

# Step4

```python
```
思考ログ：

