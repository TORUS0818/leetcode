# Step1

かかった時間：6min

計算量：

時間計算量：O(logn)

空間計算量：O(logn)

```python
class Solution:
    @cache
    def myPow(self, x: float, n: int) -> float:
        if n == 0:
            return 1
        if n < 0:
            return self.myPow(1 / x, -n)
        
        if n % 2 == 0:
            return self.myPow(x, n // 2) * self.myPow(x, n // 2)
        else:
            return x * self.myPow(x, n - 1)
```
思考ログ：
- ```n < 0```の場合は```1 / x```を考えることで、```n >= 0```の範囲で考えられるようにする
- あとはn回掛け算すれば良い
  - ```x * x * x * ... * x```
- 効率化を考える
  - ```x^(2k) = x^k * x^k```なので、logn回計算すればいい
  - nが奇数の時は、```x * x^(n - 1)```と分解して、上の計算を適用すればいい

# Step2

講師役目線でのセルフツッコミポイント：
- ```x = 0```の考慮がない

参考にした過去ログなど：
- https://github.com/hroc135/leetcode/pull/43
  - math.Powのソースコード、特殊ケースについて
  - https://github.com/hroc135/leetcode/pull/43/files#r2002298814
- https://github.com/olsen-blue/Arai60/pull/45
- https://github.com/potatokun1999/leetcode/pull/2
- https://github.com/SanakoMeine/leetcode/pull/9
  - f-stringの[コレ](https://docs.python.org/3/whatsnew/3.8.html#f-strings-support-for-self-documenting-expressions-and-debugging)知らなかった
- https://github.com/Ryotaro25/leetcode_first60/pull/48
  - float/doubleの内部実装について
    - https://github.com/Ryotaro25/leetcode_first60/pull/48/files#r1978642684
- https://github.com/Yoshiki-Iwasa/Arai60/pull/38
- https://github.com/fhiyo/leetcode/pull/46
  - ここら辺、気が回ってない
    > C++だと n = -2^31 のときに符号を逆にするとsigned intではオーバーフローする
  - https://stackoverflow.com/questions/13591970/does-python-optimize-tail-recursion
    > A tail call can never be optimized to a jump in Python. An optimization is a program transformation that preserves the program's meaning.
    > Tail-call elimination doesn't preserve the meaning of Python programs.
    - pythonだと明示的にはTCO（テールコールの除去）を行わない
  - powのcpython実装
    - https://github.com/python/cpython/blob/bdab67e1c795443a0d8f8a5bbeb3a91ac4fd5a19/Objects/longobject.c#L4894
    - メインの処理はここら辺か
      ```
      for (--i, bit >>= 1;;) {
          for (; bit != 0; bit >>= 1) {
              MULT(z, z, z);
              if (bi & bit) {
                  MULT(z, a, z);
              }
          }
          if (--i < 0) {
              break;
          }
          bi = b->long_value.ob_digit[i];
          bit = (digit)1 << (PyLong_SHIFT-1);
      }
      ```
    - 真面目にエラーハンドリングするのは結構大変
- https://github.com/yukik8/leetcode/commit/c3ac5e7e038c3da2c4b5c3a6ad24abcb48efe4e9
- https://github.com/goto-untrapped/Arai60/pull/29
  - javaも末尾再帰最適化が大変らしい
  - https://github.com/goto-untrapped/Arai60/pull/29/files#r1646519560
- https://github.com/nittoco/leetcode/pull/17
  - 再帰は（この問題の解法としては）個人的にわかりやすいと感じたが、そうでもないらしい
    - https://github.com/nittoco/leetcode/pull/17/files#r1632224692
  - 上位ビットから舐めていく方法
    - n.bit_length()知らなかった
- https://github.com/Exzrgs/LeetCode/pull/28/files
- https://github.com/Mike0121/LeetCode/pull/17
- https://github.com/SuperHotDogCat/coding-interview/pull/15
- https://github.com/shining-ai/leetcode/pull/45
- https://github.com/hayashi-ay/leetcode/pull/41
- https://discord.com/channels/1084280443945353267/1210494002277908491/1210521583605776414

loop-1 (TLE)
```python
class Solution:
    def myPow(self, x: float, n: int) -> float:
        assert x != 0 or n > 0
        
        if x == 0:
            return 0
        if n == 0:
            return 1
        if n < 0:
            x = 1 / x
            n *= -1
        
        result = 1
        for _ in range(n):
            result *= x
        
        return result
```
思考ログ：
- 何も工夫しないloop

loop-1
```python
class Solution:
    def myPow(self, x: float, n: int) -> float:
        assert x != 0 or n > 0

        if n == 0:
            return 1
        if n < 0:
            x = 1.0 / x
            n *= -1
        
        powered_x = 1
        bit_mask = 1 << n.bit_length() - 1
        while bit_mask:
            powered_x *= powered_x
            if n & bit_mask:
                powered_x *= x
            
            bit_mask >>= 1
            
        return powered_x
```
思考ログ：
- 再帰と同等の処理をloopに変換するのに時間がかかってしまった。。
  - https://github.com/hroc135/leetcode/pull/43/files#r2002294824
    - 思考のプロセスにどこか不自然な部分があるのだろうか。。
- 理解を深めるため、少し整理をしておく
  - まず再帰の考え方をおさらいすると、
    - 基本的なアイデアは、n乗をn/2乗2つに分けて計算して、マージ（積を取る）する、というもの
    - ただし、nが奇数の場合は、n-1して偶数に直したうえで計算、その分1回余分にxをかける
  - これをloopにすると、
    - 基本は、x -> x^2 -> x^4...と累乗を計算していけばいいが、割った数が奇数の場合の処理を入れないといけない
    - どのタイミングで余分にxをかける必要があるかは、上位ビットから見ていけば分かる

loop-2
```python
class Solution:
    def myPow(self, x: float, n: int) -> float:
        assert x != 0 or n > 0

        if n == 0:
            return 1
        if n < 0:
            x = 1.0 / x
            n *= -1
        
        powered_x = 1.0
        while n:
            if n & 1:
                powered_x *= x
            
            x *= x
            n >>= 1
        
        return powered_x
```
思考ログ：
- a^n = k0 * a^(2^0) * k1 * a^(2^1) * k2 * a^(2^2) * ... * ks * a^(2^s)と分解して、(k0, k1, ..., ks)がnの２進数表記と対応している、と捉えた方が、loop-1より自分はピンとくる

# Step3

かかった時間：3min

```python
class Solution:
    def myPow(self, x: float, n: int) -> float:
        if n == 0:
            return 1
        
        assert x != 0 or n > 0
        if n < 0:
            x = 1.0 / x
            n *= -1
        
        if n % 2:
            return x * self.myPow(x ** 2, n // 2)
        else:
            return self.myPow(x ** 2, n // 2)
```
思考ログ：
- 考え方としては、自分は再帰の方が分かりやすかったのでこちらを採用することにした
- 変更点は以下
  - 浮動小数を表すように.0表記に
  - myPowの仕様として```x != 0 or n > 0```のチェックを導入、ただ本来は分岐処理でちゃんと対応して値を返してあげるのが良さそう
    - それか放っておいて```ZeroDivisionError```が返ってきた方が良いのか、、？
  - 引数の方で2乗の調整（step1では、```self.myPow(x ** 2, n // 2) * self.myPow(x ** 2, n // 2)```としていた）
  - ```n < 0```の場合、以降の処理を```n > 0```として処理出来るよう変換処理を入れた
- その他、今回はpythonの実装だったので問題にならなかったが、intの-2^31を2^31に直す際に生じる問題やfloat/doubleの内部実装（IEEE754）、tail-callの諸事情など色々勉強になる課題だった

# Step4

```python
```
思考ログ：
