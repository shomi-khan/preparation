# Agents and Tool Calling

## ১. AI Agent কী?
AI Agent হলো এমন system যেখানে LLM বা অন্য decision component goal/context দেখে পরবর্তী action নির্বাচন করে এবং tools/environment-এর সঙ্গে interaction করে। সব agent-এ autonomy একই মাত্রার নয়।

## ২. LLM Application বনাম Agent
- সাধারণ LLM app: নির্দিষ্ট input -> model call -> output।
- Agentic system: observe -> decide -> tool/action -> observe result -> continue/stop।
Agent loop প্রয়োজন না হলে simple workflow-ই ভালো।

## ৩. Tool Calling কী?
Model structured tool call propose করে, যেমন `search_customer(customer_id)`। Application arguments validate ও authorization check করে tool execute করে এবং result model-এ ফেরত দেয়।

## ৪. Tool Schema
```python
from pydantic import BaseModel, Field

class CustomerLookup(BaseModel):
    customer_id: int = Field(gt=0)
```
Tool input strict schema-তে validate করো। Tool-এর permission user ও request context অনুযায়ী server-side enforce করতে হবে।

## ৫. Tool Registry
Application-এর known tool name, schema ও handler registry থাকে। Model arbitrary Python function বা shell command execute করতে পারবে না।

## ৬. Tool Calling Cycle
```text
User Request
 -> LLM chooses tool
 -> Application validates arguments
 -> Authorization check
 -> Tool executes
 -> Tool result returned to LLM
 -> LLM answers or selects next action
```

## ৭. Tool Design Principles
- Single-purpose।
- Explicit schema।
- Least privilege।
- Bounded output।
- Timeout।
- Idempotency where needed।
- Audit logging।
- Clear error contract।

## ৮. Read-only Database Tool
Natural language থেকে SQL generate করলেও direct unrestricted DB access দেওয়া যাবে না। Read-only credential, allowlisted tables, query validation, statement timeout, row limit ও parameterized values দরকার। SQL parser/AST validation regex-এর চেয়ে নির্ভরযোগ্য হতে পারে।

## ৯. Human-in-the-loop
Money transfer, account update, email send বা delete-এর মতো consequential action-এর আগে explicit confirmation প্রয়োজন হতে পারে। Model-এর নিজস্ব confidence confirmation-এর বিকল্প নয়।

## Practice
একটি calculator ও read-only customer lookup tool তৈরি করো। Tool schema validate করো এবং unauthorized customer lookup reject করো।
