# Step1

かかった時間：時間切れ

計算量：
作成するペアの数k

時間計算量：O(klogk)

空間計算量：O(k)

```python
import heapq


class Solution:
    def kSmallestPairs(self, nums1: List[int], nums2: List[int], k: int) -> List[List[int]]:
        candidates = [(nums1[0] + nums2[0], 0, 0)]
        k_pairs_with_smallest_sums = []
        seen = {(0, 0)}
        while len(k_pairs_with_smallest_sums) < k:
            _, u, v = heapq.heappop(candidates)
            k_pairs_with_smallest_sums.append([nums1[u], nums2[v]])
            for d_u, d_v in [[1, 0], [0, 1]]:
                if 0 <= u + d_u < len(nums1) and 0 <= v + d_v < len(nums2) and (u + d_u, v + d_v) not in seen:
                    heapq.heappush(candidates, (nums1[u + d_u] + nums2[v + d_v], u + d_u, v + d_v))
                    seen.add((u + d_u, v + d_v))
            
        return k_pairs_with_smallest_sums
```
思考ログ：
- 最初```(num1 + num2, num1, num2)```のタプルをヒープに詰めていってk個を超えたらpopする作戦で実装
  - 条件を見落としていてTLE、せっかくソートされてるんだし、これ全部探索したらダメなやつ
  - nums1とnums2の次の要素を比較して、小さい方を見ていけば良い？？ん？？
    - 時間切れ
- discord上を漁って解法を斜め読みして一旦上を実装
- 空間計算量について、少し迷った
  - ヒープの最初の要素は1、取り出して最小値を確定させて、ここに候補が2つ加わる
  - ヒープの要素は2、取り出して最小値を確定させて、ここに候補が2つ加わる
  - ヒープの要素は3、取り出して最小値を確定させて、ここに候補が2つ加わる...
  - 端っこにいくと追加する候補は1つだが、端っこに行かないケースでもヒープの大きさは高々Kで抑えられる

# Step2

講師役目線でのセルフツッコミポイント：
- 適度に関数化すること
- ロジック面でもう少し無駄を無くす工夫はできそう

参考にした過去ログなど：
- https://github.com/sakupan102/arai60-practice/pull/11/files
  - https://discord.com/channels/1084280443945353267/1200089668901937312/1222573940610695341
    - 無駄の省き方について
      - (x, y)より(x-1, y),(x, y-1)の方が小さいので、この二つが答えに入ってないうちは、(x, y)を候補に追加しないでおくと効率がいい
      - 言われてみればそうだが、、これ、その場で自力で思いつく感じがしない。。
      - setを使わないで実装できるか
- https://github.com/YukiMichishita/LeetCode/pull/4
- https://github.com/hayashi-ay/leetcode/pull/66
- https://docs.google.com/document/d/1dbBktdFVrwAgccaosslO_VQg3yugbfT8WCTuRCqyYDw/edit

```python
import heapq


class Solution:
    def kSmallestPairs(
        self, nums1: List[int], nums2: List[int], k: int
    ) -> List[List[int]]:
        def _is_addible(index1, index2):
            if index1 >= len(nums1) or index2 >= len(nums2):
                return False
            if index1 == 0 or index2 == 0:
                return True
            return (index1 - 1, index2) in added and (index1, index2 - 1) in added

        candidates = [(nums1[0] + nums2[0], 0, 0)]
        k_smallest_pairs = []
        added = set()
        while len(k_smallest_pairs) < k:
            _, index1, index2 = heapq.heappop(candidates)
            k_smallest_pairs.append([nums1[index1], nums2[index2]])
            added.add((index1, index2))

            for delta1, delta2 in [[0, 1], [1, 0]]:
                new_index1, new_index2 = index1 + delta1, index2 + delta2
                if _is_addible(new_index1, new_index2):
                    heapq.heappush(
                        candidates,
                        (nums1[new_index1] + nums2[new_index2], new_index1, new_index2),
                    )
        return k_smallest_pairs
```
思考ログ：
- 他の人の実装を参考にして、一部関数化と候補として入れるものの剪定を厳しくしてみた
- 候補の剪定条件が腹落ちするのに時間がかかった、こういうのが苦手なのかも、、
  - 二次元の表で考えて左上がより小さいので右か下を候補に入れていけばいい（これはそうだねという感じ）
  - 自分の左か上がpairsに入ってない時は新しく探索候補に加えなくていい
    - (まだkに達していないなら)その2つが必ず先にpairsに入ることになるから
  - どっちかが0の時は候補に入れる
    - 上の条件がメインだが、端はこの条件で判定できない（片方がないから）
    - そこら辺を考えれば思いつくはず
