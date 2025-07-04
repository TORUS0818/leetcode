# Step1

かかった時間：15min

計算量：

文字列の長さをNとして
- 時間計算量：O(N)
- 空間計算量：O(N)

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        start = 0
        stop = 1
        substring_chars = set()
        longest_substring_len = 0
        while stop <= len(s):
            c = s[stop - 1]
            while c in substring_chars:
                substring_chars.remove(s[start])
                start += 1

            substring_chars.add(c)
            longest_substring_len = max(
                longest_substring_len,
                stop - start
            )
            stop += 1
            
        return longest_substring_len
```
思考ログ：
- 文字列を延ばしていって、現在の部分文字列に重複が出たら、重複が解消されるまで頭を進める、を文字列の終わりまで繰り返す

# Step2

講師役目線でのセルフツッコミポイント：
- ```while stop < len(s): stop += 1```は、for文に直そうという気持ちになってほしい

参考にした過去ログなど：
- https://github.com/ryosuketc/leetcode_arai60/pull/37
- https://github.com/Kaichi-Irie/leetcode-python/pull/9
  - 出現した最後のインデックスを覚えておく
- https://github.com/yakataN/Arai60/pull/3
- https://github.com/skypenguins/coding-practice/pull/3
- https://github.com/fuga-98/arai60/pull/47
- https://github.com/hroc135/leetcode/pull/45
- https://github.com/olsen-blue/Arai60/pull/49
  - https://github.com/olsen-blue/Arai60/pull/49#discussion_r2005295464
- https://github.com/Ryotaro25/leetcode_first60/pull/52
  - 文字種類の制限から全探索の計算量はもう少し絞れる  
    - https://github.com/Ryotaro25/leetcode_first60/pull/52#discussion_r1981068549
- https://github.com/katsukii/leetcode/pull/5
  - 出現回数を記録していってもいい
    - https://github.com/katsukii/leetcode/pull/5/files#r1896477073
- https://github.com/philip82148/leetcode-swejp/pull/3
  - スタック&ヒープについて
    - https://github.com/philip82148/leetcode-swejp/pull/3/files#r1853944201
- https://github.com/Yoshiki-Iwasa/Arai60/pull/42
- https://github.com/rihib/leetcode/pull/7
- https://github.com/rossy0213/leetcode/pull/23
  - https://github.com/rossy0213/leetcode/pull/23#discussion_r1696208116
- https://github.com/fhiyo/leetcode/pull/48
  - leftをexclusiveにするのも一つ
  - setのドキュメント
    - https://docs.python.org/3.12/library/stdtypes.html#set
- https://github.com/sakzk/leetcode/pull/3
  - https://github.com/sakzk/leetcode/pull/3/files#r1591194013
- https://github.com/Mike0121/LeetCode/pull/21
- https://github.com/Exzrgs/LeetCode/pull/2
- https://github.com/thonda28/leetcode/pull/6
- https://github.com/SuperHotDogCat/coding-interview/pull/3
- https://github.com/nittoco/leetcode/pull/3
  - ASCIIについて
    - https://discordapp.com/channels/1084280443945353267/1198621745565937764/1211683153668866139
- https://github.com/t0d4/leetcode/pull/2
- https://github.com/shining-ai/leetcode/pull/48
- https://github.com/hayashi-ay/leetcode/pull/47
- https://discord.com/channels/1084280443945353267/1198621745565937764/1211678975156420678
  - ASCIIの話
- https://github.com/usatie/leetcode/blob/main/Blind75/03.%20Longest%20Palindromic%20Substring/ans_01_20240126.cpp

全探索
```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        def is_repeating(s: str) -> bool:
            return len(set(s)) != len(s)

        s_len = len(s)
        longest_substring_len = 0
        for left in range(s_len):
            for right in range(left, s_len):
                sub_s = s[left:right + 1]
                if not is_repeating(sub_s):
                    longest_substring_len = max(
                        longest_substring_len,
                        len(sub_s)
                    )
        
        return longest_substring_len
```
思考ログ：
- 頭から抜けていた、良くない

出現回数管理
```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        letter_to_count = {c: 0 for c in s}
        
        left = 0
        max_len = 0
        for right in range(len(s)):
            while letter_to_count[s[right]] > 0:
                letter_to_count[s[left]] -= 1
                left += 1

            letter_to_count[s[right]] += 1
            max_len = max(max_len, right - left + 1)
        
        return max_len
```
思考ログ：
- これはこれで素直だなと感じた

文字が登場した最後のインデックスを使う
```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        left = 0
        letter_to_last_index = {}
        max_len = 0
        for right in range(len(s)):
            if s[right] in letter_to_last_index:
                left = max(left, letter_to_last_index[s[right]] + 1)
            
            max_len = max(max_len, right - left + 1)
            letter_to_last_index[s[right]] = right
        
        return max_len
```
思考ログ：
- ややテクニカルだが効率はいいなと感じた
- leftを更新する際に現在のleftより前に戻らないようにすることを注意しないといけない
  - ちゃんと脳内でシミュレート出来ていれば問題ないのかもしれないが、自分のシミュレータだとちょっと危なっかしいなと感じた

# Step3

かかった時間：4min

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        letter_to_last_index = {}
        left = 0
        max_len = 0
        for right in range(len(s)):
            if s[right] in letter_to_last_index:
                left = max(left, letter_to_last_index[s[right]] + 1)
            
            max_len = max(max_len, right - left + 1)
            letter_to_last_index[s[right]] = right
        
        return max_len
```
思考ログ：
- 情報の活用の仕方に学びがあったので、こちらの解き方にした
- step2から少し日が経って書いたが、ほぼ同じものが出てきて、改めてそういうものなんだなと感じた
  - 序盤の変数定義の並びが変わっているのは、この方法で解こうという思いが強かったからかもしれない

# Step4

```python
```
思考ログ：
