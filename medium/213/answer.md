# Step1

かかった時間：8min

計算量：nums.length=Nとして

時間計算量：O(N)

空間計算量：O(N)

```python
class Solution:
    def rob(self, nums: List[int]) -> int:
        if len(nums) <= 2:
            return max(nums)
        
        def _rob_unlimited(nums: list[int]) -> int:
            '''
            note:
                numsの端が繋がってはいけないという制約を外して最大の金額を計算する
            '''
            # numsのi番目を盗むかどうかで場合わけ
            max_money_skipped_i = nums[0]
            max_money_robbed_i = nums[1]
            for i in range(2, len(nums)):
                max_money_skipped_i, max_money_robbed_i = (
                    max(max_money_skipped_i, max_money_robbed_i),
                    max_money_skipped_i + nums[i]
                )
            
            return max(max_money_skipped_i, max_money_robbed_i)
        
        return max(_rob_unlimited(nums[1:]), _rob_unlimited(nums[:-1]))
```
思考ログ：
- House Robber Iの関数がそのまま使えそうだったので再利用する形で方針を立てた
  - 最初(最後でもいいが)の要素を選ぶか選ばないかで分けて2回考えればいい
  - 最初の要素を選ぶなら最後の要素は選べないので落とす（```nums[:-1]```）
  - 最初の要素を選ばないなら最後の要素は選べるようにしておく（```nums[1:]```）
- 命名(```_rob_unlimited```)が難しいので諦めてコメントを書いた
- House Robber Iでコメントを頂いたのでSTEP2から反映してみようと思う

# Step2

講師役目線でのセルフツッコミポイント：
- ```_rob_unlimited```にインデックスを渡そう

参考にした過去ログなど：
- https://github.com/Ryotaro25/leetcode_first60/pull/38
  - 再帰
    > https://discord.com/channels/1084280443945353267/1206101582861697046/1229120056248766627  
    > ペアにして返すとメモのいらない再帰にできたりします。一応オプションとして。
- https://github.com/seal-azarashi/leetcode/pull/34
- https://github.com/Yoshiki-Iwasa/Arai60/pull/51
- https://github.com/goto-untrapped/Arai60/pull/37
- https://github.com/fhiyo/leetcode/pull/37
  - typehintについて
    -　https://docs.python.org/3/library/typing.html#typing.List
    > Note that to annotate arguments, it is preferred to use an abstract collection type such as Sequence or Iterable rather than to use list or typing.List.
    - ここでは面倒なのでやらないが、実務で使ってみよう
  - インデックスでやるか配列を投げるか問題
    > begin_index, end_indexを引数に持つように書いたが、このくらいならO(n)かけてコピーした配列を引数に持たせるようにした方が処理が分かりやすくていいかもしれない
    >  (Pythonであまりスライスによるコピーコストを気にしても意味が薄い場面も多そう。o(n)な計算量にしたいときはそれが律速になるので避けるが)。
    - 自分はどうか振り返るとインデックスでの管理が苦手というのもあり、配列で書くことが多い
- https://github.com/sakupan102/arai60-practice/pull/37
  - 命名
    > 一直線の家の並びを意識しているので`rob_house_in_line`に変更した
    - ```rob_in_line```とかでも良いかも
  - スライスについて
    -　https://docs.python.org/3/library/stdtypes.html#common-sequence-operations 
- https://github.com/shining-ai/leetcode/pull/36
- https://github.com/hayashi-ay/leetcode/pull/50

STEP1をindexで書き換える
```python
class Solution:
    def rob(self, nums: List[int]) -> int:
        if len(nums) <= 2:
            return max(nums)
        
        def _rob_in_line(left: int, right: int) -> int:
            assert 0 <= left < right <= len(nums)
            if left == right:
                return nums[left]

            # numsのleft番目を盗むかどうかで場合わけ
            max_money_skipped_last = 0
            max_money_robbed_last = 0
            while left < right:
                max_money_skipped = max(max_money_skipped_last, max_money_robbed_last)
                max_money_robbed = max_money_skipped_last + nums[left]
                max_money_skipped_last = max_money_skipped
                max_money_robbed_last = max_money_robbed
                left += 1
            
            return max(max_money_skipped_last, max_money_robbed_last)
        
        return max(_rob_in_line(1, len(nums)), _rob_in_line(0, len(nums) - 1))
```
思考ログ：
- STEP1を修正、修正点は以下
  - 命名
    - k○ndleみたいな関数名を修正
    - max_money_skipped_i, max_money_robbed_iを修正
      - https://github.com/TORUS0818/leetcode/pull/37#discussion_r1867290987
      - https://github.com/TORUS0818/leetcode/pull/37#discussion_r1867301919
  - ループの初期化部分を修正
    - https://github.com/TORUS0818/leetcode/pull/37#discussion_r1869401053
  - アンパック処理の見直し
    - https://github.com/TORUS0818/leetcode/pull/37#discussion_r1866948162

再帰
```python
class Solution:
    def rob(self, nums: List[int]) -> int:
        if len(nums) == 1:
            return nums[0]
            
        @cache
        def _rob_helper(left: int, right: int) -> int:
            assert 0 <= left < right <= len(nums)
            if right - left <= 2:
                return max(nums[left:right])

            return max(
                nums[right - 1] + _rob_helper(left, right - 2), 
                _rob_helper(left, right - 1)
            )
        
        return max(_rob_helper(1, len(nums)), _rob_helper(0, len(nums) - 1))
```
思考ログ：
- cacheのドキュメントももう一度目を通しておこう
  - https://docs.python.org/ja/3/library/functools.html#functools.cache
  - ここら辺も話題に出ていた
    > The cache is threadsafe so that the wrapped function can be used in multiple threads.
  - ここら辺も大事
    > 結果のキャッシュには辞書が使われるので、関数の位置引数およびキーワード引数は ハッシュ可能 でなくてはなりません。  
    > 引数のパターンが異なる場合は、異なる呼び出しと見なされ別々のキャッシュエントリーとなります。  
    > If typed is set to true, function arguments of different types will be cached separately.
    > キャッシュ効率の測定や maxsize パラメータの調整をしやすくするため、ラップされた関数には cache_info() 関数が追加されます。この関数は hits, misses, maxsize, currsize を示す named tuple を返します。

# Step3

かかった時間：5min

```python
class Solution:
    def rob(self, nums: List[int]) -> int:
        if len(nums) <= 1:
            return nums[0]
        
        def _rob_in_line(l: int, r: int) -> int:
            assert 0 <= l < r <= len(nums)
            # l番目の家までに盗める最大金額をl番目の家に盗みに入るかどうかで場合わけ
            max_money_skipped_last = 0
            max_money_robbed_last = 0
            while l < r:
                max_money_skipped = max(max_money_skipped_last, max_money_robbed_last)
                max_money_robbed = nums[l] + max_money_skipped_last
                max_money_skipped_last = max_money_skipped
                max_money_robbed_last = max_money_robbed
                l += 1
            
            return max(max_money_skipped_last, max_money_robbed_last)
        
        return max(_rob_in_line(0, len(nums) - 1), _rob_in_line(1, len(nums)))
```
思考ログ：
- 再帰が楽そうだが、コード量が多くてめんどくさそうな方を練習

# Step4

```python
```
思考ログ：

