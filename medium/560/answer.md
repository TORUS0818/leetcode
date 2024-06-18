# Step1

かかった時間：21min

計算量：
nums.length=Nとして

時間計算量：O(N)

空間計算量；O(N)

```python
from collections import defaultdict


class Solution:
    def subarraySum(self, nums: List[int], k: int) -> int:
        total_num_of_target_subarrays = 0
        cumsum_num = 0
        cumsum_num_to_count = defaultdict(int)
        cumsum_num_to_count[0] = 1
        for num in nums:
            cumsum_num += num
            if cumsum_num - k in cumsum_num_to_count:
                total_num_of_target_subarrays += cumsum_num_to_count[cumsum_num - k]
            cumsum_num_to_count[cumsum_num] += 1

        return total_num_of_target_subarrays
```
思考ログ：
- ぼんやり累積和で処理するプランが頭にあったのでまずその方針で進めてみた
  - ```cumsum_num_to_count[0] = 1```を忘れて少しハマった
- 自然な方法だと、愚直にsubarrayを全部見る方法
  - ```TC = O(N^2)```なので今回は選択肢には入らない方法
  - ```nums.length <= 2 * 10^4```の```2```はそういうメッセージ？
- 累積和が単調増加（```0 <= nums[i]```）ならもう少し色々出来そう

全探索（TLE）
```python
class Solution:
    def subarraySum(self, nums: List[int], k: int) -> int:
        total_num_of_target_subarrays = 0
        for i in range(len(nums)):
            for j in range(i + 1, len(nums)+1):
                if sum(nums[i:j]) == k:
                    total_num_of_target_subarrays += 1

        return total_num_of_target_subarrays
```
思考ログ：
- TLEするが、全探索も書いておく

# Step2

講師役目線でのセルフツッコミポイント：
- 変数名がしっくりきていない

参考にした過去ログなど：
- https://github.com/goto-untrapped/Arai60/pull/28
- https://github.com/Ryotaro25/leetcode_first60/pull/17
  - https://discord.com/channels/1084280443945353267/1206101582861697046/1208473290881110117
- https://github.com/fhiyo/leetcode/pull/19
  - ```if cumsum_num - k in cumsum_num_to_count:```としなくても、```total_num_of_target_subarrays += cumsum_num_to_count[cumsum_num - k]```で直接覗きに行けば良いのか
- https://github.com/SuperHotDogCat/coding-interview/pull/29  
  ```python
  for i in range(len(nums) + 1):
      for j in cumulative_sum_to_index[cumulative_sum[i] - k]:
          if j < i:
              count += 1
  ```
  > bisect で高速化できるとは思います。
  - ここら辺は漫然と読んでたら出来ないコメントだなと思った
- https://github.com/sakupan102/arai60-practice/pull/17#discussion_r1582009907
- https://github.com/ryoooooory/LeetCode/pull/3
- https://github.com/hayashi-ay/leetcode/pull/31
  - DPの実装
- https://github.com/Exzrgs/LeetCode/pull/25

主に変数名を修正
```python
from collections import defaultdict


class Solution:
    def subarraySum(self, nums: List[int], k: int) -> int:
        prefix_sum = 0
        prefix_sum_to_frequency = defaultdict(int)
        prefix_sum_to_frequency[0] = 1
        num_of_target_subarrays = 0
        for num in nums:
            prefix_sum += num
            num_of_target_subarrays += prefix_sum_to_frequency[
                prefix_sum - k
            ]
            prefix_sum_to_frequency[prefix_sum] += 1
        
        return num_of_target_subarrays
```
思考ログ：
- 特になし

DP（TLE）
```python
from collections import defaultdict


class Solution:
    def subarraySum(self, nums: List[int], k: int) -> int:
        total_num_of_target_subarrays = 0
        prev_subarray_sum_to_frequency = defaultdict(int)
        for num in nums:
            # 現在注目しているnumまでの部分列和のヒストグラム
            subarray_sum_to_frequency = defaultdict(int)
            subarray_sum_to_frequency[num] += 1
            for prefix_sum in prev_subarray_sum_to_frequency:
                subarray_sum_to_frequency[
                    prefix_sum + num
                ] += prev_subarray_sum_to_frequency[prefix_sum]

            total_num_of_target_subarrays += subarray_sum_to_frequency[k]
            prev_subarray_sum_to_frequency = subarray_sum_to_frequency

        return total_num_of_target_subarrays
```
思考ログ：
- 全探索に加えてDPも選択肢としてすぐ出てくるといい
- やや込み入ったものにはコメント補足も一考
  - https://github.com/fhiyo/leetcode/pull/19/files
    > コメントで補足して、subarray_sum_countsとかでどうでしょう？

# Step3

かかった時間：4min

```python
from collections import defaultdict


class Solution:
    def subarraySum(self, nums: List[int], k: int) -> int:
        prefix_sum = 0
        prefix_sum_to_frequency = defaultdict(int)
        prefix_sum_to_frequency[0] = 1
        num_of_target_subarrays = 0
        for num in nums:
            prefix_sum += num
            num_of_target_subarrays += prefix_sum_to_frequency[
                prefix_sum - k
            ]
            prefix_sum_to_frequency[prefix_sum] += 1
        
        return num_of_target_subarrays
```
思考ログ：
- ```prefix_sum_to_frequency[0] = 1```を忘れるうっかりを防止するには？
  - ```nums = [k]```のようなケースをチェックしておけば気づけそう
  - 上記の初期化を忘れると、```prefix_sum_to_frequency[0] == 0```なのでおかしなことになる

# Step4

```python
```
思考ログ：

