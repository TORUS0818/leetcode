# defaultdictの実装

dictをメンバとして持つ方式
```python
class MyDefaultDict():
    def __init__(self, default_factory):
        self.mydict = dict()
        self.default_factory = default_factory
    
    def __setitem__(self, key, value):
        self.mydict[key] = value

    def __getitem__(self, key):
        if key not in self.mydict:
            self.mydict[key] = self.default_factory()
        return self.mydict[key]
```

dictを継承する方式
```python
class MyDefaultDict(dict):
    def __init__(self, default_factory = None):
        super().__init__()
        self.default_factory = default_factory
    
    def __missing__(self, key):
        if self.default_factory is None:
            raise KeyError
        self[key] = self.default_factory()
```
思考ログ：
- defaultdictの実装のテーマで結構議論が盛んだったので追ってみる
  - https://discord.com/channels/1084280443945353267/1225849404037009609/1228028878589657150
  - pythonのclassについて
    - https://docs.python.org/ja/3.12/tutorial/classes.html
  - __getitem__, __setitem__, __missing__
    - https://docs.python.org/ja/3/reference/datamodel.html#emulating-container-types

# quicksort/selectの研究
- median of median
    - https://ja.wikipedia.org/wiki/%E4%B8%AD%E5%A4%AE%E5%80%A4%E3%81%AE%E4%B8%AD%E5%A4%AE%E5%80%A4
- quicksort
    - https://ja.wikipedia.org/wiki/%E3%82%AF%E3%82%A4%E3%83%83%E3%82%AF%E3%82%BD%E3%83%BC%E3%83%88
    - これを参考にもう一度書いてみる

quickselect実装
```python
def partition(array, pivot) -> list:
    left, right = 0, len(array)-1
    while True:
        while array[left] < pivot:
            left += 1
        while pivot < array[right]:
            right -= 1
        if left >= right:
            return array[:right+1], array[right+1:]
        array[left], array[right] = array[right], array[left]
        left += 1
        right -= 1

def quick_sort(array, verbose=False):
    if len(array) <= 1:
        return array
    pivot_index = random.randint(0, len(array)-1)
    pivot = array[pivot_index]
    sub_array_1, sub_array_2 = partition(array, pivot)
    if verbose:    
        print(f'pivot: {pivot}')
        print(f'array: {array}')
        print(f'-> sub_array_1: {sub_array_1}')
        print(f'-> sub_array_2: {sub_array_2}')
        print('-' * len(array))
    
    return quick_sort(sub_array_1) + quick_sort(sub_array_2)

if __name__ == '__main__':
    # random.seed(71)
    sample = [3,4,2,5,7,4,8,9,1,1]
    print(quick_sort(sample))
```
思考ログ：
- いくつかの常識
    - 最悪計算量O(N^2)
        - 分割が非効率になるようにピボットを選んでいくとこうなる
        - ピボット選択
            - 乱択
            - median of median
    - 平均計算量0(NlogN)
    - 末尾再帰最適化
        - https://takegue.hatenablog.com/entry/2014/12/18/021137
        - 末尾呼び出しのコードを戻り先を保存しないジャンプに変換することで、スタックの累計をなくし、効率の向上を図るテクニック
    - 安定ソートではない
