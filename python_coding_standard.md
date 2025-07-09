# Python Coding Standards  
<div align="right">  
2025.07.04, Version 1.1  
</div>  

---

## 1. Basic Philosophy

- **Clarity First**: "Readable code" is the most important value.  
- **Immutability Principle**: Avoid changing variable values after initial assignment whenever possible.  
- **Simplicity Priority**: Prefer simple structures over complex ones; use abstraction only when necessary by design.  
- **Functional Thinking**: Design functions to be pure as much as possible and minimize state changes.  

---

## 2. Function and Module Design Guidelines

### 2.1 Function Rules
- Functions without I/O must be pure functions (always return the same result for the same input).  
- No modification of global state inside functions.  
- Functions should be small and follow the Single Responsibility Principle (SRP).  

### 2.2 Function Structure Example

```python
# Pure function example
def calculate_total(prices):
    sum_prices = sum(prices)
    return sum_prices
```

### 2.3 I/O Functions

- Functions that must perform state changes like file or DB access should be separated.  
- Output/logging is treated as an "observable side effect" and must be clearly separated.  

### 2.4 Strengthening Function Immutability

- Prefer tuples over lists where possible.  
- Copy external input values into internal variables after processing.

```python
def some_func(list_a):
    list_b = copy(list_a)
    ...
    return list_b
```

- Avoid mutable default arguments (`def func(arg=[])` ❌).  

---

## 3. Variable Usage Principles

- Avoid reassigning the same variable name; define new names to maintain immutability.

**Forbidden code ❌**

```python
some_var = 5
some_var = some_func(some_var)
```

**Recommended usage**

```python
some_var = 5
new_some_var = some_func(some_var)
```

- Exception: loop index variables (`for i in ...`) are allowed.  

- **Use `dict.get()` for dictionary initialization and lookup**

```python
# Dictionary initialization
user = {"name": "Alice", "age": 30}

# Using dict.get()
age = user.get("age", 0)  # returns 0 if "age" key is missing
```

---

## 4. Class and Object Usage Principles

- Use classes only for clearly defined UI objects or when members and methods are explicitly defined.  
  - Example: Button, JSONValidator, etc.  
- Use `dict`, `TypedDict`, or `dataclass` for simple data transfer.

**Example: Using dict instead of class**

```python
user = {"id": 1, "name": "Alice"}  # Designed for validation based on JSON schema
```

---

## 5. Data Structure and Validation

- Use JSON-based structures as the standard (especially for APIs and internal data transfer).  
- Use libraries like `jsonschema` for automatic structure validation.

```python
from sapgen.json_pg_table import *
from jsonschema import validate

schema = {
  "type": "object",
  "properties": {
    "empno": {"type": "integer"},
    "name": {"type": "string"},
    "age": {"type": "integer"},
    "salary":{"type":"integer"}
  },
  "required": ["name", "age"],
  "key" : ["empno"],
  "validation": {
      "empno": {"min": 1, "max": 5000}, 
      "salary": {"min": 3000, "max": 100000}
  },
}

validate({"name": "Alice", "age": 30}, schema)
```

- Use `JSONInternalTable` class for data handling.

(Additional usage examples omitted for brevity, but can be included if needed.)

---

## 6. Type Hints and Comments

### 6.1 Type Hints
- Use only when they aid IDE autocomplete and collaboration.  
- Omit when trivial or obvious.

```python
# Only complex or external API functions should specify types
def parse_user(data):
    ...
```

### 6.2 Comments
- Write comments explaining "what" and "why," not "how."  
- Prefer block comments over inline comments inside functions.

```python
# Handle edge case when input is zero (prevent division by zero)
if x == 0:
    return None
```

### 6.3 Docstrings
- Write docstrings for every function, class, and module.  
- Place at the top of each block.

#### 6.3.1 Function Docstring Example

```python
def example_function(param1, param2="default"):
    """
    Function type: Pure / Impure
        - If impure, state the reason for external state changes.

    Brief description of what the function does.

    Detailed description of function behavior and return value.

    Args:
        param1 (type): Description of first parameter
        param2 (type): Second parameter, default is "default"

    Returns:
        Description of return value and type

    Raises:
        ValueError: Description of possible exceptions
    """
    ...
```

#### 6.3.2 Class Docstring Example

