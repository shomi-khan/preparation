# Agents and Tool Calling

একটি সাধারণ LLM application user-এর prompt নিয়ে response দেয়। কিন্তু কিছু task-এর জন্য শুধু answer যথেষ্ট নয়: customer record খুঁজতে হবে, calculation করতে হবে, API call করতে হবে, result দেখে পরবর্তী step ঠিক করতে হবে। এই ধরনের system-এ LLM decision-making component হিসেবে কাজ করতে পারে এবং application-controlled tool ব্যবহার করে task এগিয়ে নেয়।

## ১. AI Agent কী?

AI Agent হলো এমন system যেখানে LLM বা অন্য decision component goal/context দেখে পরবর্তী action নির্বাচন করে এবং tools/environment-এর সঙ্গে interaction করে। সব agent-এ autonomy একই মাত্রার নয়।

একটি common agent loop:

~~~mermaid
flowchart TD
    A["Observe request / state"] --> B["Decide next action"]
    B --> C{"Need a tool?"}
    C -->|হ্যাঁ| D["Propose tool call"]
    D --> E["Application validates and executes"]
    E --> F["Observe tool result"]
    F --> B
    C -->|না| G["Return final response"]
~~~

## ২. LLM Application বনাম Agent-এর পার্থক্য কী?

- **Simple LLM app:** নির্দিষ্ট input → model call → output।
- **Agentic system:** observe → decide → tool/action → observe result → continue/stop।

Agent-এর autonomy মানে unrestricted permission নয়। Application-ই tool, data এবং action-এর সীমা নির্ধারণ করে।

~~~mermaid
flowchart LR
    A["User input"] --> B["Simple LLM call"]
    B --> C["One response"]
    D["User goal"] --> E["Agent decision loop"]
    E --> F["Tool execution"]
    F --> G["Observe result"]
    G --> E
~~~

Loop প্রয়োজন না হলে simple deterministic workflow-ই ভালো। Agentic complexity latency, cost ও failure mode বাড়ায়।

## ৩. Tool Calling কী?

Model একটি নির্দিষ্ট tool call প্রস্তাব করতে পারে, যেমন `search_customer(customer_id)`। Application arguments validate ও authorization check করে tool execute করে এবং result আবার model-এ ফেরত দেয়।

~~~mermaid
sequenceDiagram
    participant U as User
    participant M as Model
    participant A as Application
    participant T as Tool
    U->>A: Request
    A->>M: Context + tool definitions
    M-->>A: Tool call proposal
    A->>A: Validate schema and permission
    A->>T: Execute allowed handler
    T-->>A: Tool result
    A->>M: Tool result
    M-->>A: Final answer or next tool call
    A-->>U: Response
~~~

Model-এর tool call proposal নিজে থেকে execution নয়। Application-এর control boundary অতিক্রম করে tool run হবে কি না, তা application নির্ধারণ করে।

## ৪. Tool Schema কেন দরকার?

Tool input explicit schema-তে define করলে invalid argument reject করা যায়।

~~~python
from pydantic import BaseModel, Field

class CustomerLookup(BaseModel):
    customer_id: int = Field(gt=0)
~~~

Schema-তে:
- Required/optional field।
- Type ও range constraint।
- Enum/allowed value।
- Maximum string/list size।
- Unknown field policy।

Schema validation authorization-এর বিকল্প নয়। Valid customer_id হলেও current user সেই customer-এর record দেখতে পারবে কি না, আলাদা check দরকার।

## ৫. Tool Registry কী?

Application-এর known tool name, schema ও handler registry থাকে। Model arbitrary Python function বা shell command execute করতে পারবে না।

~~~python
TOOLS = {
    "lookup_customer": {
        "schema": CustomerLookup,
        "handler": lookup_customer,
        "permission": "customer:read",
    }
}
~~~

এটি conceptual example। Production registry-তে tool description, timeout, side-effect classification, permission এবং audit metadata-ও রাখা যেতে পারে।

## ৬. Tool Execution-এর নিরাপদ flow কী?

~~~mermaid
flowchart TD
    A["Model proposes tool call"] --> B["Tool name allowlisted?"]
    B -->|না| C["Reject"]
    B -->|হ্যাঁ| D["Validate arguments"]
    D --> E{"Schema valid?"}
    E -->|না| C
    E -->|হ্যাঁ| F["Authorize current user"]
    F --> G{"Permission granted?"}
    G -->|না| C
    G -->|হ্যাঁ| H["Check approval / idempotency"]
    H --> I["Execute with timeout"]
    I --> J["Return bounded result"]
~~~

Tool output-ও untrusted হতে পারে। Application-এর error, database row বা external API response model-এ ফেরত দেওয়ার আগে প্রয়োজন অনুযায়ী sanitize ও bound করো।

## ৭. Tool Design Principles কী?

- **Single-purpose:** একটি tool একটি স্পষ্ট কাজ করবে।
- **Explicit schema:** input/output contract স্পষ্ট।
- **Least privilege:** প্রয়োজনীয় সর্বনিম্ন permission।
- **Bounded output:** বড় data dump নয়; relevant summary।
- **Timeout:** অনির্দিষ্টকাল block নয়।
- **Idempotency:** retry-তে duplicate side effect নয়।
- **Audit logging:** কে, কখন, কোন action execute করেছে।
- **Clear error contract:** retryable ও permanent failure আলাদা।
- **Rate/concurrency limit:** abuse ও overload নিয়ন্ত্রণ।

