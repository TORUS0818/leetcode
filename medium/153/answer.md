# Step1

かかった時間：21min

計算量：nums.length=Nとして

時間計算量：O(logN)

空間計算量：O(1)

2分探索
```python
class Solution:
    def findMin(self, nums: List[int]) -> int:
        def is_index_between_min_val_and_last_val(i: int) -> bool:
            return nums[i] <= nums[-1]

        left = 0
        right = len(nums)
        while left < right:
            middle = (left + right) // 2
            if is_index_between_min_val_and_last_val(middle):
                right = middle
            else:
                left = middle + 1
        
        return nums[left]
```
思考ログ：
- 35.Search Insert Positionと同じ感じで、半開区間で書いてみた
- 絞っていく条件がやや分かりにくいので、少し整理してみる
  - numsを回転させて昇順にしたものを[a1, a2, a3, ..., an]とする（つまりai<aj(i<j)を満たす）
  - 与えられた初期状態のnumsは[ak, ..., an, a1, ..., a(k-1)]
    - [ak, ..., an]を領域1、[a1, ..., a(k-1)]を領域2とする  
      - middle（=(left + right) // 2）が領域1にあれば探索範囲はそこより右
      - middleが領域2にあれば探索範囲はそこより左
    - 領域1、2の判別は
      - nums[middle] > nums[-1]なら領域1
      - nums[middle] <= nums[-1]なら領域2
- あとは落ち着いてleftじゃなくてnums[left]を返す
  - 何が目的だったか頭に置いておくのは結構大事だと思う

min()
```python
class Solution:
    def findMin(self, nums: List[int]) -> int:
        return min(nums)
```
思考ログ：
- 特別な要件でもない限りはこれでいい気がする

線形探索
```python
class Solution:
    def findMin(self, nums: List[int]) -> int:
        if nums[0] <= nums[-1]:
            return nums[0]

        for i in range(len(nums) - 1):
            if nums[i] > nums[i + 1]:
                return nums[i + 1]
        
        raise Exception('unreachable.')
```
- numsの要素の制約から、単調増加しなくなったポイントが最小値
  - 既にソート済みの場合は全部探索してもこの条件に当てはまらないので、事前に判別した
    - 要素が1つの場合もソート済みの特別な場合として含めた（ので```nums[0] <= nums[-1]```となっている）
      - ただ狭義単調増加という条件を考えると、この等号は混乱の元になりそうなのでコメントを書いておくのがベターか

# Step2

講師役目線でのセルフツッコミポイント：
- 空配列への対処について

参考にした過去ログなど：
- https://github.com/hroc135/leetcode/pull/40
- https://github.com/Ryotaro25/leetcode_first60/pull/46
  - チェックリスト、脳内で回してみる  
    > A 「2で割る処理がありますがこれは切り捨てでも切り上げでも構わないのでしょうか。」  
    >   ○ middle = left + (right - left) / 2;  
    >   × middle = left + (right - left + 1) / 2;  
    > B 「nums[middle] <= nums[right] とありますが、これは < でもいいですか。」  
    >   ○ nums[middle] <= nums[right]  
    >   × nums[middle] < nums[right]  
    > C 「nums[right] は、nums.back() でもいいですか。」  
    >   ○ nums[right]  
    >   × nums.back()  
    > D 「right の初期値は nums.size() でもいいですか。」  
    >   ○ right = nums.size() - 1  
    >   × right = nums.size()
    - Aが×は全部ダメ
    - (B, D) = (×, ×)もダメ
      - 要素が1つのnumsを考えればいい
      - (B, D) = (×, ○)はループに入らないので正しく動くのだが、個人的にはBは<=で持ちたい
    - (C, D) = (○, ×)もダメ
      - これ見落としていたが、ループの一回目で範囲外エラーになる
    - (B, D)と(C, D)のNGパターンに重複（(B, C, D) = (×, ○, ×)）があることに注意すると、2^4 - 2^3 - (2 + 2 - 1) = 5通りがOK
- https://github.com/seal-azarashi/leetcode/pull/39
- https://github.com/Mike0121/LeetCode/pull/44
  - 再帰の解法
- https://github.com/Yoshiki-Iwasa/Arai60/pull/35
  - https://github.com/Yoshiki-Iwasa/Arai60/pull/35/files#r1699552857  
- https://github.com/fhiyo/leetcode/pull/43
  > 右端より大きい値で一番右側にあるものを探すと考えることもできる  
  > 右端とではなく、左端との比較とすることもできる。自分は最初できないんじゃないかと思ったが、ソート済みの状態を別で考えれば `x < nums[0]` をクエリにすれば `[False, ..., True, ...]` となるようにできる。 
- https://github.com/Kitaken0107/arai60/pull/1
- https://github.com/sakupan102/arai60-practice/pull/43
- https://github.com/goto-untrapped/Arai60/pull/24
  - 匿名クラスというのを知らなかった
- https://github.com/YukiMichishita/LeetCode/pull/9
- https://github.com/thonda28/leetcode/pull/7
- https://github.com/shining-ai/leetcode/pull/42
- https://github.com/hayashi-ay/leetcode/pull/45

2分探索（左と比較ver）
```python
class Solution:
    def findMin(self, nums: List[int]) -> int:
        def is_index_between_first_val_and_max_val(i: int) -> bool:
            return nums[0] <= nums[i]

        left = 0
        right = len(nums)
        while left < right:
            middle = (left + right) // 2
            if is_index_between_first_val_and_max_val(middle):
                left = middle + 1
            else:
                right = middle
        
        return nums[0] if len(nums) == left else nums[left]
```
思考ログ：
- https://github.com/fhiyo/leetcode/pull/43
- 左と比較しても良い
  - 対称性がある解法に関しては、（自分にとって直感的に理解しやすい）片方が理解できていればまあいいか、というマインドだったが、いざ実際に手を動かしてみると意外と出来なかったりする
  - コードを読めるようになるためにも、もう少し積極的に考えてみるのが良さそう
- 参照先のコードのように初手でソートの確認をするのが良いと思うが、step1との対比のために敢えてこうしてみた
  - 左を開区間にする手もある
  - 3項演算子使わずに分けて書いた方がいいかも

再帰
```python
class Solution:
    def findMin(self, nums: List[int]) -> int:
        def find_min_helper(left: int, right: int) -> int:
            if nums[left] <= nums[right]:
                return nums[left]
            
            middle = (left + right) // 2
            if nums[middle] > nums[-1]:
                return find_min_helper(middle + 1, right)
            else:
                return find_min_helper(left, middle)
        
        return find_min_helper(0, len(nums) - 1)
```
- helper関数を作らないで、配列を渡す形にもできる
- n<=5000と小さいので再帰上限も問題ない

# Step3

かかった時間：3min

```python
class Solution:
    def findMin(self, nums: List[int]) -> int:
        def is_index_between_min_val_and_last_val(i: int) -> bool:
            return nums[i] <= nums[-1]
        
        assert len(nums) > 0
        
        left = 0
        right = len(nums)
        while left < right:
            middle = (left + right) // 2
            if is_index_between_min_val_and_last_val(middle):
                right = middle
            else:
                left = middle + 1
        
        return nums[left]
```
思考ログ：
- チェックリストの思考実験で少し理解が深まった気がする

# Step4

```python
```
思考ログ：
