# Step1

かかった時間：3min

計算量：
len(nums)=Nとして

時間計算量：O(N)

空間計算量：O(N)

```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        num_to_index = dict()
        for i in range(len(nums)):
            if target - nums[i] in num_to_index:
                return [num_to_index[target - nums[i]], i]
            num_to_index[nums[i]] = i
```
思考ログ：
- 合計がtargetになるものを探す問題については、なんとなく上記の方法（```num```を記録していき、```target-num```が見つかったら答えを返す）を覚えていたのでそれをベースにまず書いてみた
- この問題は、合計がtargetになる組み合わせを見つけるだけでなく、そのインデックスを返す必要があるので、numと共にインデックスの情報も保存しておく必要がある
  - まあ```num```だけ```set```とかで管理しておいて見つけた後にインデックスの位置を検索してもいいが
- 他の選択肢としては、素直に0(N^2)のループを回すとかか
  - この場合は空間計算量はO(1)でいける
- 変数名は今回2つしか使ってないが、インデックスは```i```で十分かなと思い```i```で、インデックスを保存する辞書は今までのコメントを反映して、```num_to_index```などとした

# Step2

講師役目線でのセルフツッコミポイント：
- ```return```されなかった場合の例外処理なりは頭にあるか

参考にした過去ログなど：
- https://github.com/fhiyo/leetcode/pull/14#pullrequestreview-2092204183
- https://github.com/nittoco/leetcode/blob/nittoco-patch-1/two_sum.md
  - 例外処理
- https://github.com/kzhra/Grind41/pull/1
- https://github.com/sakupan102/arai60-practice/pull/12
- https://github.com/goto-untrapped/Arai60/pull/2
- https://discord.com/channels/1084280443945353267/1226508154833993788/1227088321537376296
  - 最後の後始末（特殊な値の返却、例外、異常終了など）について
- https://github.com/tshimosake/arai60/blob/master/hash/two_sum.py
- https://github.com/cheeseNA/leetcode/pull/1
- https://github.com/Kitaken0107/GrindEasy/pull/4
- https://github.com/hayashi-ay/leetcode/pull/14
- https://github.com/usatie/leetcode/blob/main/Arai60/01.%20Two%20Sum/ans_01_20240124.cpp
- https://docs.google.com/document/d/1750cPc_hHZvV2jxLP3esOjJCmu9s7AHe0bRiUSxP4Fs/edit
- 最初の問題？ということもあり大量に過去ログが見つかった

2重ループで書いてみる
```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        for i in range(len(nums)):
            for j in range(i + 1, len(nums)):
                if i != j and nums[i] + nums[j] == target:
                    return [i, j]
        raise Exception('unreachable')
```
思考ログ：
- ```i != j```の条件を忘れそうになる
- ```return```されない場合は例外を投げるようにした
  - 参考：https://github.com/hayashi-ay/leetcode/pull/14

# Step3

かかった時間：2min

```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        num_to_index = dict()
        for i, num in enumerate(nums):
            if target - num in num_to_index:
                return [num_to_index[target - num], i]
            num_to_index[num] = i
        
        raise Exception('unreachable')
```
思考ログ：
- ```enumerate```でインデックスと```num```を同時に出すように変更

# Step4

2重ループの無駄な処理（```i!=j```比較）部分を修正
```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        for i in range(len(nums)):
            for j in range(i + 1, len(nums)):
                if nums[i] + nums[j] == target:
                    return [i, j]
        raise Exception('unreachable')
```
思考ログ：

