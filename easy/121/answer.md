# Step1

かかった時間：7min

計算量：prices.length=Nとして

時間計算量：O(N)

空間計算量：O(1)

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        min_price_so_far = prices[0]
        max_profit = 0
        for price in prices:
            profit = price - min_price_so_far
            if max_profit < profit:
                max_profit = profit
                continue
            min_price_so_far = min(min_price_so_far, price)
        
        return max_profit
```
思考ログ：
- 2回舐める方法をまず思いついたが、今まで見た最小の価格を覚えておけば良さそうと思い、上のように実装した
- 最大利得の更新が起きた際は今まで見た最小の価格の更新は起きないのでcontinueするようにした

再帰
```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        def _max_profit_helper(max_profit: int, min_price_so_far: int, i: int) -> int:
            if len(prices) == i:
                return max_profit
                
            profit = prices[i] - min_price_so_far
            if max_profit < profit:
                max_profit = profit
            min_price_so_far = min(min_price_so_far, prices[i])
            
            return _max_profit_helper(max_profit, min_price_so_far, i + 1)
        
        return _max_profit_helper(0, prices[0], 1)
```
思考ログ：
- 前の人からは”今までで最安だった価格”と”今までに達成することのできた最大利益”について教えて貰えばいい
- ```max_profit = max(max_profit, prices[i] - min_price_so_far)```こうしてもいい
- ```prices```の長さが最大10^5なので、デフォルト設定だと厳しい

2重ループ（TLE）
```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        max_profit = 0
        for sell_i in range(1, len(prices)):
            for buy_i in range(sell_i):
                max_profit = max(max_profit, prices[sell_i] - prices[buy_i])
        
        return max_profit
```
思考ログ：
- 売りor買いで選択するインデックスの意味が乗るようにbuy/sell_iとしてみたがどうだろうか

# Step2

講師役目線でのセルフツッコミポイント：

参考にした過去ログなど：
- https://github.com/Ryotaro25/leetcode_first60/pull/41
- https://github.com/seal-azarashi/leetcode/pull/35
- https://github.com/goto-untrapped/Arai60/pull/58
  - https://github.com/goto-untrapped/Arai60/pull/58/files#r1782742318
    > この見え方は大事だと思います。関数型っぽい感覚です。
    > scanl という考え方があります。
  - iを起点にそれより前の最小値とそれより後の最大値を比べる方法(実装してみよう)
- https://github.com/Yoshiki-Iwasa/Arai60/pull/52
- https://github.com/fhiyo/leetcode/pull/38
- https://github.com/erutako/leetcode/pull/3
- https://github.com/sakupan102/arai60-practice/pull/38
- https://github.com/kzhra/Grind41/pull/4
- https://github.com/shining-ai/leetcode/pull/37
- https://github.com/hayashi-ay/leetcode/pull/52
- https://github.com/colorbox/leetcode/pull/6
- https://github.com/Kitaken0107/GrindEasy/pull/7

0 ~ iまでの最小とi ~ len(prices)の最大を記録していく方法
```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        min_price_from_left = [prices[0]] * len(prices)
        for i in range(1, len(prices)):
            if min_price_from_left[i - 1] > prices[i]:
                min_price_from_left[i] = prices[i]
            else:
                min_price_from_left[i] = min_price_from_left[i - 1]
        
        max_price_from_right = [prices[-1]] * len(prices)
        for i in reversed(range(len(prices) - 1)):
            if prices[i] > max_price_from_right[i + 1]:
                max_price_from_right[i] = prices[i]
            else:
                max_price_from_right[i] = max_price_from_right[i + 1]
        
        max_profit = 0
        for min_price, max_price in zip(min_price_from_left, max_price_from_right):
            max_profit = max(max_profit, max_price - min_price)
        
        return max_profit
```
思考ログ：
- 以下の前処理をする
  - pricesを左から舐めてインデックスまでの最小値（cummin）を記録
  - pricesを右から舐めてインデックスまでの最大値（cummax）を記録
- それぞれをの同インデックスの値を比較して最大収益を出す
- zipについてドキュメントを見てみる
  - https://docs.python.org/ja/3/library/functions.html#zip
    > zip() は遅延評価です  
    > デフォルトでは、 zip() は最も短いイテラブルが消費しきった時点で停止します
    > zip() は、しばしば受け取ったイテラブルが全て同じ長さであるという想定の下で使われます。そのような場合、 strict=True オプションの利用が推奨されます。
    > あるイテラブルが他のイテラブルよりも先に消費しきった場合に ValueError 例外を送出します  
    > 短いイテラブルを一定の値でパディングして全てのイテラブルが同じ長さになるようにすることもできます。この機能は itertools.zip_longest() で提供されます。  
    > エッジケース: 引数としてイテラブルをひとつだけ渡した場合、 zip() は 1 タプルのイテレータを返します。引数なしの場合は空のイテレータを返します。

逆から見ていく（最大価格を保持）
```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        max_price = prices[-1]
        max_profit = 0
        for price in reversed(prices):
            max_price = max(max_price, price)
            max_profit = max(max_profit, max_price - price)
        
        return max_profit
```
思考ログ：

# Step3

かかった時間：1min

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        min_price = prices[0]
        max_profit = 0
        for price in prices:
            min_price = min(min_price, price)
            max_profit = max(max_profit, price - min_price)
        
        return max_profit
```
思考ログ：
- これが一番素直な気がした

# Step4

```python
```
思考ログ：

