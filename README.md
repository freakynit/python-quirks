### 1. **Late Binding Closures in Loops**
Python’s closures capture variables by reference, not by value, which can lead to surprising results in loops when creating functions.

**Example**:
```python
def create_multipliers():
    return [lambda x: x * i for i in range(3)]

multipliers = create_multipliers()
print(multipliers[0](2))  # Output: 4 (not 0)
print(multipliers[1](2))  # Output: 4 (not 2)
print(multipliers[2](2))  # Output: 4 (not 4)
```

**Why it’s counter-intuitive**: Each lambda captures the variable `i` by reference, so all lambdas use the final value of `i` (2) after the loop ends, not the value `i` had when the lambda was created.

**Workaround**: Use a default argument in the lambda to capture the current value of `i`:
```python
def create_multipliers():
    return [lambda x, i=i: x * i for i in range(3)]

multipliers = create_multipliers()
print(multipliers[0](2))  # Output: 0
print(multipliers[1](2))  # Output: 2
print(multipliers[2](2))  # Output: 4
```

### 2. **Integer Caching for Small Integers**
Python caches small integers (typically -5 to 256) for efficiency, so the same object is reused, which can lead to confusing behavior with `is` vs. `==`.

**Example**:
```python
a = 256
b = 256
print(a is b)  # Output: True

a = 257
b = 257
print(a is b)  # Output: False (may vary by Python implementation)
```

**Why it’s counter-intuitive**: The `is` operator checks for object identity, and small integers share the same object due to caching, but larger integers don’t, leading to inconsistent results.

**Workaround**: Use `==` for value equality, not `is`, unless you specifically need to check object identity.

### 3. **Mutable Class Variables**
Class variables in Python are shared across all instances of a class, and if they’re mutable, modifications affect all instances.

**Example**:
```python
class Dog:
    tricks = []  # Shared class variable

    def add_trick(self, trick):
        self.tricks.append(trick)

dog1 = Dog()
dog1.add_trick("roll over")
dog2 = Dog()
dog2.add_trick("play dead")
print(dog1.tricks)  # Output: ['roll over', 'play dead']
```

**Why it’s counter-intuitive**: You might expect `tricks` to be unique to each `Dog` instance, but since it’s a class variable, all instances share the same list.

**Workaround**: Use instance variables for data that should be unique to each instance:
```python
class Dog:
    def __init__(self):
        self.tricks = []  # Instance variable

    def add_trick(self, trick):
        self.tricks.append(trick)
```

### 4. **Unexpected Behavior with `+=` on Immutable Types**
The `+=` operator can behave differently depending on whether the object is mutable or immutable, leading to confusion.

**Example**:
```python
def modify_list(lst):
    lst += [1]  # Modifies the list in place

def modify_tuple(tup):
    tup += (1,)  # Creates a new tuple, doesn't modify the original

my_list = [1, 2]
my_tuple = (1, 2)
modify_list(my_list)
modify_tuple(my_tuple)
print(my_list)   # Output: [1, 2, 1]
print(my_tuple)  # Output: (1, 2) (unchanged)
```

**Why it’s counter-intuitive**: For mutable objects like lists, `+=` modifies the object in place. For immutable objects like tuples, `+=` creates a new object, and the original remains unchanged, but the assignment doesn’t propagate outside the function.

**Workaround**: Be explicit about whether you’re modifying in place or reassigning, and avoid using `+=` with immutable types if you expect in-place modification.

### 5. **Variable Scope in List Comprehensions (Python 2 vs. 3)**
In Python 3, list comprehensions have their own scope, but in Python 2, they leak variables to the outer scope, which can be surprising.

**Example (Python 2)**:
```python
i = 1
[j for j in range(3)]
print(i)  # Output: 1 (in Python 3)
          # Output: 2 (in Python 2, because `i` is overwritten by the loop)
```

**Why it’s counter-intuitive**: In Python 2, the loop variable `j` leaks to the outer scope, potentially overwriting variables. Python 3 fixes this by giving list comprehensions their own scope.

**Workaround**: Use Python 3, where this issue is resolved, or avoid reusing loop variable names in the outer scope in Python 2.

