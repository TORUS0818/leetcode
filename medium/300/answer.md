# Step1

かかった時間：7min

計算量：
nums.length=Nとして

時間計算量：O(NlogN)

空間計算量：O(N)

```python
import bisect


class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        lis = []
        for num in nums:
            if not lis or lis[-1] < num:
                lis.append(num)
                continue
            
            i = bisect.bisect_left(lis, num)
            lis[i] = num
        
        return len(lis)
```
思考ログ：
- 直近で解いて覚えていた
  - 最初はこの解法を受け入れるのに時間がかかった気がする
    - 例えば、[10, 11, 12, 1, 2]だと、変数lisは
      - [10]
      - [10, 11]
      - [10, 11, 12]
      - [1, 11, 12]
      - [1, 2, 12]
    - 大事なのは大小関係なので、それを崩さないように挿入していけば、２つの候補を同時に管理できる
- 名前がlisじゃないという議論があったと思うが、一度ロムってから考えよう、、

ボトムアップDP
```python
class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        lis = [1] * len(nums)
        for i in range(1, len(nums)):
            for j in range(i):
                if nums[j] < nums[i]:
                    lis[i] = max(lis[i], lis[j] + 1)
        
        return max(lis)
```
- 一番分かりやすいし思いつく解法では
- 命名の問題が同様にある

# Step2

講師役目線でのセルフツッコミポイント：
- 命名関連の選択肢は再度考えた方がいい
  - 名前を工夫する
  - コメントで補足する

参考にした過去ログなど：
- https://github.com/kazukiii/leetcode/pull/32
  - Seg木
- https://github.com/seal-azarashi/leetcode/pull/28
  - Javaの標準ライブラリの二分探索メソッドの返り値が悩ましい
- https://github.com/Ryotaro25/leetcode_first60/pull/34
- https://github.com/Yoshiki-Iwasa/Arai60/pull/46
  - 命名、lisに関して
- https://github.com/fhiyo/leetcode/pull/32
  - BIT、Seg木
  - BITはロジックを読んどく、Seg木は実装してみよう
- https://github.com/SuperHotDogCat/coding-interview/pull/28
- https://github.com/sakupan102/arai60-practice/pull/32
- https://github.com/goto-untrapped/Arai60/pull/18
- https://github.com/YukiMichishita/LeetCode/pull/7
- https://github.com/shining-ai/leetcode/pull/31
  - BIT、Seg木
- https://github.com/hayashi-ay/leetcode/pull/27