```python
class ExampleClass:
    """
    Brief class description.

    Detailed explanation about the class and usage.

    Methods:
        method1: Brief description of method1
        method2: Brief description of method2

    Attributes:
        attr1 (type): Description of attribute 1
        attr2 (type): Description of attribute 2
    """

    def __init__(self, attr1, attr2):
        """
        Constructor description.

        Args:
            attr1 (type): Description of attr1
            attr2 (type): Description of attr2
        """
        self.attr1 = attr1
        self.attr2 = attr2
```

#### 6.3.3 Module Docstring Example

```python
"""
Brief description of the module functions and classes.

Functions:
    func1(): Short description of func1's role
    func2(): Short description of func2's role

Classes:
    class1(): Brief description of class1
"""
...
```

---

## 7. Code Style (PEP8 compliance)

- Max 88 characters per line (Black formatter standard)  
- 4 spaces indentation (no tabs)  
- Strict distinction between `is` and `==`  
- No unnecessary spaces

**Recommended automation tools**

- `black` formatter (install VS Code extension for convenience)

---

## 8. Testing and Exception Handling

- Aim to minimize exception handling by pre-checking conditions.  
- Use conditional checks before `try-except` blocks.  
- Use default value methods like `dict.get()` to avoid `KeyError`.  
- Focus exception handling on complex external I/O or system calls.  
- Always catch specific exception classes.  
- Never leave empty except blocks or ignore exceptions.  
- Use test coverage to verify all exception scenarios.

```python
# Pre-check condition to avoid exception
if os.path.exists(filepath):
    with open(filepath) as f:
        data = f.read()
else:
    logger.warning("File does not exist")

# Use dict.get() for default values
value = config.get("timeout", 30)

# Necessary exception handling
try:
    result = external_api_call()
except ConnectionError as e:
    logger.error(f"Network error: {e}")
    raise
```

---

## 9. Logging and Output Rules

- Use `print()` only for shell testing and debugging.  
- Use `logging` in scripts or production code.

**Logging level guide**

- `logger.debug()`: Detailed info for development and debugging  
- `logger.info()`: Normal flow logs (start, completion, etc.)  
- `logger.warning()`: Recoverable exceptions, missing info, fallback  
- `logger.error()`: Failure situations (exceptions, errors)

```python
import logging
logger = logging.getLogger(__name__)
logging.basicConfig(level=logging.INFO)
```

---

## 10. Dynamic SQL Generation and Management

### 10.1 SQL Generation Guidelines

- Manage SQL as template strings, not plain strings.  
- Separate fields/conditions and assemble dynamically.  
- Use parameter binding to prevent SQL Injection.

```python
base_sql = "SELECT * FROM users WHERE 1=1"
params = {}
if name:
    base_sql += " AND name = %(name)s"
    params["name"] = name
if age:
    base_sql += " AND age >= %(age)s"
    params["age"] = age

cur.execute(base_sql, params)
```

### 10.2 SQL Management

- For long or complex SQL, separate into `.sql` files for version control.  
- Document SQL with sample parameters for testing.  
- ORM use is prohibited; write and maintain SQL directly.

### 10.3 SQL Template Management

```python
with open("queries/get_user.sql") as f:
    sql_template = f.read()

sql = sql_template.replace("{{id}}", str(user_id))
cur.execute(sql)
```

---

## 11. List Comprehension Usage Standards

- Prefer list comprehensions over `map`, `filter`, `reduce` for readability.  
- Use `map`, `filter`, `reduce` only for complex function compositions or when using external libraries.  
- Prefer list comprehension over loops to generate new lists.  
- Avoid more than 4 levels of nested comprehensions due to readability loss.  
- However, in pure functional code, deeper nesting (5 or 6 levels) is allowed due to clear input/output.  
- Use only for pure transformations without internal state changes.

```python
from functools import reduce

nums = [1, 2, 3, 4]

# map example
mapped = list(map(lambda x: x * 2, nums))
mapped_lc = [x * 2 for x in nums]

# filter example
filtered = list(filter(lambda x: x % 2 == 0, nums))
filtered_lc = [x for x in nums if x % 2 == 0]

# reduce example
total = reduce(lambda acc, x: acc + x, nums, 0)
total_lc = sum(nums)
```

### Nested List Comprehension Examples

```python
# Recommended
squares = [x * x for x in range(10)]

# Allowed
pairs = [(x, y) for x in range(3) for y in range(3) if x != y]

# Not recommended


# [(a,b,c,d) for a in A for b in B for c in C for d in D if valid(a,b,c,d)]
```

### If using non-recommended nested comprehensions, maintain readability

```python
list_comp = [
    (a,b,c,d) 
    for a in A 
    for b in B 
    for c in C 
    for d in D 
    if valid(a,b,c,d)
]
```