### 6. **Global/Nonlocal Keyword Omission**
Python’s scoping rules require explicit `global` or `nonlocal` declarations to modify variables in outer scopes, which can catch beginners off guard.

**Example**:
```python
x = 10
def modify():
    x = 20  # Creates a new local variable `x`
    print(x)

modify()
print(x)  # Output: 20 (inside function), 10 (outside)
```

**Why it’s counter-intuitive**: Without `global`, assigning to `x` creates a new local variable, leaving the outer `x` unchanged, which might not be what you expect.

**Workaround**: Use `global` or `nonlocal` explicitly:
```python
x = 10
def modify():
    global x
    x = 20
    print(x)

modify()
print(x)  # Output: 20 (inside and outside)
```

### 7. **Boolean Evaluation of Non-Boolean Types**
Python’s truthiness rules can be surprising, as many non-boolean types evaluate to `True` or `False` in unexpected ways.

**Example**:
```python
print(bool([]))      # Output: False
print(bool([0]))     # Output: True
print(bool(0))       # Output: False
print(bool(-1))      # Output: True
print(bool("False")) # Output: True
```

**Why it’s counter-intuitive**: Empty sequences (`[]`, `""`, `()`) evaluate to `False`, while non-empty sequences or non-zero numbers evaluate to `True`, even if the value seems “falsey” (like `"False"` or `-1`).

**Workaround**: Be explicit about conditions (e.g., `if x == False` instead of `if not x`) when you need specific value checks.

### 8. **Method Resolution Order (MRO) in Multiple Inheritance**
Python’s MRO in multiple inheritance (using the C3 linearization algorithm) can lead to unexpected method resolution.

**Example**:
```python
class A:
    def method(self):
        print("A")

class B(A):
    pass

class C(A):
    def method(self):
        print("C")

class D(B, C):
    pass

d = D()
d.method()  # Output: C
```

**Why it’s counter-intuitive**: You might expect `A`’s method to be called since `B` inherits from `A`, but Python’s MRO prioritizes `C` because it appears later in the inheritance list (`B, C`).

**Workaround**: Check the MRO with `D.__mro__` to understand the resolution order, and design inheritance hierarchies carefully.

### 9. **Floating-Point Arithmetic Precision**
Floating-point arithmetic in Python (and most languages) can produce unexpected results due to IEEE 754 representation.

**Example**:
```python
print(0.1 + 0.2)  # Output: 0.30000000000000004
```

**Why it’s counter-intuitive**: Floating-point numbers are approximations, so simple additions can introduce small errors.

**Workaround**: Use the `decimal` module for precise arithmetic or round results:
```python
from decimal import Decimal
print(Decimal('0.1') + Decimal('0.2'))  # Output: 0.3
```

### 10. **Dictionary Modification During Iteration**
Modifying a dictionary (size-wise only) while iterating over it raises a `RuntimeError`.

**Example**:
```python
d = {'a': 1, 'b': 2}
for k in d:
    del d[k]  # Raises RuntimeError: dictionary changed size during iteration
```

**Why it’s counter-intuitive**: You might expect to be able to modify a dictionary while iterating, but Python prohibits this to avoid inconsistent behavior.

**Workaround**: Iterate over a copy of the dictionary’s keys:
```python
d = {'a': 1, 'b': 2}
for k in list(d.keys()):
    del d[k]
print(d)  # Output: {}
```

### 11. **Mutable Default Arguments Persist Across Calls**
In Python, default arguments are evaluated once at function definition time, not at each function call. If the default is a mutable object (like a list or dict), all calls that don’t override the argument will share the same object, leading to unexpected behavior.

**Example**:
```python
def append_item(item, my_list=[]):
    my_list.append(item)
    return my_list

print(append_item(1))  # Output: [1]
print(append_item(2))  # Output: [1, 2] (not [2])
print(append_item(3))  # Output: [1, 2, 3] (not [3])
```

**Why it’s counter-intuitive**: Many assume that a new empty list is created each time the function is called without `my_list`. Instead, the same list persists across calls since the default `[]` is created once at definition time.

**Workaround**: Use `None` as the default value and create a new list inside the function if needed.
```python
def append_item(item, my_list=None):
    if my_list is None:
        my_list = []
    my_list.append(item)
    return my_list

print(append_item(1))  # Output: [1]
print(append_item(2))  # Output: [2]
print(append_item(3))  # Output: [3]
```

