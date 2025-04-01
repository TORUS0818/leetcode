# Step1

かかった時間：33min

計算量：N = len(weights), M = sum(weights)として

時間計算量：O(NlogM)

空間計算量：O(1)

```python
class Solution:
    def shipWithinDays(self, weights: List[int], days: int) -> int:
        def can_ship_capacity_in_days(capacity: int, days: int) -> bool:
            total_weight = 0
            i = 0
            while days:
                if total_weight + weights[i] > capacity:
                    total_weight = 0
                    days -= 1
                    continue

                total_weight += weights[i]
                i += 1
                if i == len(weights):
                    return True
            
            return False
        
        left = 1
        right = sum(weights)
        while left < right:
            middle = (left + right) // 2
            if can_ship_capacity_in_days(middle, days):
                right = middle
            else:
                left = middle + 1
        
        return left
```
思考ログ：
- 問題を読み違えて時間を無駄にしていた（weightsの順序を保つ制約を無視していた）
- 二分探索の探索用変数の名前を横着している、step2以降なんとかする
- 類似の問題を昔解いたのを覚えていた
  - あるcapacityにした時、days内で輸送できるか考える
  - capacity = sum(weights)とすれば、必ず1日で輸送できる
  - 輸送できるcapacityのうち最小のものを探す
    - [輸送できない, 輸送できない, ..., 輸送できない, (輸送できる], 輸送できる, ... ,輸送できる]

# Step2

講師役目線でのセルフツッコミポイント：
- weights=[], days=0の時は？

参考にした過去ログなど：
- https://github.com/saagchicken/coding_practice/pull/15
  - 探索範囲は1からでなくmax(weights)からでいい
- https://github.com/saagchicken/coding_practice/pull/10
- https://github.com/olsen-blue/Arai60/pull/44
- https://github.com/hroc135/leetcode/pull/42
  - goのbinary-searchの実装  
- https://github.com/Ryotaro25/leetcode_first60/pull/51
  - Makefileについて
- https://github.com/Mike0121/LeetCode/pull/46
- https://github.com/Yoshiki-Iwasa/Arai60/pull/37
- https://github.com/rossy0213/leetcode/pull/26
- https://github.com/goto-untrapped/Arai60/pull/41
- https://github.com/fhiyo/leetcode/pull/45
  - https://github.com/fhiyo/leetcode/pull/45/files#r1682470839
- https://github.com/sakupan102/arai60-practice/pull/45
- https://github.com/SuperHotDogCat/coding-interview/pull/27
  - https://github.com/SuperHotDogCat/coding-interview/pull/27/files#r1631399685
  - 命名はなかなか難しいなと
  - 実態を表すよう無理に情報を詰め込むと拗らせるので、そういう場合はなるべくシンプルにして、必要ならコメント補足を心掛ける
- https://github.com/YukiMichishita/LeetCode/pull/10
  - bisect_left/right
    - bisect_rightを勘違いしていた（等号は含まない）
      - https://docs.python.org/ja/3.13/library/bisect.html  
        >  all(elem > x for elem in a[ip : hi]) is true for the right slice.
- https://github.com/shining-ai/leetcode/pull/44
- https://github.com/hayashi-ay/leetcode/pull/55

所要日数を計算する
```python
class Solution:
    def shipWithinDays(self, weights: List[int], days: int) -> int:
        assert days > 0
        if not weights:
            return 0

        def calculate_shipping_days(capacity: int) -> int:
            total_weight = 0
            days_required = 1
            for weight in weights:
                if total_weight + weight > capacity:
                    total_weight = 0
                    days_required += 1
                
                total_weight += weight

            return days_required

        low = max(weights)
        high = sum(weights)
        while low < high:
            candidate_capacity = (low + high) // 2
            if calculate_shipping_days(candidate_capacity) <= days:
                high = candidate_capacity
            else:
                low = candidate_capacity + 1
        
        return low
```
思考ログ：

bisect_right（練習）
```python
class Solution:
    def shipWithinDays(self, weights: List[int], days: int) -> int:
        assert days > 0
        if not weights:
            return 0
            
        def can_ship(capacity: int) -> bool:
            total_weight = 0
            days_required = 1
            for weight in weights:
                if total_weight + weight > capacity:
                    total_weight = 0
                    days_required += 1

                total_weight += weight
            
            return days_required <= days
        
        return bisect.bisect_right(
            range(sum(weights) + 1),
            False,
            lo=max(weights),
            hi=sum(weights) + 1,
            key=can_ship
        )
```
思考ログ：
- 今回の課題で敢えて使うこともないが、bisect_rightの練習で書いてみた
- 関数名、```can_ship```くらいでいい気もするので少し変更

# Step3

かかった時間：5min

```python
class Solution:
    def shipWithinDays(self, weights: List[int], days: int) -> int:
        assert days > 0
        if not weights:
            return 0
        
        def can_ship(capacity: int) -> bool:
            total_weight = 0
            days_required = 1
            for weight in weights:
                if weight > capacity:
                    return False

                if total_weight + weight > capacity:
                    total_weight = 0
                    days_required += 1
                
                total_weight += weight
            
            return days_required <= days
        
        return bisect.bisect_left(
            range(sum(weights)),
            True,
            key=can_ship,
            lo=max(weights),
            hi=sum(weights)
        )
```
思考ログ：
- ```weight > capacity```の早期リターンを追加

# Step4

```python
```
思考ログ：
