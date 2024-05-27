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
