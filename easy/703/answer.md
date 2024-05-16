# Step1

かかった時間：9min

計算量：
```nums```のサイズをN、addメソッドの呼び出し回数をM、ヒープサイズをKとすると

時間計算量：O(Max(N, M)logK)

空間計算量：O(K)
- 時間計算量について
  - O(N)[heapify] + O((N-K)log(K)) + O(Mlog(K)))、かな？
  - Step2のところでheapqの調査も含めて記載

```python
import heapq
class KthLargest:
    def __init__(self, k: int, nums: List[int]):
        heapq.heapify(nums)
        while len(nums) > k:
            heapq.heappop(nums)
        self.kth_nums = nums
        self.k = k

    def add(self, val: int) -> int:
        if len(self.kth_nums) < self.k:
            heapq.heappush(self.kth_nums, val)
            return self.kth_nums[0]
        
        heapq.heappushpop(self.kth_nums, val)
        return self.kth_nums[0]

# Your KthLargest object will be instantiated and called as such:
# obj = KthLargest(k, nums)
# param_1 = obj.add(val)
```
思考ログ：
- どこかで見た事がある問題
- あまり考えずに解こうとすると、以下を繰り返す感じで効率が頗る悪そう
  - 配列をソート
  - k番目に大きい要素を確認
  - 新しい値を追加
- 確か、全ての要素を保持しておく必要はない、というのがポイントだったなと思い、最小値を考える問題に変換できないか考える
  - 大きい順にK個の要素を保持すれば良さそう
  - 追加してはソートを繰り返すからデータ構造はヒープがいいか
- ```heapq```を使うも理解があやふや、Step2でドキュメントや実装を確認しよう

# Step2

講師役目線でのセルフツッコミポイント：
- ```kth_nums```は大きい順？小さい順？
- ```add```メソッドが冗長に感じつつもいい案が思いつかなかった

参考にした過去ログなど：
- https://docs.python.org/ja/3/library/heapq.html
  - heapqについて
- https://github.com/python/cpython/blob/main/Lib/heapq.py
  - heapqのcpython実装
- https://qiita.com/gteu/items/f40bdee41dd6a272a47e
  - 何故heapifyはO(n)なのか
    - 木の深さをh、注目しているノード群の深さをkとする
    - 深さkのノードは2^k個あって、それぞれh-k個だけ親子の大小関係を確認する必要がある
    - \sigma{k=0}{h}(h-k)*2^k = O(N) (h = logN)
- https://github.com/sakupan102/arai60-practice/pull/9
  - ヒープの実装あり
- https://github.com/cheeseNA/leetcode/pull/12#discussion_r1547048916
  - こちらもヒープの実装あり
  - 公開されていないメソッド名やインスタンス変数名の注意点
  - priority queue は abstract data type であり, heap はその実装の一つ
- https://github.com/hayashi-ay/leetcode/pull/54
  - こちらもヒープの実装あり
- https://discordapp.com/channels/1084280443945353267/1183683738635346001/1185972070165782688
  - クイックセレクトの話
    - クイックセレクトについて
      - https://www-ui.is.s.u-tokyo.ac.jp/~takeo/course/2019/algorithm/quickselect.pdf
      - k番目の要素を探す時、要はあるピボットに対する大小で配列を分けて（ここまではクイックソートと同じ流れ）要素数とkを比べてどっちに目的のものが入ってるか絞り込んでいくアルゴリズム
      - 再帰をしていくわけだが、後半に含まれる場合はkをk-前半の要素数、のように更新していくのがポイント
- https://discordapp.com/channels/1084280443945353267/1201211204547383386/1203363969235165204
  - heapifyでの初期化についての議論
  - 確かに全体を最小ヒープにする必要がない、、

```python
import heapq
class KthLargest:

    def __init__(self, k: int, nums: List[int]):
        self._kth_largest_nums = []
        self._k = k
        for num in nums:
            self.add(num)

    def add(self, val: int) -> int:
        heapq.heappush(self._kth_largest_nums, val)
        if len(self._kth_largest_nums) > self._k:
            heapq.heappop(self._kth_largest_nums)
        
        return self._kth_largest_nums[0]

# Your KthLargest object will be instantiated and called as such:
# obj = KthLargest(k, nums)
# param_1 = obj.add(val)
```
思考ログ：
- 初期化で```nums```全体を最小ヒープにしていたが、色々漁っているうちにこれが無駄なことに気づいた
  - ```heapify```はO(n)だし、別に良くない？と思っていたが、大事なのはヒープの大きさだった
- せっかくだから初期化でも```add```メソッドを活用した
- 大きい順なので```_kth_largest_nums```に変更
- プライベートなメンバの名前の対応

# Step3

かかった時間：3min

```python
import heapq
class KthLargest:

    def __init__(self, k: int, nums: List[int]):
        self._k = k
        self._kth_largest_nums = []
        for num in nums:
            self.add(num)

    def add(self, val: int) -> int:
        heapq.heappush(self._kth_largest_nums, val)
        if len(self._kth_largest_nums) > self._k:
            heapq.heappop(self._kth_largest_nums)

        return self._kth_largest_nums[0]

# Your KthLargest object will be instantiated and called as such:
# obj = KthLargest(k, nums)
# param_1 = obj.add(val)
```
思考ログ：
- 特になし

# Step4

```python
```
思考ログ：
