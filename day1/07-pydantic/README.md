# Pydantic

External system থেকে আসা data-কে application-এর ভিতরে ব্যবহার করার আগে parse ও validate করতে হয়। JSON payload দেখতে ঠিকঠাক হলেও field missing, wrong type, invalid range বা unexpected value থাকতে পারে। **Pydantic** Python type hints ব্যবহার করে runtime data parsing, validation এবং serialization-এর কাজ সহজ করে।

এই chapter Pydantic **v2** syntax ধরে লেখা।

## ১. Pydantic কী?

Pydantic হলো Python library, যা type annotations ও declared constraints ব্যবহার করে data parse ও validate করে। API request body, configuration, event payload এবং external JSON data model করতে এটি উপকারী।

Install:

~~~bash
pip install "pydantic>=2,<3"
~~~

Email validation-এর মতো কিছু type-এর জন্য অতিরিক্ত dependency লাগতে পারে:

~~~bash
pip install email-validator
~~~

## ২. Basic Model কীভাবে তৈরি করব?

`BaseModel` inherit করে model তৈরি করা হয়। Field type এবং `Field()` দিয়ে constraints declare করা যায়।

~~~python
from pydantic import BaseModel, EmailStr, Field

class UserCreate(BaseModel):
    name: str = Field(min_length=2, max_length=80)
    age: int = Field(ge=18, le=120)
    email: EmailStr

user = UserCreate(
    name="Shomi Khan",
    age=32,
    email="shomi@example.com",
)

print(user.name)
print(user.model_dump())
~~~

এখানে Pydantic input parse করে model তৈরি করে এবং field constraint validate করে। `EmailStr` ব্যবহার করতে `email-validator` dependency লাগতে পারে।

~~~mermaid
flowchart TD
    A["Input data"] --> B["UserCreate(...)"]
    B --> C["Parse fields"]
    C --> D["Check types and constraints"]
    D --> E{"Valid?"}
    E -->|হ্যাঁ| F["UserCreate model instance"]
    E -->|না| G["ValidationError"]
~~~

## ৩. Invalid data দিলে কী হয়?

Validation fail হলে Pydantic `ValidationError` raise করে। Error object-এ field location, error type এবং message পাওয়া যায়।

~~~python
from pydantic import ValidationError

try:
    UserCreate(name="A", age=12, email="invalid")
except ValidationError as error:
    print(error)
    print(error.errors())
~~~

Production API-তে validation error user-কে উপযুক্ত client error হিসেবে দেখানো যায়, কিন্তু internal traceback বা sensitive input অযথা expose করা উচিত নয়।

## ৪. Parsing এবং Strict Validation-এর পার্থক্য কী?

Pydantic default mode-এ অনেক input value target type-এ convert করতে পারে। যেমন integer field-এ numeric string কিছু ক্ষেত্রে integer-এ parse হতে পারে। Strict mode চাইলে coercion সীমিত করা যায়।

~~~python
from pydantic import BaseModel, ConfigDict

class StrictUser(BaseModel):
    model_config = ConfigDict(strict=True)

    age: int
~~~

~~~python
StrictUser(age=25)     # valid
# StrictUser(age="25")  # strict validation error
~~~

Strictness পুরো model বা নির্দিষ্ট field-এ configure করা যায়। সব field strict করা সবসময় সঠিক নয়; external API contract অনুযায়ী সিদ্ধান্ত নাও।

## ৫. Field default এবং Optional field কি একই?

না। Type-এ `None` allow করা এবং field omit করা আলাদা বিষয়।

~~~python
from pydantic import BaseModel, Field

class SearchRequest(BaseModel):
    query: str
    limit: int = Field(default=10, ge=1, le=100)
    cursor: str | None
    language: str | None = None
~~~

- `limit` বাদ দিলে default 10 হবে।
- `cursor` None হতে পারে, কিন্তু default নেই বলে field required।
- `language` বাদ দেওয়া যায় এবং বাদ দিলে None হবে।

~~~mermaid
flowchart TD
    A["Field declaration"] --> B{"Default value আছে?"}
    B -->|হ্যাঁ| C["Field omit করা যায়"]
    B -->|না| D["Field required"]
    C --> E{"Type None allow করে?"}
    D --> E
    E -->|হ্যাঁ| F["Explicit None accepted"]
    E -->|না| G["None invalid"]
~~~

## ৬. Custom Field Validator কী?

একটি নির্দিষ্ট field normalize বা validate করতে `@field_validator` ব্যবহার করা হয়।

~~~python
from pydantic import BaseModel, field_validator

