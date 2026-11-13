# Exception Handling and Type Hints

## ১. Exception কী?

Program চলার সময় runtime error বা exceptional condition ঘটলে Python exception raise করে। উপযুক্ত জায়গায় exception handle করা যায়।

```python
try:
    value = int("not-a-number")
except ValueError as error:
    print(f"Invalid integer: {error}")
```

## ২. `try / except / else / finally`

- `try`: যে code exception raise করতে পারে।
- `except`: নির্দিষ্ট exception handle করে।
- `else`: exception না হলে execute হয়।
- `finally`: সাধারণত success বা failure উভয় ক্ষেত্রেই cleanup-এর জন্য execute হয়।

```python
try:
    result = 10 / 2
except ZeroDivisionError:
    print("Cannot divide by zero")
else:
    print("Result:", result)
finally:
    print("Operation finished")
```

## ৩. নির্দিষ্ট Exception ধরবে কেন?

`except Exception:` দিয়ে সব error নির্বিচারে গিলে ফেলা উচিত নয়। এতে bug লুকিয়ে যেতে পারে। যতটা সম্ভব expected exception ধরবে এবং unexpected error propagate হতে দেবে।

খারাপ:

```python
try:
    process_data()
except Exception:
    pass
```

ভালো:

```python
try:
    user_id = int(raw_user_id)
except ValueError as error:
    raise ValueError("user_id must be an integer") from error
```

## ৪. Custom Exception

```python
class DataValidationError(Exception):
    """Raised when input data violates domain rules."""

def validate_age(age: int) -> None:
    if age < 0:
        raise DataValidationError("age cannot be negative")
```

Domain-specific error প্রকাশ করতে custom exception উপকারী।

## ৫. Type Hints কী?

Type Hints function parameter, return value ও variable-এর প্রত্যাশিত type প্রকাশ করে। এগুলো readability, IDE support এবং static analysis উন্নত করে। Python runtime সাধারণভাবে type hints enforce করে না।

```python
def calculate_total(prices: list[float]) -> float:
    return sum(prices)
```

## ৬. Optional এবং Union Type

```python
def find_user(user_id: int) -> str | None:
    if user_id == 1:
        return "Shomi"
    return None
```

`str | None` মানে return value string অথবা None হতে পারে। Python 3.10+ এ এই syntax সমর্থিত।

## ৭. Typed Dictionary এবং Dataclass

```python
from dataclasses import dataclass
from typing import TypedDict

class UserPayload(TypedDict):
    id: int
    name: str

@dataclass
class User:
    id: int
    name: str
```

- `TypedDict` dictionary-shaped data-এর static type description দেয়।
- `dataclass` data container class তৈরির boilerplate কমায়।
- কোনোটিই নিজে থেকে পূর্ণ runtime input validation করে না।

## ৮. Type Hints কি validation?

না। API বা external input validate করতে Pydantic-এর মতো runtime validation library ব্যবহার করা যায়। Type checker যেমন mypy বা pyright development-time error ধরতে সাহায্য করে।

## Practice

1. একটি function লেখো যা string থেকে integer parse করবে এবং invalid input-এ meaningful exception দেবে।
2. Type hints-সহ CSV row parser লেখো।
3. একটি custom exception তৈরি করো।
4. mypy বা pyright দিয়ে ছোট module check করো।
