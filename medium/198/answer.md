# Step1

かかった時間：12min

計算量：nums.length=Nとして

時間計算量：O(N)

空間計算量：O(N)

DP
```python
class Solution:
    def rob(self, nums: List[int]) -> int:
        if len(nums) < 3:
            return max(nums)
        
        maximum_amount_so_far = [0] * len(nums)
        maximum_amount_so_far[0] = nums[0]
        maximum_amount_so_far[1] = max(nums[0], nums[1])
        for i in range(2, len(nums)):
            maximum_amount_so_far[i] = max(
                maximum_amount_so_far[i - 1],
                nums[i] + maximum_amount_so_far[i - 2]
            )

        return maximum_amount_so_far[-1]
```
思考ログ：
- 現時点までの最大金額を順に考えていく
  - 一個前の家でお金を盗んだかどうかで場合わけ
- ```if len(nums) <= 2:```の方が意図が分かりやすかったかも
  - 因みに```len(nums) == 1```だけ特殊扱いすれば良いのだが、ここは好みかなと

```python
class Solution:
    def rob(self, nums: List[int]) -> int:
        @cache
        def _rob_helper(i):
            if i == 1:
                return max(nums[0], nums[1])
            if i == 0:
                return nums[0]
            return max(_rob_helper(i - 2) + nums[i], _rob_helper(i - 1))
        
        return _rob_helper(len(nums) - 1)
```
思考ログ：
- numsの長さ上限が100なので、再帰も問題ない

# Step2

講師役目線でのセルフツッコミポイント：

参考にした過去ログなど：
- https://github.com/nittoco/leetcode/pull/39
  - https://github.com/nittoco/leetcode/pull/39#discussion_r1855954945
    - 最大金額を更新していく方法（多重代入）
    - 最大金額を更新していく方法
    - 前回盗んだかどうかで分けて管理する方法    
- https://github.com/seal-azarashi/leetcode/pull/33
  - 配列を使わない方法、これ前回も頭から落ちていた  
- https://github.com/Mike0121/LeetCode/pull/47
  - スレッドセーフの議論
    - GILについて
      - https://docs.python.org/ja/3.6/c-api/init.html  
- https://github.com/Ryotaro25/leetcode_first60/pull/36
- https://github.com/Yoshiki-Iwasa/Arai60/pull/50
  > 各家の前に、泥棒の手下が一人ずつ立って、前から伝言をもらって、最後のところで求めたい数字を知りたいとします。
  >「伝言」の内容は「ここまで最大いくら取れる、俺の眼の前の家に盗みに入らないとすると最大いくら取れる」の二つだけじゃないですか。
- https://github.com/Kitaken0107/arai60/pull/2
- https://github.com/goto-untrapped/Arai60/pull/36
- https://github.com/fhiyo/leetcode/pull/36
- https://github.com/YukiMichishita/LeetCode/pull/16
- https://github.com/sakupan102/arai60-practice/pull/36
  - なるほど
  ```python
  max_sum_before_1, max_sum_before_2 = max(max_sum_before_1, max_sum_before_2 + nums[i]), max_sum_before_1
  ```
- https://github.com/shining-ai/leetcode/pull/35
  - maxのdefault
    - https://docs.python.org/ja/3/library/functions.html#max
- https://github.com/hayashi-ay/leetcode/pull/48

配列を使わない方法1
```python
class Solution:
    def rob(self, nums: List[int]) -> int:
        if len(nums) <= 2:
            return max(nums)
        
        # numsのi番目を盗んだかどうかで場合わけ
        # i = 1で初期化しておく
        max_money_skipped_i = nums[0]
        max_money_robbed_i = nums[1]
        for i in range(2, len(nums)):
            max_money_skipped_i, max_money_robbed_i = (
                max(max_money_robbed_i, max_money_skipped_i), 
                max_money_skipped_i + nums[i]
            )
            
        return max(max_money_robbed_i, max_money_skipped_i)
```
思考ログ：
- O(1)の方法が頭から抜けていた
- ```nums[i]```を取るかどうかで場合わけ
- 1つ前、2つ前の最大金額を管理する方法もあるが、個人的にはこちらの方が自然だった

配列を使わない方法2
```python
class Solution:
    def rob(self, nums: List[int]) -> int:
        if len(nums) <= 2:
            return max(nums)
        
        # 現在地点から1つ前、2つ前の時点での盗める金額の最大値を管理する
        max_money_2_step_back = nums[0]
        max_money_1_step_back = max(nums[0], nums[1])
        for i in range(2, len(nums)):
            max_money = max(max_money_2_step_back + nums[i], max_money_1_step_back)
            # 次のループのためにアップデート（i+1から見て2or1 step back）
            # i-2(2-step-back) i-1(1-step-back) i(current)
            #                  i-1(2-step-back) i(1-step-back) i+1(current)
            max_money_2_step_back = max_money_1_step_back
            max_money_1_step_back = max_money
            
        return max(max_money_2_step_back, max_money_1_step_back)
```
思考ログ：
- 1つ前、2つ前の最大金額を管理する方法も書いてみたが、少し詰まった
  - 今（max_money）、1つ前、2つ前の3地点で考えればいい
  - 1つ前、2つ前を更新するには今=>1つ前、1つ前=>2つ前と推移させればいい

# Step3

かかった時間：3min

```python
class Solution:
    def rob(self, nums: List[int]) -> int:
        if len(nums) <= 2:
            return max(nums)
        
        # i番目を盗むかどうかで場合わけ
        max_money_skipped_i = nums[0]
        max_money_robbed_i = nums[1]
        for i in range(2, len(nums)):
            max_money_skipped_i, max_money_robbed_i = (
                max(max_money_skipped_i, max_money_robbed_i),
                max_money_skipped_i + nums[i]
            )

        return max(max_money_skipped_i, max_money_robbed_i)
```
思考ログ：

# Step4

```python
```
思考ログ：