## ৮. Read-only Database Tool কীভাবে design করব?

Natural language থেকে SQL generate করলেও direct unrestricted DB access দেওয়া যাবে না।

Controls:
- Read-only database credential।
- Allowlisted schema/table।
- Query validation; SQL parser/AST validation regex-এর চেয়ে নির্ভরযোগ্য হতে পারে।
- Parameterized values।
- Statement timeout।
- Row/result size limit।
- Tenant/user authorization।
- Sensitive column filtering।
- Audit log।

Read-only access-ও sensitive data exfiltration ঘটাতে পারে। তাই least privilege ও row-level authorization জরুরি।

## ৯. Side-effecting Tool কী? Idempotency কেন দরকার?

Read-only tool data retrieve করে; side-effecting tool system state বদলায়, যেমন email send, payment initiate, account update বা record delete।

Timeout বা retry-তে একই logical action একাধিকবার execute হতে পারে। Idempotency key/deduplication দিয়ে duplicate action আটকাও।

~~~mermaid
flowchart TD
    A["Action request + idempotency key"] --> B{"Key already processed?"}
    B -->|হ্যাঁ| C["Return stored result"]
    B -->|না| D["Validate and authorize"]
    D --> E["Execute action"]
    E --> F["Persist result with key"]
    F --> G["Return result"]
~~~

Idempotency key unique হওয়া, correct scope-এ রাখা এবং result persistence atomic হওয়া গুরুত্বপূর্ণ।

## ১০. Human-in-the-loop কখন দরকার?

Money transfer, account update, email send বা delete-এর মতো consequential action-এর আগে explicit confirmation প্রয়োজন হতে পারে।

~~~mermaid
flowchart TD
    A["Agent proposes action"] --> B["Policy / permission check"]
    B --> C{"High impact or irreversible?"}
    C -->|না| D["Execute within policy"]
    C -->|হ্যাঁ| E["Show exact action and consequences"]
    E --> F{"User confirms?"}
    F -->|হ্যাঁ| G["Execute once"]
    F -->|না| H["Cancel / revise"]
~~~

Confirmation-এ action, target, amount/content এবং consequences স্পষ্ট দেখাও। Model-এর নিজস্ব confidence confirmation-এর বিকল্প নয়। Confirmation state server-side verify করো।

## ১১. Tool Error কীভাবে handle করব?

Tool result structured হওয়া উচিত, যেমন:
- success + bounded data।
- validation_error।
- permission_denied।
- not_found।
- rate_limited।
- timeout।
- temporary_unavailable।

Agent retry করবে কি না error type, retry budget এবং idempotency দেখে decide করো। একই failing tool বারবার call করতে দেওয়া যাবে না।

## ১২. Tool Calling এবং MCP কি একই জিনিস?

না। Tool/function calling হলো model/application interaction pattern: model tool call propose করে, application execute করে। MCP (Model Context Protocol) হলো model application-কে tools/resources/prompts expose করার protocol। MCP server-ও tool expose করতে পারে, কিন্তু tool calling concept MCP-এর বাইরেও ব্যবহার করা যায়।

## Quick Interview Revision

| প্রশ্ন | সংক্ষিপ্ত উত্তর |
|---|---|
| AI Agent? | Goal/context দেখে action নির্বাচন ও tool interaction করা system |
| Agent loop? | Observe → decide → act → observe |
| Tool calling? | Model proposal, application-controlled execution |
| Tool schema? | Structured input contract |
| Tool registry? | Allowed tool, schema ও handler mapping |
| Authorization কোথায়? | Application/server-side |
| Least privilege? | প্রয়োজনীয় minimum permission |
| Idempotency? | Retry-তে duplicate side effect আটকানো |
| Human-in-the-loop? | Sensitive action-এর আগে explicit approval |
| Tool error? | Structured, classified, bounded retry |
| MCP = tool calling? | না; MCP protocol, tool calling interaction pattern |

## Practice

1. Calculator ও read-only customer lookup tool তৈরি করো।
2. Tool schema Pydantic দিয়ে validate করো।
3. Unauthorized customer lookup reject করো।
4. Unknown tool name reject করো।
5. Tool output size limit করো।
6. Side-effecting tool-এ idempotency key implement করো।
7. High-impact action-এর জন্য explicit confirmation flow তৈরি করো।
8. Timeout, rate limit ও permission error আলাদাভাবে test করো।
9. Agent loop-এ max tool call ও total time budget রাখো।
10. Tool proposal, authorization decision ও execution audit log করো।

## Practice করার পদ্ধতি

1. Model-এর proposal-কে untrusted request হিসেবে ধরো।
2. Schema validation ও authorization আলাদা রাখো।
3. Read-only এবং side-effecting tool আলাদা classify করো।
4. Tool call count, time ও output size bound করো।
5. High-impact action-এ user confirmation server-side enforce করো।