class Product(BaseModel):
    sku: str

    @field_validator("sku")
    @classmethod
    def normalize_sku(cls, value: str) -> str:
        value = value.strip().upper()
        if not value:
            raise ValueError("sku cannot be empty")
        return value

product = Product(sku="  ab-123 ")
print(product.sku)  # AB-123
~~~

Validator default mode-এ parsed value পায়। `mode="before"` দিলে raw input-এর উপর validation/normalization করা যায়। Before validator-এ input-এর type সম্পর্কে সতর্ক থাকতে হবে, কারণ raw value প্রত্যাশিত type নাও হতে পারে।

## ৭. Model Validator কখন ব্যবহার করব?

একাধিক field-এর relationship যাচাই করতে `@model_validator` ব্যবহার করা হয়।

~~~python
from pydantic import BaseModel, model_validator

class DateRange(BaseModel):
    start: int
    end: int

    @model_validator(mode="after")
    def validate_range(self):
        if self.end <= self.start:
            raise ValueError("end must be greater than start")
        return self
~~~

Field validator একক field-এর rule-এর জন্য; model validator পুরো model-এর cross-field invariant-এর জন্য।

## ৮. Nested Model কীভাবে তৈরি করব?

একটি model-এর field type হিসেবে আরেকটি model ব্যবহার করা যায়। এতে nested JSON structure-ও validate হয়।

~~~python
from pydantic import BaseModel

class Address(BaseModel):
    city: str
    country: str

class Customer(BaseModel):
    name: str
    address: Address

customer = Customer.model_validate({
    "name": "Mina",
    "address": {
        "city": "Dhaka",
        "country": "Bangladesh",
    },
})
~~~

Nested model ব্যবহার করলে validation rule ছোট ছোট reusable model-এ ভাগ করা যায়।

## ৯. Extra field কীভাবে handle করব?

Input-এ model-এ declare না করা field এলে Pydantic configuration অনুযায়ী ignore, allow বা forbid করতে পারে। API contract কঠোর রাখতে `extra="forbid"` উপকারী।

~~~python
from pydantic import BaseModel, ConfigDict

class CreateOrder(BaseModel):
    model_config = ConfigDict(extra="forbid")

    product_id: int
    quantity: int
~~~

~~~python
# CreateOrder(product_id=10, quantity=2, admin=True)
# ValidationError: extra field is not permitted
~~~

কোন policy ব্যবহার করবে তা API compatibility ও forward-compatibility requirement-এর উপর নির্ভর করে।

## ১০. Serialization এবং Deserialization কীভাবে হয়?

- `model_validate()`: Python object/dictionary থেকে model তৈরি করে।
- `model_validate_json()`: JSON string/bytes থেকে model তৈরি করে।
- `model_dump()`: Python dictionary তৈরি করে।
- `model_dump_json()`: JSON string তৈরি করে।

~~~python
payload = {
    "name": "Shomi Khan",
    "age": 32,
    "email": "shomi@example.com",
}

user = UserCreate.model_validate(payload)
json_text = user.model_dump_json()
restored = UserCreate.model_validate_json(json_text)

print(user.model_dump())
print(json_text)
print(restored)
~~~

~~~mermaid
flowchart LR
    A["Python dict / JSON"] --> B["model_validate / model_validate_json"]
    B --> C["Validated model"]
    C --> D["model_dump"]
    C --> E["model_dump_json"]
    D --> F["Python dict"]
    E --> G["JSON string"]
~~~

Serialization-এ `include`, `exclude`, `exclude_none`, alias এবং custom serializer ব্যবহার করা যায়। Sensitive field serialize করার সময় সচেতন থাকতে হবে।

## ১১. Alias কী?

External JSON key এবং Python attribute-এর নাম আলাদা হলে alias ব্যবহার করা যায়।

~~~python
from pydantic import BaseModel, Field

class UserPayload(BaseModel):
    user_id: int = Field(validation_alias="userId")

user = UserPayload.model_validate({"userId": 7})
print(user.user_id)
~~~

API contract camelCase হলে Python code-এ snake_case বজায় রেখেও alias দিয়ে map করা যায়। Validation alias ও serialization alias আলাদা করে configure করার option-ও আছে।

## ১২. Pydantic কি security boundary?

Pydantic input shape ও constraints validate করতে সাহায্য করে, কিন্তু এটি authorization বা business permission-এর বিকল্প নয়।

এটি নিজে থেকে:
- User-এর resource access permission নিশ্চিত করে না।
- SQL injection প্রতিরোধ করে না।
- Business rule-এর সব semantic condition জানে না।
- External service-এর response সত্য বা trustworthy প্রমাণ করে না।

