# Step1

かかった時間：11min

計算量：
emails.length=N, emails[i].length=M, uniqueなemailの数をLとすると

時間計算量：O(NM)

空間計算量：O(L)

```python
class Solution:
    def numUniqueEmails(self, emails: List[str]) -> int:
        unique_email_addresses = set()
        for email in emails:
            local = email.split('@')[0]
            domain = email.split('@')[1]
            if local.find('+') > 0:
                local = local[:local.find('+')]
            local = local.replace('.', '')
            unique_email_addresses.add(local + '@' + domain)

        return len(unique_email_addresses)
```
思考ログ：
- 正規表現を真っ先に候補に考えたが、とりあえず```split```と```find```で文字列処理をしてパターン数を```set```を使って確認する方針で実装
- strのリファレンスを確認しよう
  - https://docs.python.org/ja/3/library/stdtypes.html#text-sequence-type-str

  > str.index(sub[, start[, end]])  
  > find() と同様ですが、部分文字列が見つからなかったとき ValueError を送出します。
  
  > str.partition(sep)  
  > 文字列を sep の最初の出現位置で区切り、 3 要素のタプルを返します。タプルの内容は、区切りの前の部分、区切り文字列そのもの、そして区切りの後ろの部分です。
  
  > str.rfind(sub[, start[, end]])  
  > 文字列中の領域 s[start:end] に sub が含まれる場合、その最大のインデックスを返します。
  
  > str.translate(table)
  > 与えられた変換テーブルに基づいて文字列を構成する各文字をマッピングし、マッピング後の文字列のコピーを返します。  
  > 文字から文字への異なる形式のマッピングから変換マップを作成するために、 str.maketrans() が使えます。
  
  > str.split(sep=None, maxsplit=-1)  
  > maxsplit が与えられていれば、最大で maxsplit 回分割されます。

# Step2

講師役目線でのセルフツッコミポイント：
- strのドキュメント

参考にした過去ログなど：
- https://github.com/fhiyo/leetcode/pull/17
  - '@'が複数含まれている場合を考慮する
    - https://en.wikipedia.org/wiki/Email_address
    - RFC1034
      - https://www.ietf.org/rfc/rfc1034.txt#:~:text=The%20labels%20must%20follow%20the%20rules%20for%20ARPANET%20host%20names.%20%20They%20must%0Astart%20with%20a%20letter%2C%20end%20with%20a%20letter%20or%20digit%2C%20and%20have%20as%20interior%0Acharacters%20only%20letters%2C%20digits%2C%20and%20hyphen.%20%20There%20are%20also%20some%0Arestrictions%20on%20the%20length.%20%20Labels%20must%20be%2063%20characters%20or%20less
    - RFC5322
      - https://datatracker.ietf.org/doc/html/rfc5322#:~:text=local-part%20%20%20%20%20%20%3D%20%20%20dot-atom%20/%20quoted-string%20/%20obs-local-part
    
  > 入力が不正で@が含まれない場合、 `local_name, domain_name = email.rsplit('@', maxsplit=1)` の行でunpackできずにエラーになる。まあいいんじゃないだろうか
  - 現状の自分の実装だとここで検知できないことに気づく
  - state（ローカルパートなのか、さらに+以降の部分なのか）を管理して一文字ずつ処理していく方法もある
  - あと正規表現、ただ目に優しくない
- https://github.com/sakupan102/arai60-practice/pull/15
  - （上でも書いてあったが）文字列再構成の話
    - https://google.github.io/styleguide/pyguide.html#310-strings
      > Avoid using the + and += operators to accumulate a string within a loop. In some conditions, accumulating a string with addition can lead to quadratic rather than linear running time. Although common accumulations of this sort may be optimized on CPython, that is an implementation detail. The conditions under which an optimization applies are not easy to predict and may change. Instead, add each substring to a list and ''.join the list after the loop terminates, or write each substring to an io.StringIO buffer. These techniques consistently have amortized-linear run-time complexity.
    - Cpython上で最適化がかかるかもしれないが、その条件を特定したり、今後の変更に配慮しないといけないの大変ですよね、という話
- https://github.com/tshimosake/arai60/pull/2
  - "@"が複数紛れていた場合の挙動について、選択肢
    > Exception を投げる。  
    > プログラムが停止する。  
    > エラーを示す値を返す。  
