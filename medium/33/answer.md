# Step1

かかった時間：解けず

計算量： N=len(nums)として

時間計算量：O(logN)

空間計算量：O(1)

2分探索
```python
class Solution:
    def search(self, nums: List[int], target: int) -> int:
        left = 0
        right = len(nums)
        while left < right:
            middle = (left + right) // 2
            if nums[middle] == target:
                return middle

            if nums[0] <= nums[middle]:
                if nums[0] <= target < nums[middle]:
                    right = middle - 1
                else:
                    left = middle + 1
            else:
                if nums[middle] < target <= nums[-1]:
                    left = middle + 1
                else:
                    right = middle - 1
        
        return -1
```
思考ログ：
- targetの位置、middleの位置で場合わけをしていたら、よく分からなくなってしまった
  - 一回で探索する方法が思いつかなくても、前回の153.の結果を使って最小値を求めたあと、もう一度探索する方法は思いつくはず
  - 綺麗に解こうとせず、愚直でもやってみる精神が足りていない気がする
- 探すものはtarget以上になる最小の数字の場所
  - pivotの左か右の領域が必ず（狭義）単調増加列になっていることを利用する
  - 単調増加列の中にtargetがある可能性を確認し、結果に応じて探索範囲を狭める（right = middle - 1 or left = middle + 1）
  - 道中でtargetが見つかればそのindexを返し、そうでなければ見つからなかったことになるので-1を返す
- この実装だとrightより右側の領域が、target以上になる最小の数字 or それより右の数字、とはならない
  - 例えば```nums = [2,0,1]```, ```target = 3```
    - これは0で止まるが、0, 1は、numsの中で3以上になる最小の数字 or それより右の数字ではない
    - この場合は[F, F, F]なのでleft=0で止まって欲しい
      - これを満たすようにうまく条件を設定できるのか、少し考えたがよく分からなかった。。

再帰
```python
class Solution:
    def search(self, nums: List[int], target: int) -> int:
        def search_helper(left: int, right: int) -> int:
            middle = (left + right) // 2
            if nums[middle] == target:
                return middle
            
            if left == right:
                return -1
            
            if target <= nums[-1]:
                if target <= nums[middle] <= nums[-1]:
                    return search_helper(left, middle)
                else:
                    return search_helper(middle + 1, right)
            else:
                if (
                    target <= nums[middle]
                    or nums[middle] <= nums[-1]
                ):
                    return search_helper(left, middle)
                else:
                    return search_helper(middle + 1, right)
        
        return search_helper(0, len(nums))
```
- targetとnumsの端を比較してもいい
- 少し冗長になるが、全部nums[-1]と比較するようにしてもいい

O(N)ループ
```python
class Solution:
    def search(self, nums: List[int], target: int) -> int:
        assert len(nums) > 0

        for i in range(len(nums)):
            if nums[i] == target:
                return i

        return -1 
```

# Step2

講師役目線でのセルフツッコミポイント：
- 条件の処理が込み入っているので、適当に関数にした方が良さそう

参考にした過去ログなど：
- https://github.com/olsen-blue/Arai60/pull/43
  - cmp関数を初めて知った、(a < b, a == b, a > b) -> (-1, 0, 1)に割り当てる
  - python3.0以前は組み込み関数としてあったらしい
    - https://docs.python.org/ja/3.7/whatsnew/3.0.html
  - 説明が分かりやすいのでメモしておく
    >  A : 崖の前/後が F/T つまり 0/1 で表されている。この時点で、この後Bでいくら頑張っても挽回できない絶対的な格付け序列が作られてしまうイメージ。  
    >   - 今回は、後ろの方が高い格のイメージ。(年末年始の某番組を思い出しました。)  
    >   - (改良前のコードの、-2で差をつける処理は、これに近いことをしていたんですね。)
    
    >  B : Aで決められた枠の中で、さらに細かい格付け序列を作るために、cmp(x, target)を利用している。  
    >   - targetがnums内にあれば、...-1, -1, 0(targetの位置), 1, 1, ... となる。bisect_leftの返り値は、target の位置インデックスになる。  
    >   - targetがnums内になければ、...-1, -1, 1*, 1, ...  となる。bisect_leftの返り値は、1* の位置インデックスになる。  
