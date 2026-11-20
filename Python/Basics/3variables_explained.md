# Python Variables: Walkthrough of `3variables.py`

This guide follows [3variables.py](./3variables.py) from top to bottom. Each section names the concept, shows what the file does, explains why, and gives a runnable example.

---

## 1. What is a variable?

```python
age = 21
name = "Sam"
```

A variable is a **name that points to a value** stored in memory. `=` is the *assignment operator*. It does not mean "equals" as in maths. It means "make the name on the left refer to the value on the right".

- No keyword such as `var`, `let` or `int` is needed to declare a variable. The first assignment creates it.
- `age` points to the integer `21`, and `name` points to the string `"Sam"`.

**Answer to "Guess the type":** `age` is `int` and `name` is `str`. Check with `type()`:

```python
age = 21
name = "Sam"
print(type(age))   # <class 'int'>
print(type(name))  # <class 'str'>
```

---

## 2. Dynamic typing: a variable can change type

```python
age = 21
age = "Sam"   # now age is a str
```

Python is **dynamically typed**. The *value* has a type, and the *variable name* does not. You can rebind a name to a value of a different type at any time.

```python
x = 10
print(type(x))   # <class 'int'>
x = "ten"
print(type(x))   # <class 'str'>
x = [1, 2, 3]
print(type(x))   # <class 'list'>
```

**Why it matters:** This is flexible, but it can cause bugs when you reuse a name by accident. Optional type hints document your intent:

```python
age: int = 21          # a hint only, Python does not enforce it
age = "Sam"            # still runs, but tools like mypy/Pylance will warn
```

Type hints matter a lot in FastAPI and Pydantic, where they drive validation.

---

## 3. Casting: converting between types

```python
int_example = int(55.5)       # 55
float_example = float(55.5)   # 55.5
string_example = str('55.5')  # '55.5'
```

Casting uses the type's constructor: `int()`, `float()`, `str()`, `bool()`, `list()` and so on.

| Call | Result | Note |
|------|--------|------|
| `int(55.5)` | `55` | **Truncates** toward zero. It does not round. |
| `int(-55.9)` | `-55` | Truncation, not floor. |
| `float(55)` | `55.0` | |
| `str(55.5)` | `'55.5'` | |
| `int("42")` | `42` | A string works if it looks like an integer. |
| `int("55.5")` | `ValueError` | Go through `float()` first: `int(float("55.5"))`. |

```python
print(int(55.9))            # 55 (truncated)
print(round(55.9))          # 56 (rounded)
print(int("42") + 8)        # 50
print(int(float("55.5")))   # 55

try:
    int("abc")
except ValueError as e:
    print("Cannot cast:", e)
```

**Strings can use single or double quotes.** `'55.5'` and `"55.5"` are identical. Pick one style and stay consistent. Use the other kind when the text contains a quote: `"it's fine"`.

---

## 4. Mixing types in arithmetic

```python
print(int_example)                  # 55
print(float_example)                # 55.5
print(string_example)               # 55.5   (it's a string, but prints the same)
print(int_example + float_example)  # 110.5  (int + float -> float)
#print(int_example + string_example)  # TypeError
```

- `int + float` gives a `float`. Python promotes the int automatically.
- `int + str` raises `TypeError: unsupported operand type(s) for +: 'int' and 'str'`. Python will not guess whether you mean addition or text joining. That is why the line is commented out.
- `print(float_example)` and `print(string_example)` look identical, but `type()` shows they differ.

Fix it by casting explicitly:

```python
a = 55
b = "55.5"
print(a + float(b))        # 110.5  (numeric add)
print(str(a) + b)          # 5555.5 (text join)
```

---

## 5. Case sensitivity

```python
small = 6
Small = 7
print(type(small))   # <class 'int'>
```

**Answer: different variables.** Python names are **case-sensitive**: `small`, `Small` and `SMALL` are three separate names. The file prints `<class 'int'>` because `small` is `6`.

```python
count = 1
Count = 2
print(count, Count)   # 1 2
```

**Naming rules:**
- Must start with a letter or underscore, never a digit (`2name` is invalid).
- Can contain letters, digits and underscores only (no `-` or spaces).
- Cannot be a keyword (`if`, `for`, `class`, `None`, and so on).

---

## 6. Naming conventions for multi-word names

```python
camelCaseAdd = "camel case"      # camelCase    -> JavaScript/Java style
PascalCaseAdd = "Pascal case"    # PascalCase   -> used for CLASS names in Python
snake_case = "snake case"        # snake_case   -> the Python standard (PEP 8)
```