- https://github.com/hayashi-ay/leetcode/pull/25
- 色々過去ログを読んでみて、問題について気にする範囲がまだまだ狭いなと反省
  - 問題の制約を取っ払ってみる、とか、少し設定をいじってみてどう、とか
  - これもACすることに執着してしまっている弊害なのだろう、、意識して矯正する

まず、"@"問題に対処しつつStep1を書き換える
```python
class Solution:
    def numUniqueEmails(self, emails: List[str]) -> int:
        def normalize(email: str) -> str:
            local_name, domain_name = email.rsplit('@', maxsplit=1)
            normalized_local_name = local_name.split('+')[0]
            normalized_local_name = normalized_local_name.replace('.', '')
            
            return f'{normalized_local_name}@{domain_name}'
        
        return len(set(map(normalize, emails)))
```
思考ログ：
- 正規化の部分は関数に切り出した方が見通し良さそう
- '@'が複数入っている場合も対応できるようにした  
  ```local_name, domain_name = email.rsplit('@', maxsplit=1)```
- '+'以降を消す処理は```find```から```split```に変更
- 正規化メールアドレスの再構築はfstringで
- ```emails```への適用はループをやめて```map```に変更

一文字ずつ処理していく方法
```python
class Solution:
    def numUniqueEmails(self, emails: List[str]) -> int:
        def normalize(email: str) -> str:
            is_local_part = True
            is_ignore_part = False
            elements_of_normalization = []
            for char in email:
                if not is_local_part:
                    elements_of_normalization.append(char)
                    continue
                if char == '+':
                    is_ignore_part = True
                    continue
                if char == '@':
                    is_local_part = False
                    elements_of_normalization.append(char)
                    continue
                if char == '.' or is_ignore_part:
                    continue
                elements_of_normalization.append(char)
            
            return ''.join(elements_of_normalization)
            
        return len(set(map(normalize, emails)))
```
思考ログ：
- ```yield```も一考
  - そして```yield```をよく分かってなかったのでdocument
  - https://docs.python.org/ja/3/reference/expressions.html#yieldexpr

正規表現
```python
import re


class Solution:
    def numUniqueEmails(self, emails: List[str]) -> int:
        def normalize(email: str) -> str:
            m = re.match(r'([^+]+)(?:\+.*)?@(.+)', email)
            normalized_local_part = m[1].replace('.', '')
            domain_part = m[2]

            return f'{normalized_local_part}@{domain_part}'

        return len(set(map(normalize, emails)))
```
思考ログ：
- re
  - https://docs.python.org/ja/3/library/re.html
    > (?:...)  
    > 普通の丸括弧の、キャプチャしない版です。丸括弧で囲まれた正規表現にマッチしますが、このグループがマッチした部分文字列は、マッチを実行したあとで回収することも、そのパターン中で以降参照することもできません。
    
    > Match.group([group1, ...])
    > 複数の引数があれば、結果は引数ごとに 1 項目のタプルです。引数がなければ、 group1 はデフォルトで 0 (マッチ全体が返される) です。 groupN 引数が 0 なら、対応する返り値はマッチした文字列全体です。
  
    > 補集合 をとって範囲内にない文字にマッチできます。集合の最初の文字が '^' なら、集合に 含まれない 全ての文字にマッチします。例えば、 [^5] は '5' を除くあらゆる文字にマッチし、 [^^] は '^' を除くあらゆる文字にマッチします。
- やはり可読性は劣る気がする

# Step3

かかった時間：1min

```python
class Solution:
    def numUniqueEmails(self, emails: List[str]) -> int:
        def normalize(email: str) -> str:
            local_part, domain_part = email.rsplit('@', maxsplit=1)
            normalized_local_part = local_part.split('+')[0]
            normalized_local_part = normalized_local_part.replace('.', '')

            return f'{normalized_local_part}@{domain_part}'

        return len(set(map(normalize, emails)))
```
思考ログ：
- この手の話は気を遣わなくてはならないことが結構あるなあと改めて感じた

# Step4

findを使う方法
```python
class Solution:
    def numUniqueEmails(self, emails: List[str]) -> int:
        unique_email_address = set()
        for email in emails:
            local_part, domain_part = email.rsplit('@', maxsplit=1)
            
            index_of_plus = local_part.find('+')
            if index_of_plus != -1:
                local_part = local_part[:index_of_plus]
            local_part = local_part.replace('.', '')
            
            unique_email_address.add(f'{local_part}@{domain_part}')
        
        return len(unique_email_address)
```
思考ログ：
- 無駄なfindをしない