セグ木
```python
IDENTITY_ELEMENT = 0

class SegTree:
    def __init__(self, n: int):
        leaf_n = 1
        while leaf_n < n:
            leaf_n *= 2
        self.leaf_n = leaf_n
        self.tree = [0] * (2 * leaf_n - 1)

    def _convert_to_tree_index(self, i: int) -> int:
        return i + self.leaf_n - 1

    def query(self, l: int, r: int):
        '''
        note: 
            右半開区間でquery範囲を指定する（x ∈ [l, r)）
        '''
        l = self._convert_to_tree_index(l)
        r = self._convert_to_tree_index(r)

        result = IDENTITY_ELEMENT
        while l < r:
            # if l is left child
            if l % 2 == 0:
                result = max(result, self.tree[l])
            # if r-1 is right child
            if r % 2 == 0:
                result = max(result, self.tree[r - 1])
            l = (l - 1) // 2
            r = (r - 1) // 2

        return result

    def update(self, i: int, val: int) -> None:
        i = self._convert_to_tree_index(i)
        self.tree[i] = val

        # get parent_index
        i = (i - 1) // 2
        while i >= 0:
            self.tree[i] = max(self.tree[2 * i + 1], self.tree[2 * i + 2])
            i = (i - 1) // 2

class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        def compress(nums: list[int]) -> list[int]:  
            num_to_compressed_num = {}
            for i, num in enumerate(sorted(set(nums))):
                num_to_compressed_num[num] = i
            
            compressed_num = [num_to_compressed_num[num] for num in nums]
            return compressed_num

        compressed_num = compress(nums)
        n = len(set(compressed_num))
        st = SegTree(n)
        for i, num in enumerate(compressed_num):
            res = st.query(0, num)
            st.update(num, res + 1)
        
        return st.tree[0]
```
思考ログ：
- この問題に対してoverkillなのは分かっているが、皆さんが結構実装されているのでこの機会にお勉強してみた
- シンプルなセグ木の理解自体はそんなにかからなかったが、寧ろこの問題での使い方の部分で躓いた（それはセグ木を理解できていな(ry）
- 考え方を簡単にまとめておく
  - 座標圧縮
    - 大小の情報だけあればいいので大きさ順に番号を振り直す
  - ```query```  で自分より小さい数字で終わる最長増加部分列(LIS)を探す
  - ```update```で自分の担当部分のLISの情報を更新する
    - ```query```で返ってきた結果を一つ大きくすればいい
    - 自分の先祖の情報も全部更新する
  - 最後にrootの値を取ってくればいい
- TIPS
  - 木は配列で管理すると扱いやすい
    - （0-indexで考えて）親から子供のインデックスを知りたい時は、親のインデックス * 2 + 1, 親のインデックス * 2 + 2
    - 子供から親のインデックスを知りたい時は、(子のインデックス - 1) // 2
    - ビット演算でもできるが、今回は（自分にとっての）分かりやすさを優先した
  - 更新と区間の情報取得が入り乱れている時に有用
    - 更新が一回だけの場合などは累積和のロジックなど、前処理をしておけば良いが、道中更新があると困る
  - 任意の区間を二分木のノードに分割できるのがミソ
  - 端の扱いなどをちゃんと意識して書いていかないと容易にバグる

# Step3

かかった時間：1min

```python
import bisect


class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        # 長さがindex+1になるような任意の増加部分列を考えた時
        # それらの最後の要素のうち最小のものを記録するための配列
        # ex) [4,6], [3,5], [1,2]ならmin(6, 5, 2) = 2が入る
        minimum_last_val_subseq = []
        for num in nums:
            if not minimum_last_val_subseq \
                or minimum_last_val_subseq[-1] < num:
                minimum_last_val_subseq.append(num)
            
            i = bisect.bisect_left(minimum_last_val_subseq, num)
            minimum_last_val_subseq[i] = num
        
        return len(minimum_last_val_subseq)
```
思考ログ：
- コメントで補足する癖をつけよう
  - あんまり冗長にやってもアレなので匙加減と、逆に無駄に不要なコメントを残さないようにも気を付ける
- GPTに聞いてみた
  > 候補となる変数名  
  > min_last_elements_of_increasing_subsequences  
  > 変数が持つ情報をそのまま説明する形で、冗長ですが内容が明確です。長さを意識しつつ正確さを優先したい場合に適しています。  
  > min_ends_of_increasing_subsequences  
  > 1の名前を少し短くした形です。「各長さごとの増加部分列の最小の終点（最後の要素）」という意味が伝わりやすいでしょう。  
  > increasing_subseq_min_ends  
  > 読みやすさを保ちながら短縮した名前で、PEP 8の命名規則に合致しています。変数の用途や内容が概ね分かりやすくなります。  
  > min_last_of_increasing_sublists  
  > subsequencesではなくsublist（部分リスト）という名前で少し簡略化していますが、役割は理解しやすいでしょう。  
  > おすすめ  
  > min_ends_of_increasing_subsequencesが適切です。この名前は要素が増加部分列の最終要素の最小値であることを明確に示し、長さも扱いやすいです。  
  > もし簡潔さを重視する場合は、increasing_subseq_min_endsも良い選択です。
  - なお、質問は以下
    > 今pythonでコードを書いているのですが、変数名の命名で困っています。
    > この変数Xは配列で、以下のような特徴を持ちます。
    > - X[i]は、ある数列Yの長さi+1の増加部分列が対応します。  
    > - X[i]にはそのような部分列の最終要素の中で最小のものが入ります。  
    > 例えばY = [1,3,2,4]を考えるとX[1]に対応する増加部分列は[1,3], [1,2], [1,4], [3,4], [2,4]となり、min(3, 2, 4, 4, 4) = 2なのでX[1]=2となります。  
    > Xとして適切な名前を提案してください。なお、pep8の命名規則を守ってください。

# Step4

```python
```
思考ログ：

