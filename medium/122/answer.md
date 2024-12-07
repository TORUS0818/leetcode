# Step1

かかった時間：5min

計算量：prices.length=Nとして

時間計算量：O(N)

空間計算量：O(1)

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        prev_price = prices[0]
        max_profit = 0
        for price in prices:
            if prev_price < price:
                max_profit += (price - prev_price)
            prev_price = price
        
        return max_profit
```
思考ログ：
- pricesのi番目の要素とi+1番目の要素の関係に注目する
  - prices[i] < prices[i + 1]: prices[i]で買ってprices[i+1]で売る
  - prices[i] >= prices[i + 1 ]: 何もしない
- i番目で買ってi+k番目で売るのは、buy(i) & sell(i + 1) & buy(i + 1) & sell(i + 2) & buy(i + 2)... & buy(i + k - 1) & sell(i + k)と同じなので分解して一つずつ考える

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        return sum([
            prices[i] - prices[i - 1] for i in range(1, len(prices)) 
            if prices[i] - prices[i - 1] > 0
        ])
```
思考ログ：
- やっていることは同じ、差分をとってプラスのものだけ足して返す
- 簡潔に書けるが、一度```price_diff```のような変数を介してあげた方が意図は取りやすいと思う（あと同じ計算している）

# Step2

講師役目線でのセルフツッコミポイント：

参考にした過去ログなど：
- https://github.com/Ryotaro25/leetcode_first60/pull/42
- https://github.com/seal-azarashi/leetcode/pull/36
- https://github.com/goto-untrapped/Arai60/pull/59
  - top-down DP
    - これが一番理解に時間がかかった
- https://github.com/Yoshiki-Iwasa/Arai60/pull/53
  - 追加課題：最大利益を実現できる最小の売買回数は？  
- https://github.com/fhiyo/leetcode/pull/39
- https://github.com/sakupan102/arai60-practice/pull/39
- https://github.com/shining-ai/leetcode/pull/38
- https://github.com/hayashi-ay/leetcode/pull/56
  - 2ndの解法の関数の切り分けが勉強になった（同様のロジックを書く際に自分でもこのように分けて整理できるか）
- https://discord.com/channels/1084280443945353267/1210494002277908491/1210521728699076628

DP
```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        # pl: profit & lossの略語
        max_pl_hold_last = -prices[0]
        max_pl_no_position_last = 0
        for i in range(1, len(prices)):
            max_pl_hold = max(
                max_pl_hold_last, 
                max_pl_no_position_last - prices[i] # 購入
            )
            max_pl_no_position = max(
                max_pl_hold_last + prices[i], # 売却
                max_pl_no_position_last
            )
            max_pl_hold_last = max_pl_hold
            max_pl_no_position_last = max_pl_no_position
        
        return max_pl_no_position_last
```
思考ログ：
- 他の人の解答を漁っていて見つけた、株を持っている状態かどうかで場合わけ
  - 最初よく分からなかったので自分なりに書いてみた
  - 株を持っている場合は以下の大きい方を採用
    - 前回株を持っている状態での最大利得をそのまま引き継ぐ（今回は何もしない）
    - 前回株を持っていない状態での最大利得に今回株を新規購入した分のお金を引いたもの
  - 株を持っていない場合は以下の大きい方を採用
    - 前回株を持っている状態での最大利得に今回株を決済した分のお金を追加したもの
    - 前回株を持っていない状態での最大利得をそのまま引き継ぐ（今回は何もしない）
- あんまり専門用語を使うのは憚られるが、そういう現場にいるという体で```no_position```は```square```とかでも良かったかも

素直にvalley-to-peakをとっていく方法
```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        def find_next_valley(start: int) -> int:
            i = start
            while i < len(prices) - 1:
                if prices[i] >= prices[i + 1]:
                    i += 1
                else:
                    return i
            return i
        
        def find_next_peak(start: int) -> int:
            i = start
            while i < len(prices) - 1:
                if prices[i] <= prices[i + 1]:
                    i += 1
                else:
                    return i
            return i

        i = 0
        max_profit = 0
        while i < len(prices):
            valley_i = find_next_valley(i)
            buy_price = prices[valley_i]
            peak_i = find_next_peak(valley_i)
            sell_price = prices[peak_i]
            max_profit += sell_price - buy_price
            i = peak_i + 1
        
        return max_profit
```
思考ログ：
- 以前はこういう処理を関数に分けずに書き連ねてバグらせる、というのがよくあった
- ```valley_i```とか```peak_i```のように名前を変えずに```i```を更新していく感じでもいいのだろうか
- ```find_next_valley/peak```は、```find_next_valley/peak_index```のように目的語を入れた方がいい？自明か？
- ```find_next_valley/peak```は、else以下をbreakしてreturnをまとめるでも良かったか
- 同値や端っこの処理はちゃんと考えないと（選択しないと）いけない
  - ```if prices[i] <= prices[i + 1]:```とか```=```を入れるかどうか、とか
- [追加課題](https://github.com/Yoshiki-Iwasa/Arai60/pull/53/files#r1730194725)は、この実装で自然に答えられる

再帰（top-down）
```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        def max_profit_helper(day: int, price_tomorrow: int, max_profit: int) -> int:
            if day < 0:
                return max_profit
            
            price_today = prices[day]
            max_profit += max(0, price_tomorrow - price_today)

            return max_profit_helper(day - 1, price_today, max_profit)
        
        return max_profit_helper(len(prices) - 2, prices[len(prices) - 1], 0)
```
思考ログ：
- やっていることが分かりやすくなるよう、少し持ち物を冗長にした（本当はdayとmax_profitだけでいい）
  - 明日から来た自分から今日の日付と明日の株価と明日以降達成できる最大収益を教えてもらう

# Step3

かかった時間：2min

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        prev_price = prices[0]
        max_profit = 0
        for price in prices:
            max_profit += max(0, price - prev_price)
            prev_price = price
        
        return max_profit
```
思考ログ：
- DPも面白いけど、こちらの方が自然な感じはする

# Step4

```python
```
思考ログ：