### 12. **Iterators and Generators are One-Shot/Exhaustible**
Iterators (including generators and the results of `map`, `filter`, `zip`, etc.) are designed to be consumed only once. Once they have yielded all their values, they are *exhausted* and cannot be reset or reused.

**Example**:
```python
def count_to_three():
    for i in range(3):
        yield i

my_generator = count_to_three()

# First consumption
print(list(my_generator))  # Output: [0, 1, 2]

# Second consumption attempt
print(list(my_generator))  # Output: [] (Empty list, generator is exhausted)
```

**Why it’s counter-intuitive**: You might expect the generator to behave like a list or tuple and simply return its values again. However, generators save memory by computing values on the fly and discarding the state once the value is returned, making them single-use. The same applies to objects returned by functions like `zip()` or `map()` in Python 3.

**Workaround**: If you need to iterate over the data multiple times, you must either:
1.  Store the result in a reusable data structure (like a `list` or `tuple`).
2.  Recreate the iterator/generator for each use.

```python
# Workaround 1: Store the result
my_list = list(count_to_three())
print(my_list)     # Output: [0, 1, 2]
print(my_list)     # Output: [0, 1, 2] (Reusable)

# Workaround 2: Recreate the generator (best for memory)
print(list(count_to_three())) # Output: [0, 1, 2]
print(list(count_to_three())) # Output: [0, 1, 2]
```

### 13. **Exception Variables are Deleted Outside `except` Block (Python 3)**
In Python 3, the variable bound to the exception in an `except` clause (`except Exception as e:`) is explicitly cleared and deleted once the `except` block is finished. This is to prevent a common memory leak issue.

**Example**:
```python
def check_exception():
    e = None
    try:
        raise ValueError("Something went wrong.")
    except ValueError as e:
        print(f"Inside: {e}")
    # print(e) # Uncommenting this line will cause a NameError

check_exception()
# Outside the function, you cannot access 'e' defined in the except block
```

**Why it’s counter-intuitive**: In many C-like languages and in older Python 2 code, variables declared in a local block often persist after the block finishes. Python 3 deliberately restricts the scope of the exception variable to the `except` block to avoid creating cyclic references between the stack frame and the exception object, which could lead to memory leaks.

**Workaround**: If you need to access the exception object (or information from it) outside of the `except` block, assign it to a variable in the outer scope *inside* the `except` block.

```python
def check_exception_fixed():
    exception_details = None
    try:
        raise ValueError("Something went wrong.")
    except ValueError as e:
        print(f"Inside: {e}")
        # Capture the necessary info
        exception_details = str(e)

    if exception_details:
        print(f"Outside (captured): {exception_details}")

check_exception_fixed()
```

### 14. **Non-Guaranteed String Interning with `is`**
Python (specifically CPython) performs a process called "interning" on some strings for optimization, similar to integer caching. This means identical, short, identifier-like strings might reference the exact same object, leading to surprising results with the identity operator (`is`). However, this is an implementation detail and is not guaranteed for all strings.

**Example**:
```python
# Short, simple strings are often interned
str1 = "python"
str2 = "python"
print(str1 is str2)  # Output: True (Often the case due to interning)

# Strings created via expression or with spaces are less likely to be interned
str3 = "py" + "thon"
str4 = "python"
print(str3 is str4)  # Output: Could be False (depends on implementation)

# Spaces inhibit interning
str5 = "hello world"
str6 = "hello world"
print(str5 is str6)  # Output: Could be False (depends on implementation)
```

**Why it’s counter-intuitive**: Since `str1 == str2` is true, you might expect `str1 is str2` to always be true if the strings are identical, just as it is with small integers. The inconsistency arises because interning is a non-guaranteed CPython optimization designed for performance, not a rule of the language.

**Workaround**: **Always** use the equality operator (`==`) when comparing string *values*. Reserve the identity operator (`is`) only for checking if two variables reference the *exact same* object (e.g., checking for `None`).

```python
str3 = "py" + "thon"
str4 = "python"
print(str3 == str4)  # Output: True (Correct way to check value)
```