- https://github.com/saagchicken/coding_practice/pull/14
- https://github.com/saagchicken/coding_practice/pull/9
- https://github.com/hroc135/leetcode/pull/41
  > 1. true: nums中のtargetと一致する要素とそれより右側の要素、  
  - これを考えなかった、targetが入っていなかったら全部Falseになると考えればよかった
- https://github.com/Ryotaro25/leetcode_first60/pull/47
  > 問題のモデル化  
  > 探索空間の定義  
  > 初期値の設定  
  > ループ不変条件の設定  
  > 探索ロジックの設計  
  > 検証  
  > 実行と反省  
  > といった流れで考えられるようにしたほうが良いと思います。
- https://github.com/Mike0121/LeetCode/pull/45
- https://github.com/Yoshiki-Iwasa/Arai60/pull/36
  - https://github.com/Yoshiki-Iwasa/Arai60/pull/36/files#r1704471895  
- https://github.com/fhiyo/leetcode/pull/44
  - offsetをとって昇順に戻す方法
- https://github.com/sakupan102/arai60-practice/pull/44
- https://github.com/sakzk/leetcode/pull/7/files
- https://github.com/SuperHotDogCat/coding-interview/pull/10
- https://github.com/shining-ai/leetcode/pull/43
- https://github.com/hayashi-ay/leetcode/pull/49

bisect_left*2
```python
class Solution:
    def search(self, nums: List[int], target: int) -> int:
        def is_between_min_val_and_last_val(num: int) -> bool:
            return num <= nums[-1]

        min_val_index = bisect_left(
            nums, 
            True, 
            key=is_between_min_val_and_last_val
        )

        if target <= nums[-1]:    
            lo = min_val_index
            hi = len(nums)
        else:
            lo = 0
            hi = min_val_index

        first_ge_target_index = bisect_left(
            nums, 
            target, 
            lo=lo,
            hi=hi
        )

        if nums[first_ge_target_index] == target:
            return target_index

        return -1
```
思考ログ：
- bisect_*で、keyやlo, hiを使ったことがなかったのでドキュメントを振り返りながら実装した
  - https://docs.python.org/ja/3.13/library/bisect.html
  - https://github.com/python/cpython/blob/main/Lib/bisect.py

bisect_left*1
```python
class Solution:
    def search(self, nums: List[int], target: int) -> int:
        def is_le_last_val_and_ge_target(num):
            return (num <= nums[-1], target <= num)

        first_ge_target_index = bisect_left(
            nums,
            is_le_last_val_and_ge_target(target),
            key=is_le_last_val_and_ge_target
        )

        if nums[first_ge_target_index] == target:
            return first_ge_target_index
        
        return -1
```
思考ログ：
- これは自力では思いつかないなと思った
  - [num]にtargetがあるか探す問題から[f(num)]にf(target)があるか探す問題に変換する
- 具体例で考えてみる
  - ```nums = [4,5,6,7,0,1,2]```の場合
    - ```target = 5```とすると、```key(target) = (False, True)```
      - 2（=nums[-1]）以下の要素は、全て```(True, False)```になるので、True判定
      - 2（=nums[-1]）より大きい要素は、target以上だったときにTrue判定が返る
    - ```target = 1```とすると、```key(target) = (True, True)```
      - 2（=nums[-1]）以下の要素で、target以上だったとき```（True, True）```にTrue判定が返る

# Step3

かかった時間：5min

```python
class Solution:
    def search(self, nums: List[int], target: int) -> int:
        def find_min_index(nums: list[int]) -> int:
            left = 0
            right = len(nums)
            while left < right:
                middle = (left + right) // 2
                if nums[middle] <= nums[-1]:
                    right = middle
                else:
                    left = middle + 1
            
            return left
        
        min_index = find_min_index(nums)
        if target <= nums[-1]:
            candidate_index = bisect.bisect_left(
                nums, 
                target, 
                lo=min_index, 
                hi=len(nums)
            )
        else:
            candidate_index = bisect.bisect_left(
                nums, 
                target, 
                lo=0, 
                hi=min_index
            )

        if (
            candidate_index < len(nums) 
            and nums[candidate_index] == target
        ):
            return candidate_index
        
        return -1

```
思考ログ：
- 無理せず2回探索するのが自然な気がしたのでこれに落ち着いた
- 最後の```candidate_index < len(nums) ```は必要ないのだけど、その確認を読み手にさせないためにも置いておくといいと思った
  - https://github.com/hayashi-ay/leetcode/pull/49/files#r1527168476

# Step4

```python
```
思考ログ：
