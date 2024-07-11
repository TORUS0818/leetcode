# デコレータの理解とLRUCacheの実装

# Step1

まずデコレータの実装を試してみる
```python
from functools import wraps


def cache(func):
    cache = {}
    @wraps(func)
    def wrapper(*args, **kwargs):
        if args in cache:
            # print('found cache')
            return cache[args]
        
        result = func(*args, **kwargs)
        cache[args] = result
        return result

    return wrapper

def test_raw():
    def fib(n):
        if n < 2:
            return n
        return fib(n - 1) + fib(n - 2)
    
    return fib(20)

def test_cached():
    @cache
    def fib(n):
        if n < 2:
            return n
        return fib(n - 1) + fib(n - 2)
    
    return fib(20)

if __name__ == '__main__':
    repeat_n = 1000
    print(timeit.timeit(test_raw, number=repeat_n) / repeat_n) # 0.00127596425
    print(timeit.timeit(test_cached, number=repeat_n) / repeat_n) # 8.028708000000107e-06
```
思考ログ：
- まずはドキュメント
  - https://docs.python.org/ja/3/library/functools.html
     > @functools.lru_cache
     
     > 結果のキャッシュには辞書が使われるので、関数の位置引数およびキーワード引数は ハッシュ可能 でなくてはなりません。
     
     > キャッシュ効率の測定や maxsize パラメータの調整をしやすくするため、ラップされた関数には cache_info() 関数が追加されます。
     
     > このデコレータは、キャッシュの削除と無効化のための cache_clear() 関数も提供します。
- 実装
  - https://github.com/python/cpython/blob/1a6e2138773b94fdae449b658a9983cd1fc0f08a/Lib/functools.py
- 本来はスレッドの排他制御も考えなくてはならない
- @functools.wraps
  > これはラッパー関数を定義するときに update_wrapper() を関数デコレータとして呼び出す便宜関数です。
  - 正しく元の関数の名前やDocstringを表示できるようにする
- 最初違和感があるがだんだん慣れてきた

# Step2

LRUCacheの実装
```python
import timeit
from typing import Optional

from functools import wraps


class ListNode:
    def __init__(self, key=None, val=None, prev=None, next=None):
        self.key = key
        self.val = val
        self.prev = prev
        self.next = next

class DoubleLinkedList:
    def __init__(self):
        self.head = ListNode()
        self.tail = ListNode()
        self.head.next = self.tail
        self.tail.prev = self.head
    
    def add(self, node: ListNode):
        node.next = self.head.next
        node.prev = self.head
        self.head.next = node
        node.next.prev = node
    
    @staticmethod
    def remove(node: ListNode):
        node.next.prev = node.prev
        node.prev.next = node.next

    def move_to_head(self, node: ListNode):
        self.remove(node)
        self.add(node)

class LRUCache:
    def __init__(self, capacity: int = 100):
        self.capacity = capacity
        self.key_to_node = {}
        self.linked_list = DoubleLinkedList()
    
    def __str__(self):
        node = self.linked_list.head.next
        items = []
        while node != self.linked_list.tail:
            items.append(f'ListNode(key: {node.key}, value: {node.val})')
            node = node.next
        return ' <-> '.join(items)

    def get(self, key) -> Optional[int]:
        if key not in self.key_to_node:
            return None
        node = self.key_to_node[key]
        self.linked_list.move_to_head(node)
        return node.val
    
    def put(self, key, val) -> None:
        if key in self.key_to_node:
            self.key_to_node[key].val = val
            self.linked_list.move_to_head(self.key_to_node[key])
            return None
        
        self.key_to_node[key] = ListNode(key, val)
        self.linked_list.add(self.key_to_node[key])
        
        if len(self.key_to_node) > self.capacity:
            del self.key_to_node[self.linked_list.tail.prev.key]
            self.linked_list.remove(self.linked_list.tail.prev)

def lru_cache(func):
    cache = LRUCache()
    @wraps(func)
    def wrapper(*args, **kwargs):
        if args in cache.key_to_node:
            return cache.key_to_node[args].val
        output = func(*args, **kwargs)
        cache.put(args, output)
        return output
        
    return wrapper

def test_raw():
    def fib(n):
        if n < 2:
            return n
        return fib(n - 1) + fib(n - 2)
    
    return fib(20)

def test_cached():
    @lru_cache
    def fib(n):
        if n < 2:
            return n
        return fib(n - 1) + fib(n - 2)
    
    return fib(20)

if __name__ == '__main__':
    repeat_n = 1000
    print(timeit.timeit(test_raw, number=repeat_n) / repeat_n) # 0.0012727971250000001
    print(timeit.timeit(test_cached, number=repeat_n) / repeat_n) # 2.2349541999999944e-05
```
思考ログ：
- 色々調べながら実装
- LinkedListの復習にもなってよかった、が少し疲れた
