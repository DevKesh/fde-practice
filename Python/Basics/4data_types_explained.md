# Python Data Types: Walkthrough of `4data_types.py`

This guide follows [4data_types.py](./4data_types.py). The file reassigns one variable `x` to a value of each built-in type. Because Python is dynamically typed, `x` can hold a different type each time. Use `type(x)` to see the current one.

> **Note on the file:** every line reassigns `x`, so only `x = None` is left at the end. Add `print(type(x))` after each line to see each type as you go.

---

## 1. Text: `str`
```python
x = "Hello World"
```
An immutable sequence of characters.
```python
s = "Hello World"
print(type(s))        # <class 'str'>
print(s.upper())      # HELLO WORLD
print(s[0], s[-1])    # H d  (indexing)
print(s[0:5])         # Hello  (slicing)
print(f"Say: {s}")    # f-string formatting
# s[0] = "J"  -> TypeError (immutable)
```

## 2. Numeric: `int`, `float`, `complex`
```python
x = 20      # int
x = 20.5    # float
x = 1j      # complex
```
- `int` has unlimited size. `float` is a 64-bit decimal. `complex` has a real and an imaginary part (`j` is the imaginary unit).
```python
print(type(20), type(20.5), type(1j))
print(10 / 3)      # 3.333...  (/ always returns float)
print(10 // 3)     # 3         (floor division)
print(10 % 3)      # 1         (remainder)
print(2 ** 100)    # huge int, no overflow
print(0.1 + 0.2)   # 0.30000000000000004  (float precision)
c = 3 + 4j
print(c.real, c.imag, abs(c))   # 3.0 4.0 5.0
```
For exact money-style decimals, use `from decimal import Decimal`.

## 3. Sequences: `list`, `tuple`, `range`
```python
x = ["apple", "banana", "cherry"]   # list
x = ("apple", "banana", "cherry")   # tuple
x = range(6)                        # range
```
| Type | Syntax | Mutable? | Use |
|------|--------|----------|-----|
| `list` | `[...]` | Yes | Ordered collection you change |
| `tuple` | `(...)` | No | Fixed record, can be a dict key |
| `range` | `range(n)` | No | Lazy sequence of numbers |
```python
fruits = ["apple", "banana"]
fruits.append("cherry")
fruits[0] = "mango"
print(fruits, len(fruits))     # ['mango', 'banana', 'cherry'] 3

point = (10, 20)
# point[0] = 5  -> TypeError
single = (5,)                  # a one-item tuple needs the comma

r = range(6)
print(list(r))                 # [0, 1, 2, 3, 4, 5]  (stop is excluded)
print(list(range(2, 10, 3)))   # [2, 5, 8]  (start, stop, step)
```
`range` doesn't store the numbers, so `range(10**9)` uses almost no memory.

## 4. Mapping: `dict`
```python
x = {"name": "John", "age": 36}
```
Key-value pairs. Keys must be hashable (str, int, tuple, ...). Insertion order is preserved.
```python
person = {"name": "John", "age": 36}
print(person["name"])              # John
print(person.get("city", "N/A"))   # N/A  (safe lookup)
person["city"] = "Pune"            # add or update
for key, value in person.items():
    print(key, value)
# person["zip"] -> KeyError if missing
```
FastAPI JSON bodies map directly onto dicts.

## 5. Sets: `set`, `frozenset`
```python
x = {"apple", "banana", "cherry"}             # set
x = frozenset({"apple", "banana", "cherry"})  # frozenset
```
Unordered collections of **unique** items. `set` is mutable and `frozenset` is immutable.
```python
s = {1, 2, 2, 3}
print(s)                    # {1, 2, 3}  (duplicates removed)
s.add(4)
print({1, 2} | {2, 3})      # {1, 2, 3}  union
print({1, 2} & {2, 3})      # {2}        intersection
print({1, 2} - {2, 3})      # {1}        difference
print(2 in s)               # True  (fast lookup)

fs = frozenset([1, 2])
# fs.add(3) -> AttributeError
empty_set = set()           # {} would create an empty dict!
```
A common use is deduplicating a list with `list(set(items))`.

## 6. Boolean: `bool`
```python
x = True
```
`True` or `False`. `bool` is a subclass of `int`, so `True == 1`.
```python
print(5 > 2)                       # True
print(True + True)                 # 2
print(bool(0), bool(""), bool([]), bool(None))   # all False
print(bool(1), bool("a"), bool([0]))             # all True
```
**Falsy values:** `0`, `0.0`, `""`, `[]`, `()`, `{}`, `set()`, `None`, `False`. Everything else is truthy.

## 7. Binary: `bytes`, `bytearray`, `memoryview`
```python
x = b"Hello"                       # bytes
x = bytearray(5)                   # bytearray
x = memoryview(bytes(5))           # memoryview
```
| Type | Mutable? | Meaning |
|------|----------|---------|
| `bytes` | No | Raw bytes (values 0–255) |
| `bytearray` | Yes | Editable bytes |
| `memoryview` | n/a | A view onto another object's bytes with no copy |
```python
b = b"Hello"
print(b[0])                        # 72  (ints, not characters)
print("héllo".encode("utf-8"))     # str -> bytes
print(b.decode("utf-8"))           # bytes -> str

ba = bytearray(5)                  # 5 zero bytes
ba[0] = 65
print(ba)                          # bytearray(b'A\x00\x00\x00\x00')

mv = memoryview(b"Hello")
print(mv[1], bytes(mv[1:3]))       # 101 b'el'
```
Bytes appear in files, network data, images and uploads (FastAPI's `UploadFile`).

## 8. `None`: `NoneType`
```python
x = None
```
The single value meaning "no value". Compare with `is`, not `==`.
```python
result = None
print(type(result))        # <class 'NoneType'>
if result is None:
    print("nothing yet")

def no_return():
    pass
print(no_return())         # None  (functions return None by default)
```
In FastAPI, `Optional[str] = None` marks an optional field.

## 9. Utilities for working with types
```python
x = 20.5
print(type(x))                  # <class 'float'>
print(isinstance(x, float))     # True
print(isinstance(x, (int, float)))   # True
```
Prefer `isinstance()` over `type(x) == ...`, since it handles subclasses.

---

## Mutable vs immutable
| Immutable | Mutable |
|-----------|---------|
| `str`, `int`, `float`, `complex`, `bool`, `tuple`, `range`, `frozenset`, `bytes`, `None` | `list`, `dict`, `set`, `bytearray` |

Mutable objects can change in place, which matters for shared references and function arguments. Only immutable (hashable) values can be dict keys or set members.

## Quick recap
| Category | Types | Example |
|----------|-------|---------|
| Text | `str` | `"Hello"` |
| Numeric | `int`, `float`, `complex` | `20`, `20.5`, `1j` |
| Sequence | `list`, `tuple`, `range` | `[1]`, `(1,)`, `range(6)` |
| Mapping | `dict` | `{"a": 1}` |
| Set | `set`, `frozenset` | `{1, 2}`, `frozenset({1})` |
| Boolean | `bool` | `True` |
| Binary | `bytes`, `bytearray`, `memoryview` | `b"Hi"` |
| None | `NoneType` | `None` |
