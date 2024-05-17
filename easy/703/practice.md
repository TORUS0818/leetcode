# priority_queueの実装

# Step1
色々見た実装を思い出しながら書いてみた
```python
class MyHeap:
    def __init__(self, array: list = []):
        self.replace_array(array)
    
    def replace_array(self, array: list):
        self.heap_array = array
        self._heapify(array)

    def _heapify(self, array: list):
        for i in reversed(range(len(array))):
            self._sift_up(i)
            
    def pop(self) -> int:
        if not self.heap_array:
            return 
        
        self._swap(0, len(self.heap_array) - 1)
        min_val = self.heap_array.pop()
        self._sift_up(0)
        return min_val

    def push(self, val: int) -> None:
        self.heap_array.append(val)
        self._shift_down(len(self.heap_array) - 1)

    def _sift_up(self, parent_index: int) -> None:
        if not self._has_child(parent_index):
            return
        
        min_child_index = self._get_min_child_index(parent_index)
        if self.heap_array[min_child_index] < self.heap_array[parent_index]:
            self._swap(min_child_index, parent_index)
        self._sift_up(min_child_index)

    def _shift_down(self, child_index: int) -> None:
        parent_index = (child_index - 1) // 2
        if self.heap_array[child_index] < self.heap_array[parent_index]:
            self._swap(child_index, parent_index)
        
        if parent_index > 0:
            self._shift_down(parent_index)

    def _get_min_child_index(self, parent_index: int) -> int:
        if not self._has_child(parent_index):
            return None
        
        left_child_index = 2 * parent_index + 1
        right_child_index = 2 * parent_index + 2
        if not self._has_right_child(parent_index):
            return left_child_index
        
        if self.heap_array[left_child_index] < self.heap_array[right_child_index]:
            return left_child_index
        else:
            return right_child_index

    def _has_child(self, node_index: int) -> bool:
        return self._has_left_child(node_index)
    
    def _has_left_child(self, node_index: int) -> bool:
        if 2 * node_index + 1 < len(self.heap_array):
            return True
        return False
    
    def _has_right_child(self, node_index: int) -> bool:
        if 2 * node_index + 2 < len(self.heap_array):
            return True
        return False

    def _swap(self, i: int, j: int) -> None:
        assert 0 <= i < len(self.heap_array) and 0 <= j < len(self.heap_array)
        self.heap_array[i], self.heap_array[j] = self.heap_array[j], self.heap_array[i]


if __name__ == '__main__':
    # 初期化
    my_heap = MyHeap([3,4,7,5,2,6,1])
    print(my_heap.heap_array) # [1, 2, 3, 5, 4, 6, 7]
    
    # 他の配列をセットし直す
    my_heap.replace_array([7,6,5,4,3,2,1])
    print(my_heap.heap_array) # [1, 3, 2, 4, 6, 7, 5]
    
    # pop
    print(my_heap.pop()) # 1
    print(my_heap.pop()) # 2
    print(my_heap.heap_array) # [3, 4, 5, 7, 6]

    # push
    my_heap.push(1)
    my_heap.push(2)
    print(my_heap.heap_array) # [1, 4, 2, 7, 6, 5, 3]
```
思考ログ：
- とりあえず書けたが、色々見返してみて修正したい部分が出てきたのでStep2で清書する

# Step2

```python
class MyHeap:
    def __init__(self, array: list = []):
        self.heap_array = []
        if array:
            self.replace_heap_array(array)

    def replace_heap_array(self, array: list) -> None:
        self.heap_array = array
        self._heapify(array)

    def _heapify(self, array: list) -> None:
        for i in reversed(range(len(array))):
            self._sift_up(i)

    def pop(self) -> int:
        if not self.heap_array:
            return

        self._swap(0, len(self.heap_array) - 1)
        min_val = self.heap_array.pop()
        self._sift_up(0)
        return min_val

    def push(self, val: int) -> None:
        self.heap_array.append(val)
        self._shift_down(len(self.heap_array) - 1)

    def _sift_up(self, parent_index: int) -> None:
        smaller_child_index = self._get_smaller_child_index(parent_index)
        if not smaller_child_index:
            return

        if self.heap_array[smaller_child_index] < self.heap_array[parent_index]:
            self._swap(smaller_child_index, parent_index)
        self._sift_up(smaller_child_index)

    def _shift_down(self, child_index: int) -> None:
        parent_index = self._get_parent_index(child_index)
        if self.heap_array[child_index] < self.heap_array[parent_index]:
            self._swap(child_index, parent_index)

        if parent_index > 0:
            self._shift_down(parent_index)

    def _get_parent_index(self, child_index: int) -> int:
        return (child_index - 1) // 2

    def _get_smaller_child_index(self, parent_index: int) -> int:
        if not self._has_child(parent_index):
            return None

        left_child_index = self._get_left_child_index(parent_index)
        right_child_index = self._get_right_child_index(parent_index)

        smaller_child_index = left_child_index
        if (
            not right_child_index
            and self.heap_array[right_child_index] < self.heap_array[left_child_index]
        ):
            smaller_child_index = right_child_index

        return smaller_child_index

    def _get_left_child_index(self, parent_index: int) -> int:
        if not self._has_left_child(parent_index):
            return None
        return 2 * parent_index + 1

    def _get_right_child_index(self, parent_index: int) -> int:
        if not self._has_right_child(parent_index):
            return None
        return 2 * parent_index + 2

    def _has_child(self, node_index: int) -> bool:
        return self._has_left_child(node_index)

    def _has_left_child(self, node_index: int) -> bool:
        return 2 * node_index + 1 < len(self.heap_array)

    def _has_right_child(self, node_index: int) -> bool:
        return 2 * node_index + 2 < len(self.heap_array)

    def _swap(self, i: int, j: int) -> None:
        assert 0 <= i < len(self.heap_array) and 0 <= j < len(self.heap_array)
        self.heap_array[i], self.heap_array[j] = self.heap_array[j], self.heap_array[i]


if __name__ == "__main__":
    # 初期化
    my_heap = MyHeap([3, 4, 7, 5, 2, 6, 1])
    print(my_heap.heap_array)  # [1, 2, 3, 5, 4, 6, 7]

    # 他の配列をセットし直す
    my_heap.replace_heap_array([7, 6, 5, 4, 3, 2, 1])
    print(my_heap.heap_array)  # [1, 3, 2, 4, 6, 7, 5]

    # pop
    print(my_heap.pop())  # 1
    print(my_heap.pop())  # 2
    print(my_heap.heap_array)  # [3, 4, 5, 7, 6]

    # push
    my_heap.push(1)
    my_heap.push(2)
    print(my_heap.heap_array)  # [1, 4, 2, 7, 6, 5, 3]
    
```
思考ログ：
- ```_has_child```系のif文は不要
- ```_get_min_child_index```はミスリードする可能性があるのでヤメ（欲しいのは最小の子ではない）
  - ```_get_smaller_child```に変更
- 32~33と49~50が被っていて無駄なので統合する
- 子や親のインデックスを取得する関数を追加
- 初期化のデザインが微妙な気がするが、、うーん
