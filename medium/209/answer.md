# Step1

かかった時間：13min

計算量：

nums.lengthをNとして
- 時間計算量：O(N)
- 空間計算量：O(1)

```python
class Solution:
    def minSubArrayLen(self, target: int, nums: List[int]) -> int:
        min_len = sys.maxsize
        left = 0
        subarray_sum = 0
        for right in range(len(nums)):
            subarray_sum += nums[right]
            while subarray_sum >= target:
                min_len = min(min_len, right - left + 1)
                subarray_sum -= nums[left]
                left += 1
        
        if min_len == sys.maxsize:
            return 0
        else:
            return min_len
```
思考ログ：
- targetを超えるまでrightを動かしてnumsの要素を足していき、超えたら、target未満になるまでleftを動かしていく

全探索(TLE)
```python
class Solution:
    def minSubArrayLen(self, target: int, nums: List[int]) -> int:
        min_len = sys.maxsize
        for left in range(len(nums)):
            subarray_sum = 0
            for right in range(left, len(nums)):
                subarray_sum += nums[right]
                if subarray_sum >= target:
                    min_len = min(min_len, right - left + 1)
                
            if subarray_sum < target:
                break
        
        if min_len == sys.maxsize:
            return 0
        
        return min_len
```
思考ログ：


# Step2

講師役目線でのセルフツッコミポイント：

参考にした過去ログなど：
- https://github.com/ryosuketc/leetcode_arai60/pull/38
- https://github.com/fuga-98/arai60/pull/48
  - 2分探索の方法
- https://github.com/hroc135/leetcode/pull/46
- https://github.com/olsen-blue/Arai60/pull/50
- https://github.com/Ryotaro25/leetcode_first60/pull/53
- https://github.com/Yoshiki-Iwasa/Arai60/pull/43
  - 要素が正なので、右のポインタが最後まで進んでいて、合計がtarget未満なら、探索を切り上げることができる
- https://github.com/fhiyo/leetcode/pull/49
- https://github.com/goto-untrapped/Arai60/pull/40
  - https://github.com/goto-untrapped/Arai60/pull/40/files#r1685435343
    - numsに負の値が入っていたら？
- https://github.com/Mike0121/LeetCode/pull/22
  - https://github.com/Mike0121/LeetCode/pull/22#discussion_r1623412986
- https://github.com/shining-ai/leetcode/pull/49
  - https://github.com/shining-ai/leetcode/pull/49/files#r1564568071
    - dis
      - バイトコードの逆アセンブラ  
      - https://docs.python.org/ja/3.13/library/dis.html  
- https://github.com/hayashi-ay/leetcode/pull/51

2分探索
```python
class Solution:
    def minSubArrayLen(self, target: int, nums: List[int]) -> int:
        if sum(nums) < target:
            return 0
        min_len = len(nums)

        prefix_sum = [0] * (len(nums) + 1)
        for i in range(1, len(prefix_sum)):
            prefix_sum[i] = prefix_sum[i - 1] + nums[i - 1]
        
        for left in range(len(prefix_sum)):
            right = bisect.bisect_left(
                prefix_sum,
                prefix_sum[left] + target
            )
            if right == len(prefix_sum):
                break
            
            min_len = min(min_len, right - left)
        
        return min_len
```
思考ログ：
- 累積和について整理
  - [...Σ_i num_k...]
  - j番目とi番目の要素の差をとると、num_i+1, ... , num_jのj-i個の和になる
- bisect_leftで挿入された1つ右の累積和がtargetを超える最小の（i+1から始まっている）累積和
  - 挿入場所が右端になっているということは、（i+1から始まっている）累積和では足りなかったことを意味する
- bisect_rightを思い出しておく
  - https://docs.python.org/ja/3.13/library/bisect.html#bisect.bisect_right
  - https://github.com/TORUS0818/leetcode/pull/46#discussion_r2022377608
- 最初に解の存在を確認してlen(nums)を初期値に使うのも一つ
  - numsにマイナスがあると使えないことも留意する

# Step3

かかった時間：3min

```python
class Solution:
    def minSubArrayLen(self, target: int, nums: List[int]) -> int:
        left = 0
        subarray_sum = 0
        min_len = sys.maxsize
        for right in range(len(nums)):
            subarray_sum += nums[right]
            while subarray_sum >= target:
                min_len = min(min_len, right - left + 1)
                subarray_sum -= nums[left]
                left += 1
                
        if min_len == sys.maxsize:
            return 0
        
        return min_len
```
思考ログ：
- 初期化について
  - sys.maxsize, math.inf, len(nums) + 1, 最初に解の存在チェックをしてlen(nums), min_lenの更新があったかをフラグで管理、あたりがある

# Step4

```python
```
思考ログ：