~~~mermaid
flowchart TD
    A["Untrusted API input"] --> B["Pydantic parsing and validation"]
    B --> C{"Schema valid?"}
    C -->|না| D["Reject invalid request"]
    C -->|হ্যাঁ| E["Authentication / authorization"]
    E --> F{"Allowed?"}
    F -->|না| G["Forbidden"]
    F -->|হ্যাঁ| H["Business rules"]
    H --> I["Database / service operation"]
~~~

Validation, authentication, authorization এবং domain rule আলাদা layer হিসেবে design করো।

## ১৩. FastAPI-তে Pydantic কীভাবে ব্যবহৃত হয়?

FastAPI request body validation এবং OpenAPI schema তৈরিতে Pydantic model ব্যবহার করে।

~~~python
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class PromptRequest(BaseModel):
    prompt: str = Field(min_length=1, max_length=10_000)
    max_tokens: int = Field(default=200, ge=1, le=2000)

@app.post("/generate")
async def generate(request: PromptRequest):
    return {"received": request.prompt}
~~~

FastAPI request body parse করে model validation চালায়। Invalid input হলে default behavior-এ validation response দেয়। Business authorization ও AI provider-এর usage limit আলাদাভাবে enforce করতে হবে।

## ১৪. Pydantic Settings কী?

Application configuration environment variable বা secrets থেকে load করতে `pydantic-settings` ব্যবহার করা হয়। এটি Pydantic-এর আলাদা package।

~~~bash
pip install pydantic-settings
~~~

~~~python
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    database_url: str
    debug: bool = False

    model_config = SettingsConfigDict(env_file=".env")

settings = Settings()
~~~

Production secret source ও deployment environment অনুযায়ী configure করতে হবে। Secret কখনো source control-এ commit কোরো না।

## ১৫. Dataclass, Pydantic এবং TypedDict-এর পার্থক্য কী?

| Tool | মূল কাজ | Runtime validation |
|---|---|---|
| `TypedDict` | Dictionary shape-এর static typing | না |
| `dataclass` | Data container class ও boilerplate কমানো | সাধারণত না |
| `Pydantic BaseModel` | Parsing, validation ও serialization | হ্যাঁ |

Use case অনুযায়ী বেছে নাও: internal data container-এর জন্য dataclass, static dictionary contract-এর জন্য TypedDict, untrusted external input boundary-তে Pydantic।

## Quick Interview Revision

| প্রশ্ন | সংক্ষিপ্ত উত্তর |
|---|---|
| Pydantic কী? | Type hints ভিত্তিক runtime parsing ও validation library |
| BaseModel? | Validated data model |
| model_validate? | Python data থেকে model তৈরি |
| model_validate_json? | JSON থেকে model তৈরি |
| model_dump? | Python dictionary |
| model_dump_json? | JSON string |
| Strict mode? | Coercion সীমিত করে |
| Field validator? | একটি field normalize/validate |
| Model validator? | একাধিক field-এর relation validate |
| Optional type ও optional field? | None allow করা ও field omit করা আলাদা |
| extra='forbid'? | Undeclared field reject |
| Pydantic authorization দেয়? | না |
| TypedDict vs Pydantic? | Static shape typing বনাম runtime validation |
| Pydantic Settings? | Environment/configuration load ও validate |

## Practice Exercises

1. AI chat request-এর জন্য Pydantic model তৈরি করো।
2. `max_tokens` ১ থেকে ২০০০-এর মধ্যে সীমাবদ্ধ করো।
3. Empty prompt reject করো এবং whitespace-only prompt-এর behavior define করো।
4. Strict mode ও default coercion compare করো।
5. `field_validator` দিয়ে prompt trim ও normalize করো।
6. `model_validator` দিয়ে start/end date বা min/max value relationship validate করো।
7. Nested model দিয়ে user profile ও address validate করো।
8. Extra field allow, ignore ও forbid behavior compare করো।
9. Model-কে dictionary ও JSON-এ serialize/deserialize করো।
10. FastAPI endpoint-এ invalid payload পাঠিয়ে validation response observe করো।
11. Pydantic Settings দিয়ে environment variable থেকে configuration load করো।
12. Validation pass হলেও authorization failure হতে পারে এমন একটি scenario design করো।

## Practice করার পদ্ধতি

1. Valid, invalid, missing, extra এবং boundary input test করো।
2. Coercion দরকার নাকি strict input দরকার তা API contract থেকে ঠিক করো।
3. Field-level ও cross-field rule আলাদা করে implement করো।
4. Validation-এর পরে authorization ও business rule কোথায় হবে তা explain করো।
5. Pydantic v2 API ব্যবহার করো; পুরোনো v1 syntax-এর সঙ্গে গুলিয়ে ফেলো না।