- 上記の剪定が思いつかなくても、少なくとも最小値の組み合わせを中心に探索していくアイデアは思いつきたい
- あと、```is_addible```について、こういう書き方が出来るのは分かっているのだけど、関数の外の変数（nums1, nums2, added）を使っているのが若干気になる
  - 冗長だけど、引数に持たせたくなってしまうのだが、どうなんだろう、、

入れたものを管理しないで済ます方法（setを使わない方法）
```python
import heapq


class Solution:
    def kSmallestPairs(self, nums1: List[int], nums2: List[int], k: int) -> List[List[int]]:
        candidates = [(nums1[0] + nums2[0], 0, 0)]
        k_smallest_pairs = []
        while len(k_smallest_pairs) < k:
            _, index1, index2 = heapq.heappop(candidates)
            k_smallest_pairs.append([nums1[index1], nums2[index2]])
            if index1 == 0 and index2 + 1 < len(nums2):
                heapq.heappush(
                    candidates, 
                    (nums1[index1] + nums2[index2 + 1], index1, index2 + 1)
                )
            if index1 + 1 < len(nums1):
                heapq.heappush(
                    candidates, 
                    (nums1[index1 + 1] + nums2[index2], index1 + 1, index2)
                )
        return k_smallest_pairs
```
思考ログ：
- こちらの方がシンプルではあるが、より思いつく自信がない
- やってることは、、
  - ind1 * ind2の表を作って、(0, 0)からスタートし、ひたすら下に候補を探していくというもの
  - ただ最初の行とind1が0の時だけは右も追加する
  - 次のような場合は大丈夫だろうか？
    - 上でやった右と下を順次入れていく方法と比較すると、この方法では（最初の行とind1が0の時を除いて）下の候補しか追加しない
    - では右の候補が次の最小値になる可能性はないのだろうか、、
      - -> 大丈夫
      - もしそうならその上が既に答えに入っているはず
      - これを繰り返すと最初の行まで到達する（つまりもしそのようなシチュエーションなら右の候補は既に候補に入っているはず）
- やはり頭の使い方が下手な気がする、もう少しシンプルに考えられないだろうか、、

# Step3

かかった時間：8min

```python
import heapq


class Solution:
    def kSmallestPairs(self, nums1: List[int], nums2: List[int], k: int) -> List[List[int]]:
        def _is_addible(index1: int, index2: int) -> list:
            if index1 >= len(nums1) or index2 >= len(nums2):
                return False
            if index1 == 0 or index2 == 0:
                return True
            return (index1 - 1, index2) in added and (index1, index2 - 1) in added

        candidates = [(nums1[0] + nums2[0], 0, 0)]
        added = set()
        k_smallest_pairs = []
        while len(k_smallest_pairs) < k:
            _, index1, index2 = heapq.heappop(candidates)
            k_smallest_pairs.append([nums1[index1], nums2[index2]])
            added.add((index1, index2))

            if _is_addible(index1 + 1, index2):
                heapq.heappush(
                    candidates,
                    (nums1[index1 + 1] + nums2[index2], index1 + 1, index2)
                )
            if _is_addible(index1, index2 + 1):
                heapq.heappush(
                    candidates,
                    (nums1[index1] + nums2[index2 + 1], index1, index2 + 1)
                )
        return k_smallest_pairs
```

思考ログ：
- 少しボリュームがあったため、タイポとかケアレスミスが多かった印象
- ただ全体の構成は頭に入ったので、同じものを再現するのは問題なかった

# Step4

かかった時間：10min