| Style | Example | Used in Python for |
|-------|---------|--------------------|
| `snake_case` | `user_name` | **Variables, functions, modules** |
| `PascalCase` | `UserProfile` | **Classes** |
| `UPPER_SNAKE_CASE` | `MAX_RETRIES` | **Constants** (a convention only; Python doesn't enforce it) |
| `camelCase` | `userName` | Not idiomatic. It is common in JS, which is why the file's comment mentions "FE languages". |

```python
MAX_RETRIES = 3            # constant by convention

class UserProfile:         # class -> PascalCase
    pass

def get_user_name():       # function -> snake_case
    user_name = "Sam"      # variable -> snake_case
    return user_name
```

---

## 7. Assigning many values at once

### Many values to many variables

```python
example_1, example_2, example_3 = "value 1", "value 2", "value 3"
```

The right side is a tuple, which Python **unpacks** into the names on the left. The counts must match, or you get a `ValueError`.

```python
a, b, c = 1, 2, 3
print(a, b, c)    # 1 2 3

# Swap two variables without a temp variable
a, b = b, a
print(a, b)       # 2 1

# a, b = 1, 2, 3  -> ValueError: too many values to unpack
```

### One value to many variables

```python
example_4 = example_5 = "orange"
```

Both names point to the **same object**. This is fine for immutable values (str, int, tuple). Be careful with mutable ones:

```python
x = y = "orange"
print(x is y)        # True, the same object

list_a = list_b = [1, 2]
list_a.append(3)
print(list_b)        # [1, 2, 3], list_b changed too, because it's the same list!

# To get independent lists:
list_a = [1, 2]
list_b = [1, 2]
```

---

## 8. Unpacking a collection

```python
student_name, student_age = ["Sam", 21]
```

Any iterable (list, tuple, set, string, dict keys) can be unpacked into variables.

```python
point = (10, 20)
x, y = point
print(x, y)               # 10 20

first, second, third = "abc"
print(first, second, third)   # a b c

# Starred unpacking collects the leftovers into a list
head, *rest = [1, 2, 3, 4]
print(head, rest)         # 1 [2, 3, 4]

first, *middle, last = [1, 2, 3, 4, 5]
print(first, middle, last)   # 1 [2, 3, 4] 5

# Use _ for values you don't care about
_, age = ("Sam", 21)
print(age)                # 21
```

---

## 9. Tuples and "guess the type"

```python
english_mark, hidi_mark, cs_mark = (85, 90, 95)
```

**Types:** the right side `(85, 90, 95)` is a **`tuple`**. After unpacking, each of `english_mark`, `hidi_mark` and `cs_mark` is an **`int`**.

```python
marks = (85, 90, 95)
print(type(marks))        # <class 'tuple'>

english, hindi, cs = marks
print(type(english))      # <class 'int'>
print(english + hindi + cs)   # 270
```

Tuples are **immutable**. You can't change an item after creation:

```python
marks = (85, 90, 95)
# marks[0] = 100   -> TypeError: 'tuple' object does not support item assignment
```

Parentheses are optional. `85, 90, 95` is already a tuple, and the file's next line `english_mark, hidi_mark, cs_mark = 85, 90, 95` works the same way.

**Heads-up:** In the file, `total_marks` is calculated **before** the second assignment, but the values are the same, so the result is `270` either way. If the values had changed in between, `total_marks` would **not** update. Variables hold the result at that moment, not a formula.

```python
a = 1
b = 2
total = a + b     # 3, computed now
a = 100
print(total)      # still 3
```

(Also note `hidi_mark` is probably a typo for `hindi_mark`. It works, but clear names help readability.)

---

## 10. Global vs local variables (scope)

This is the most important concept in the file.

### Case A: assigning inside a function creates a *new local* variable

```python
global_variable = "I am a global variable"

def global_function_example():
    global_variable = "I am a local variable"   # NEW local variable
    print(global_variable)

global_function_example()   # I am a local variable
print(global_variable)      # I am a global variable   (unchanged)
```

Inside the function, `global_variable = ...` creates a **separate local variable** that happens to share the name. It disappears when the function ends. The global one is untouched.

### Case B: the `global` keyword modifies the global variable

```python
global_variabl_1 = "I am a global variable"

def global_function_example1():
    global global_variabl_1                     # "use the module-level one"
    global_variabl_1 = "I am a local variable"  # changes the GLOBAL
    print(global_variabl_1)

global_function_example1()   # I am a local variable
print(global_variabl_1)      # I am a local variable   (changed!)
```

The text says "local" but the variable is global. The message is misleading, and it's the *point* of the demo: the global value really did change.

### Reading vs writing

- **Reading** a global inside a function works without `global`.
- **Assigning** to it requires `global`. Without it, Python treats the name as local.

```python
counter = 0

def read_counter():
    print(counter)       # OK, just reads the global

def broken_increment():
    counter += 1         # UnboundLocalError! Python sees an assignment, so it assumes 'counter' is local

def increment():
    global counter
    counter += 1         # works

read_counter()           # 0
increment()
print(counter)           # 1
```

### Best practice

Avoid `global` where you can. Prefer passing values in and returning them out:

```python
def increment(value):
    return value + 1

counter = 0
counter = increment(counter)
print(counter)   # 1
```

This is easier to test and debug, and it avoids hidden side effects.

### Bonus: scope order (LEGB)

Python looks up a name in this order: **L**ocal → **E**nclosing function → **G**lobal → **B**uilt-in.

```python
x = "global"

def outer():
    x = "enclosing"
    def inner():
        print(x)       # finds 'enclosing' first
    inner()

outer()    # enclosing
print(x)   # global
```

---

## Expected output of `3variables.py`

```
55
55.5
55.5
110.5
<class 'int'>
Sam
21
85 90 95 270
I am a local variable
I am a global variable
I am a local variable
I am a local variable
```

## Quick recap

| Concept | One-line takeaway |
|---------|-------------------|
| Variable | A name bound to a value with `=`. |
| Dynamic typing | The value has the type, and a name can be rebound to any type. |
| Casting | `int()`, `float()` and `str()` convert explicitly. `int()` truncates. |
| Type mixing | `int + float` works. `int + str` raises `TypeError`. |
| Case sensitivity | `small` and `Small` are different variables. |
| Naming | `snake_case` for variables and functions, `PascalCase` for classes, `UPPER_CASE` for constants. |
| Multiple assignment | `a, b = 1, 2` unpacks. `a = b = 0` shares one object. |
| Unpacking | Works on any iterable, with `*rest` to collect leftovers. |
| Tuples | Immutable sequences, such as `(85, 90, 95)`. |
| Scope | Assigning in a function makes a local variable. Use `global` to change a module-level one, or better, return values. |
