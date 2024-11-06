# Step1

かかった時間：12min

計算量：nums.length = Nとして

時間計算量：O(N)

空間計算量：O(N)

```python
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        maximum_subarray_so_far = [0] * len(nums)
        maximum_subarray_so_far[0] = nums[0]
        for i in range(1, len(nums)):
            maximum_subarray_so_far[i] = max(
                maximum_subarray_so_far[i - 1] + nums[i], 
                nums[i]
            )
        
        return max(maximum_subarray_so_far)
```

思考ログ：
- 1個前までのmaximum_subarrayから更に伸ばした方がいいか、一旦切るか考える
- 最初```maximum_subarray_so_far[-1]```を反射的に返してしまった
  - 配列の要素の意味をちゃんと確認すること

# Step2

講師役目線でのセルフツッコミポイント：

参考にした過去ログなど：
- https://github.com/seal-azarashi/leetcode/pull/30
- https://github.com/Ryotaro25/leetcode_first60/pull/35
- https://github.com/Yoshiki-Iwasa/Arai60/pull/48
- https://github.com/fhiyo/leetcode/pull/33
  - 再帰の実装
- https://github.com/goto-untrapped/Arai60/pull/30
- https://github.com/YukiMichishita/LeetCode/pull/13
  - 色々な解放がコンパクトにまとめてあるので復習時に参照したい
- https://github.com/sakupan102/arai60-practice/pull/33
  - ```prefix_sum - min_prefix_sum```から```current_sum```を作る（Kadane）
    - https://github.com/sakupan102/arai60-practice/pull/33/files#r1611415355
- https://github.com/SuperHotDogCat/coding-interview/pull/14
  - 分割統治 
- https://github.com/shining-ai/leetcode/pull/33
  - 分割統治
- https://github.com/hayashi-ay/leetcode/pull/36
  - 分割統治

step1を配列を使わずに
```python
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        maximum_subarray_sum = -sys.maxsize
        prefix_sum = 0
        for num in nums:
            prefix_sum = max(prefix_sum + num, num)
            maximum_subarray_sum = max(maximum_subarray_sum, prefix_sum)
        
        return maximum_subarray_sum
```
思考ログ：
- 配列が必要かどうか、は考えておきたい（DPだと反射的に配列を用意しがち）

再帰
```python
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        def max_sub_array_helper(i: int) -> tuple[int]:
            if i == 0:
                return nums[0], nums[0]

            prefix_sum, max_sub_array_sum = max_sub_array_helper(i - 1)
            prefix_sum = max(nums[i], prefix_sum + nums[i])
            max_sub_array_sum = max(max_sub_array_sum, prefix_sum)
            return prefix_sum, max_sub_array_sum

        prefix_sum, max_sub_array_sum = max_sub_array_helper(len(nums) - 1)
        return max_sub_array_sum
```
思考ログ：
- 以下で実装されていたのでやってみる
  - https://github.com/fhiyo/leetcode/pull/33/files
  - nonlocalをやめて返り値で渡していく方式にしてみた
  - あとで見て気づいたが、ここだけ```sub_array```にしてるの気になる（元の関数はSubArrayだからsub_arrayにしたのだろう）

累積和を求めて全探索(TLE)
```python
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        prefix_sum = [0]
        for num in nums:
            prefix_sum.append(prefix_sum[-1] + num)
        
        maximum_subarray_sum = nums[0]
        for i in range(len(prefix_sum) - 1):
            for j in range(i + 1, len(prefix_sum)):
                maximum_subarray_sum = max(
                    maximum_subarray_sum,
                    prefix_sum[j] - prefix_sum[i]
                )
        
        return maximum_subarray_sum
```
思考ログ：
- 計算量的に通らないなと思って実装しなかったもの
- https://discord.com/channels/1084280443945353267/1206101582861697046/1208473290881110117
  > どうせ、1番上は思いついたけれども、価値のないものだと思って捨てたでしょう。

今まで見た中で最小のprefix_sumを記録しておく方法
```python
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        min_prefix_sum_so_far = 0
        prefix_sum = 0
        maximum_subarray_sum = nums[0]
        for num in nums:
            prefix_sum += num
            maximum_subarray_sum = max(
                maximum_subarray_sum, 
                prefix_sum - min_prefix_sum_so_far
            )
            min_prefix_sum_so_far = min(min_prefix_sum_so_far, prefix_sum)
        
        return maximum_subarray_sum
```
思考ログ：
- 累積和の履歴で最も小さかったもの（谷底）から現在までの距離を測ることで要素和が最大の部分列を探していく

# Step3

かかった時間：2min

```python
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        maximum_subarray_sum = nums[0]
        prefix_sum = 0
        for num in nums:
            prefix_sum = max(prefix_sum, 0) + num
            maximum_subarray_sum = max(maximum_subarray_sum, prefix_sum)
        
        return maximum_subarray_sum
```
思考ログ：
- ```prefix_sum```の更新で```num```を外に出してみた

# Step4

```python
```
思考ログ：

