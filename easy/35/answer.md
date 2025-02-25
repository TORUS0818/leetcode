# Step1

かかった時間：2min

計算量：nums.length=Nとして

時間計算量：O(logN)

空間計算量：O(1)


```python
import bisect


class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        return bisect.bisect_left(nums, target)
```
思考ログ：
- bisect周りの仕様を復習しよう

シンプルに探索
```python
class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        for i, num in enumerate(nums):
            if target <= num:
                return i
            
        return len(nums)
```
思考ログ：

再帰
```python
class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        if len(nums) == 1:
            if nums[0] < target:
                return 1
            else:
                return 0
        
        split_index = len(nums) // 2
        if target >= nums[split_index]:
            return split_index + self.searchInsert(nums[split_index:], target)
        else:
            return self.searchInsert(nums[:split_index], target)
```
- クイックセレクトを思い出した

# Step2

講師役目線でのセルフツッコミポイント：

参考にした過去ログなど：
- https://github.com/Ryotaro25/leetcode_first60/pull/45
- https://github.com/seal-azarashi/leetcode/pull/38
  - left/rightの位置について、何が言えるか
  - 停止性問題
- https://github.com/Mike0121/LeetCode/pull/43
- https://github.com/Yoshiki-Iwasa/Arai60/pull/34
- https://github.com/nittoco/leetcode/pull/28
  - bisectのコード
    - https://github.com/python/cpython/blob/3.12/Lib/bisect.py
- https://github.com/fhiyo/leetcode/pull/42
- https://github.com/sakupan102/arai60-practice/pull/42
- https://github.com/rm3as/code_interview/pull/5
- https://github.com/shining-ai/leetcode/pull/41
- https://github.com/hayashi-ay/leetcode/pull/40
  - Arrays.binarySearch（java）を使った解法
    - bit反転と2の補数表現

bisect使わず２分探索
```python
class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        def is_larger_than_target(i: int) -> bool:
            return target <= nums[i] 

        left = 0
        right = len(nums)
        while left < right:
            middle = (left + right) // 2
            if is_larger_than_target(middle):
                right = middle
            else:
                left = middle + 1
        
        return left
```
思考ログ：
- 二分探索についてはdiscord上でかなり議論されているので、別途検索をかけてみる
- 二分探索について、もう少し一般化して思考の痕を残しておく
  - とある条件を満たす集合Xと満たさない集合X^cを考える
    - 今回のお題ではtarget以上の数が入っている配列のインデックス集合をXとおく
  - 探索する配列（arrとする）の要素について、ある整数kが存在して、arr[i]∈X^c (i<=k)、arr[i]∈X (i>k)、とできるとする（前提条件）
    - 今回のお題では配列がソートされているのでこの前提条件を満たす
  - 探索インデックスの範囲を[left, right)と設定する
  - mid = (left + right) // 2を計算し、midが条件を満たすかどうか確認して、探索範囲を更新する
    - left未満のインデックスはX^cの要素、right以上のインデックスはXの要素となるように更新する
      - mid∈X^cの時、left = mid + 1
      - mid∈Xの時、right = mid
  - 探索範囲は単調に縮まり最終的にleft==rightとなりループが停止する
  - この時left==rightのインデックスは、Xの下限となっている（X^cの上限でもある）
- パラパラ漫画も作ってみた
  - https://docs.google.com/presentation/d/1QABhB8fFeR5nt8x58dpSa3R3cNGzZ4Rcnmj-1cX74qA/edit#slide=id.p

pivotをランダムにしてみる
```python
class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        def is_larger_than_target(i: int) -> bool:
            return target <= nums[i] 

        left = 0
        right = len(nums)
        while left < right:
            pivot = random.randint(left, right - 1)
            if is_larger_than_target(pivot):
                right = pivot
            else:
                left = pivot + 1
        
        return left
```
思考ログ：

# Step3

かかった時間：2min

```python
class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        def is_larger_than_target(i: int) -> bool:
            return target <= nums[i]

        left = 0
        right = len(nums)
        while left < right:
            middle = (left + right) // 2
            if is_larger_than_target(middle):
                right = middle
            else:
                left = middle + 1
        
        return left
```
思考ログ：
- pythonでは心配はないが、オーバーフローを気にする場合はleft + (right - left) // 2の選択肢も頭においておく
- left <= middle < rightになっているのも大事なポイント

# Step4

```python
```
思考ログ：
