# Step1

かかった時間：15min

計算量：coins.length=N, amount=Mとして

時間計算量：O(N*M)

空間計算量：O(M)

```python
class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        amount_to_num_coins = [-1] * (amount + 1)
        amount_to_num_coins[0] = 0
        for i in range(1, amount + 1):
            for coin in coins:
                if i - coin < 0:
                    continue
                if amount_to_num_coins[i - coin] == -1:
                    continue
                if amount_to_num_coins[i] == -1:
                    amount_to_num_coins[i] = amount_to_num_coins[i - coin] + 1
                    continue
                amount_to_num_coins[i] = min(
                    amount_to_num_coins[i],
                    amount_to_num_coins[i - coin] + 1
                )

        return amount_to_num_coins[amount]
```
思考ログ：
- 方針としては、各amountを達成する最小コイン数を管理する配列を用意して、確認したいamountからcoinを引いたamountが最小何枚のコインで作れるかの情報を元に更新していく、という感じ
- 初期値として-1を入れておくと最後はシンプルに書けるが道中煩雑になる
  - 十分に大きい数（sys.maxsizeやmath.inf）を入れておくのも一つ  

再帰に直す
```python
class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        @cache
        def coin_change_helper(amount):
            if amount == 0:
                return 0
            if amount < 0:
                return -1

            min_num_coins = -1
            for coin in coins:
                num_coins = coin_change_helper(amount - coin)
                if num_coins == -1:
                    continue
                if min_num_coins == -1:
                    min_num_coins = num_coins + 1
                    continue
                min_num_coins = min(min_num_coins, num_coins + 1)
            
            return min_num_coins
        
        return coin_change_helper(amount)
```
思考ログ：
- 最悪、amount=10^4、coin=1みたいなケースで10^4回再帰呼び出しが起きるのでデフォルトの設定だと厳しい

BFS
```python
from collections import deque


class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        if amount == 0:
            return 0
            
        amount_and_num_coins = deque([(0, 0)])
        found = set()
        while amount_and_num_coins:
            current_amount, num_coins = amount_and_num_coins.popleft()
            if current_amount in found:
                continue
            found.add(current_amount)

            for coin in coins:
                next_amount = current_amount + coin
                next_num_coins = num_coins + 1
                if next_amount == amount:
                    return next_num_coins

                if next_amount > amount:
                    continue
                amount_and_num_coins.append((next_amount, next_num_coins))
            
        return -1
```
思考ログ：
- これまでに以下のような議論があったのを思い出す
  - 1つの変数に2つ以上の情報を持たせることについて
    - dataclassなどで新たにデータ構造を定義するのも一つ
  - 間引きをするのはqueueに入れる時にするか、出た時にするか
- L72,73の処理が気になるが、```amount_and_num_coins```に入れるタイミングで色々処理したい
  - L81の後ろにこの判定処理を持ってくればこの特別使いは不要だが、上の実装の方が効率は良さそう

# Step2

講師役目線でのセルフツッコミポイント：

参考にした過去ログなど：
- https://github.com/Ryotaro25/leetcode_first60/pull/44
- https://github.com/nittoco/leetcode/pull/38
- https://github.com/seal-azarashi/leetcode/pull/37
  - D/BFSの解法でスタックに詰める条件を気をつけないと指数オーダーになるという話
  - coinsをソートしてDFSという手があったか
- https://github.com/Yoshiki-Iwasa/Arai60/pull/54
- https://github.com/fhiyo/leetcode/pull/41
  - 初期化の話、今回はamount + 1と置くのもアリ、という話 
    - https://github.com/fhiyo/leetcode/pull/41/files#r1679440981
  - ハッシュじゃなくて配列を用意しておくのも一つ
    - https://github.com/fhiyo/leetcode/pull/41/files#r1679589925
  - どのタイミングでチェックしたり更新したりするかで結構効率が変わってくる
- https://github.com/goto-untrapped/Arai60/pull/34
- https://github.com/sakupan102/arai60-practice/pull/41
- https://github.com/ryoooooory/LeetCode/commit/c08cb0d8f5d7f1fd62c78e02fb6d7292499c8913#diff-8d9e9215dc393d847a16bf015fdf63b12d066253098cac2fbc8a2b07a6bbd9b6
- https://github.com/thonda28/leetcode/pull/1
- https://github.com/shining-ai/leetcode/pull/40
  - @cacheの話
    - https://github.com/shining-ai/leetcode/pull/40/files#r1550074736
    - 以前に同じ指摘を頂いたが、inner-functionであれば、毎回別のオブジェクトとして生成されるのでcacheの再利用はない、という話
- https://github.com/hayashi-ay/leetcode/pull/68

DFS（coinsソート版）
```python
class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        coins = sorted(coins)
        amount_and_num_coins = deque([(0, 0)])
        amount_to_num_coins = {}
        while amount_and_num_coins:
            current_amount, num_coins = amount_and_num_coins.pop()
            if not current_amount in amount_to_num_coins:
                amount_to_num_coins[current_amount] = num_coins
            else:
                if amount_to_num_coins[current_amount] <= num_coins:
                    continue
                amount_to_num_coins[current_amount] = num_coins

            for coin in coins:
                if current_amount + coin > amount:
                    break
                amount_and_num_coins.append((current_amount + coin, num_coins + 1))
        
        if not amount in amount_to_num_coins:
            return -1
        return amount_to_num_coins[amount]
```
思考ログ：
- 幾つか気づかなかった調整を加えた
  - coinsを昇順にする
  - L134のbreak（coinsの順序からcontinueではなくbreakできる）
  - 更新のタイミング
    - https://github.com/seal-azarashi/leetcode/pull/37/files#r1833635438  

# Step3

かかった時間：4min

```python
class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        UNREACHABLE = -1

        amount_to_num_coins = [-1] * (amount + 1)
        amount_to_num_coins[0] = 0
        for current_amount in range(1, amount + 1):
            for coin in coins:
                prev_amount = current_amount - coin
                if prev_amount < 0:
                    continue
                if amount_to_num_coins[prev_amount] == UNREACHABLE:
                    continue
                if amount_to_num_coins[current_amount] == UNREACHABLE:
                    amount_to_num_coins[current_amount] = amount_to_num_coins[prev_amount] + 1
                    continue
                amount_to_num_coins[current_amount] = min(
                    amount_to_num_coins[current_amount],
                    amount_to_num_coins[prev_amount] + 1
                )
        return amount_to_num_coins[amount]
```
思考ログ：
- DPで、STEP1からいくつか軽微な修正を加えた
  - ループ変数```i```を```current_amount```へ
    - 正直これもあんまりしっくり来ないが、```amount```が使われてしまっているのが痛い、、
  - マジックナンバーの解消（UNREACHABLE）
  - ```prev_amount = current_amount - coin```の定義
    - これは右辺のままの方が分かりやすい気もするので無理に変数に置かなくていいかもしれない
- ```current_amount + coin```のように確認していってもいい
  - その場合は```current_amount```は0から舐めるようにする

# Step4

```python
```
思考ログ：
