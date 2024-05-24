# Step1

かかった時間：7min

計算量：
```len(nums)=N```として

時間計算量：O(NlogN)

空間計算量：O(N)

```python
from collections import defaultdict


class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        num_counter = defaultdict(int)
        for num in nums:
            num_counter[num] += 1

        top_k_frequent_elements = []
        for num, counter in sorted(
            num_counter.items(), key=lambda x: x[1], reverse=True
        )[:k]:
            top_k_frequent_elements.append(num)

        return top_k_frequent_elements
```
思考ログ：
- numsの要素の数を数えてソートして大きい順にk個取ってくる、を自然に実装する感じ
- そういえば、defaultdictとかsortedとかドキュメントを読んでおこう
  - defaultdict
    - https://docs.python.org/ja/3.10/library/collections.html#collections.defaultdict
  - sorted
    - https://docs.python.org/ja/3/howto/sorting.html
    - Timsort
      - https://ja.wikipedia.org/wiki/%E3%83%86%E3%82%A3%E3%83%A0%E3%82%BD%E3%83%BC%E3%83%88
      - 安定ソート、マージソート&挿入ソート

# Step2

講師役目線でのセルフツッコミポイント：
- 選択肢をもっと考えよう
  - sort, heap, quickselect
  - 便利なライブラリがあるか？その中身の確認

参考にした過去ログなど：
- https://github.com/YukiMichishita/LeetCode/pull/3#discussion_r1611345189
- https://github.com/t-ooka/leetcode/pull/3#discussion_r1610933361
- https://github.com/Mike0121/LeetCode/pull/3
  - defaultdictの実装あり（糖衣構文周りの話）
  - 自分も実装してみる
- https://github.com/sakupan102/arai60-practice/pull/10
- https://github.com/YukiMichishita/LeetCode/pull/3
  - quicksort/selectの実装
  - これも自分で実装してみる
- https://github.com/cheeseNA/leetcode/pull/13#discussion_r1541333578
- https://github.com/hayashi-ay/leetcode/pull/60#discussion_r1536783455
  - 色々な選択肢が網羅的に実装されている

とりあえず便利なメソッドを使う系を試してみる

Counter & most_common
```python
from collections import Counter


class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        num_counter = Counter(nums)
        return [num for num, _ in num_counter.most_common(k)]
```
heapq & nlargest
```python
import heapq
from collections import defaultdict


class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        num_counter = defaultdict(int)
        for num in nums:
            num_counter[num] += 1

        top_k_frequent_elements = [
            num for num, _ in heapq.nlargest(k, num_counter.items(), key=lambda x: x[1])
        ]

        return top_k_frequent_elements
```
思考ログ：
- Counter
  - https://docs.python.org/ja/3/library/collections.html#collections.Counter
    - 3.10から存在しない要素はカウントがゼロであるとみなされるようになった
      - ```Counter(a=1) == Counter(a=1, b=0)```
    - Counterオブジェクトの演算も独特なので注意
  - https://gist.github.com/kristi/835743
    - ```most_common```は```heapq.nlargest```

heapqを使った実装（O(Nlogk)）
```python
import heapq
from collections import defaultdict


class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        num_counter = defaultdict(int)
        for num in nums:
            num_counter[num] += 1
        
        pairs_of_count_and_num = []
        for num, count in num_counter.items():
            heapq.heappush(pairs_of_count_and_num, (count, num))
            if len(pairs_of_count_and_num) > k:
                heapq.heappop(pairs_of_count_and_num)
        
        return [num for _, num in pairs_of_count_and_num]
```
思考ログ：
- これは初手で候補に入っているべきだなと思った
  - 上からk番目〜みたいなケースではheapを想起した方がいい

QuickSelect
```python
from collections import Counter


class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        def quick_select(array: List[Tuple[int]], k: int) -> List[Tuple[int]]:
            if len(array) < k:
                return array

            pivot = array[len(array) // 2][1]
            smaller_than_pivot = [element for element in array if element[1] < pivot]
            equal_to_pivot = [element for element in array if element[1] == pivot]
            larger_than_pivot = [element for element in array if element[1] > pivot]

            if k <= len(smaller_than_pivot):
                return quick_select(smaller_than_pivot, k)

            if k <= len(smaller_than_pivot) + len(equal_to_pivot):
                candidate_elements = smaller_than_pivot + equal_to_pivot
                return candidate_elements[:k]

            return (
                smaller_than_pivot
                + equal_to_pivot
                + quick_select(
                    larger_than_pivot, k - len(smaller_than_pivot) - len(equal_to_pivot)
                )
            )

        pairs_of_num_and_minus_count = [
            (num, -count) for num, count in Counter(nums).items()
        ]
        return [num for num, _ in quick_select(pairs_of_num_and_minus_count, k)]

```
思考ログ：
- 一旦上記で実装したが、別枠（practice.md）で他の人の実装も参考に実装を考えてみたい
  - pivotの取り方とか、swapでやる方法とか
- めんどくさそうでも実装してみるとやはり理解は深まるなと当たり前のことを思った
- ただ普通に組み込みsortを使った方が速そう
  - https://discord.com/channels/1084280443945353267/1200089668901937312/1204649012695793695

# Step3

かかった時間：5min

```python
import heapq
from collections import defaultdict


class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        num_counter = defaultdict(int)
        for num in nums:
            num_counter[num] += 1

        if len(num_counter) == k:
            return [num for num, _ in num_counter.items()]

        pairs_of_count_num = []
        for num, count in num_counter.items():
            heapq.heappush(pairs_of_count_num, (count, num))
            if len(pairs_of_count_num) > k:
                heapq.heappop(pairs_of_count_num)

        return [num for _, num in pairs_of_count_num]
```
思考ログ：
- num_counterのキーがk個あるなら、heap処理の前で結果を返せる

# Step4

```python
```
思考ログ：