https://github.com/TORUS0818/leetcode/pull/12#discussion_r1623243567
を受けて
```python
import heapq


class Solution:
    def kSmallestPairs(self, nums1: List[int], nums2: List[int], k: int) -> List[List[int]]:
        # 9:59 >> 10:10
        def _is_addible(index1: int, index2: int) -> bool:
            if index1 >= len(nums1) or index2 >= len(nums2):
                return False
            if index1 == 0 or index2 == 0:
                return True
            return (index1 - 1, index2) in added and (index1, index2 - 1) in added

        def _add_to_candidates_if_necessary(index1: int, index2: int) -> None:
            if _is_addible(index1, index2):
                heapq.heappush(
                    candidates,
                    (nums1[index1] + nums2[index2], index1, index2)
                )

        k_smallest_pairs = []
        added = set()
        candidates = [(nums1[0] + nums2[0], 0, 0)]
        while len(k_smallest_pairs) < k:
            _, index1, index2 = heapq.heappop(candidates)
            k_smallest_pairs.append([nums1[index1], nums2[index2]])
            added.add((index1, index2))
            _add_to_candidates_if_necessary(index1 + 1, index2)
            _add_to_candidates_if_necessary(index1, index2 + 1)
        
        return k_smallest_pairs
```
思考ログ：

# Step5

かかった時間：16min

- odaさんからの提案
https://github.com/TORUS0818/leetcode/pull/12#discussion_r1623146530
- liquo-riceさんからのヒント（解答）
https://github.com/TORUS0818/leetcode/pull/12#discussion_r1623354548

上記を受けて再実装
```python
import heapq


class Solution:
    def kSmallestPairs(self, nums1: List[int], nums2: List[int], k: int) -> List[List[int]]:
        k_smallest_pairs = []
        candidates = [(nums1[0] + nums2[0], 0, 0)]
        next_index1 = [0] * len(nums2)
        next_index2 = [0] * len(nums1)
        while len(k_smallest_pairs) < k:
            _, index1, index2 = heapq.heappop(candidates)
            k_smallest_pairs.append([nums1[index1], nums2[index2]])
            next_index1[index2] += 1
            next_index2[index1] += 1
            if index1 + 1 < len(nums1) and index2 == next_index2[index1 + 1]:
                heapq.heappush(
                    candidates, 
                    (nums1[index1 + 1] + nums2[index2], index1 + 1, index2)
                )
            if index2 + 1 < len(nums2) and index1 == next_index1[index2 + 1]:
                heapq.heappush(
                    candidates, 
                    (nums1[index1] + nums2[index2 + 1], index1, index2 + 1)
                )
        return k_smallest_pairs
```
思考ログ：
- 理解はできたが、頭の使い方に混乱する、、
    - といいつつ、全部やってることは同じ（如何に無駄なく右下を候補に加えるか）なのでそこさえ分かれば 、、
    - シンプルに書けるし、setで管理する必要もなくなるので良いかなと思いつつ
    - ぱっと見で分かりやすい処理なのかと言われると？
        - 自分が未熟が故そう感じるのか？界隈では割と常識的な手続きなのだろうか
- 簡単にまとめる
    - 基本のアイデアは今までと一緒、最小のペアを取り出してその右か下を候補に追加したい
    - 以下、右を候補に追加できるか、の例に限定してシミュレーションする（*は答えに追加済み、@が現在地、+が次候補とする）
        - パターン１
          ```
          * * * *
          * * * 
          * * @ +
          * * * 
          ``` 
        - パターン2 <br>
          ```
          * * * *
          * * * *
          * * @ *(+)
          * * * *
          ```
        - パターン3
          ```
          * * * *
          * * * *
          * * @ +
          * * *
          ```
        - パターン1では候補に追加はできるものの、それが選ばれる前に答えに追加されるべき箇所が存在する（+の真上）
        - パターン2ではもう既に答えに追加されているので、また追加したら重複が生じてしまう
        - パターン3が候補として入れるべき状態
  - 以上をどう検知するかという話
      - 行、列ごとに答えとして何ペア入れたかの実績を記録しておく
        ```
        * * * * 4
        * * * * 4
        * * @ + 4
        * * *   3
        4 4 4 2
        ```
      - +の位置が、今回の例なら上から2+1=3番目であれば候補に入れられる
          - 何故か
              - 原則を思い出すと、自分の左と上は必ず先に答えにいる、というものがあった
              - +から見て左は@でもう答えに移動しているからOK
              - +の上が問題だが、履歴を確認すると、この列は+の現在地3行目の一個手前の行まで答えに入っていることがわかるからOK

# Step6

```python
```
思考ログ：
