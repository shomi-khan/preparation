# Day 6 Practical Exercises

এই chapter-এ একটি bounded tool-calling agent এবং তার production boundary implement করবে। লক্ষ্য autonomy বাড়ানো নয়; model-এর decision-কে application-controlled, auditable ও safe রাখা।

## Exercise 1: Tool Calling

Calculator এবং read-only customer lookup tool বানাও।

Requirements:
- Pydantic দিয়ে arguments validate।
- Tool registry-তে allowed tool define।
- Unknown tool name reject।
- Output size limit।
- Tool timeout ও structured error।

## Exercise 2: Bounded Agent Loop

Agent-কে সর্বোচ্চ ৪টি tool call করতে দাও।

~~~mermaid
flowchart TD
    A["User goal"] --> B["Agent decision"]
    B --> C{"Tool call budget left?"}
    C -->|না| D["Stop with budget-exhausted status"]
    C -->|হ্যাঁ| E["Validate and execute tool"]
    E --> F["Observe result"]
    F --> G{"Task complete?"}
    G -->|না| B
    G -->|হ্যাঁ| H["Final response"]
~~~

Step limit, total timeout, token budget ও explicit stop condition যোগ করো। Repeated identical tool call detect করো।

## Exercise 3: SQL Safety

Natural language question থেকে SELECT query তৈরি করো। Enforce করো:
- Read-only DB user।
- Single statement।
- Allowlisted tables/columns।
- Statement timeout।
- Maximum row count।
- No DDL/DML।
- Tenant/row-level authorization।
- Parameterized values।

Regex-only SQL validation-এর limitation explain করো।

## Exercise 4: Human Approval

একটি mock refund tool যোগ করো। Refund execution-এর আগে user confirmation ছাড়া tool execute হবে না।

Requirements:
- Exact amount, target account ও reason user-কে দেখাও।
- Approval একটি specific action payload-এর সঙ্গে bind করো।
- Approval pending অবস্থায় state persist করো।
- Confirmation-এর সময় permission ও resource state আবার verify করো।
- Duplicate confirmation-এ duplicate refund যেন না হয়।

~~~mermaid
stateDiagram-v2
    [*] --> Proposed
    Proposed --> PendingApproval
    PendingApproval --> Approved
    PendingApproval --> Rejected
    Approved --> Executed
    Approved --> Cancelled: State changed / permission revoked
    Executed --> [*]
    Rejected --> [*]
    Cancelled --> [*]
~~~

## Exercise 5: Failure Handling

Tool timeout, invalid arguments, permission denied ও provider rate limit simulate করো।

- Retryable/non-retryable error আলাদা করো।
- Exponential backoff/jitter।
- Max retry count।
- Side effect-এ idempotency key।
- Circuit breaker behavior (optional)।

## Exercise 6: Durable State and Checkpoint

Agent workflow-এর state persist করো:
- Task ID।
- Current step।
- Completed step result।
- Tool call count।
- Pending approval ID।
- Retry/budget count।
- State version।

Process restart simulate করে checkpoint থেকে resume করো। Duplicate side effect না হয় তা test করো।

## Exercise 7: Observability

প্রতিটি request-এর correlation ID, model, prompt version, tool call count, latency ও token usage record করো। Sensitive prompt content, API key বা personal data log করবে না।

Track:
- Task completion।
- Tool selection/argument validity।
- Policy rejection।
- Retry count।
- Cost per successful task।

## Exercise 8: Docker

একটি FastAPI application containerize করো।

- Non-root user।
- Health endpoint।
- Environment-based configuration।
- Pinned dependencies।
- Runtime secret injection।
- Graceful shutdown।
- Resource limit documentation।

## Exercise 9: Agent Evaluation

কমপক্ষে ১৫টি scenario তৈরি করো:
- Happy path।
- Unknown tool।
- Invalid argument।
- Unauthorized resource।
- Tool timeout।
- Repeated tool failure।
- Prompt injection in tool output।
- Approval rejected।
- Approval stale।
- Duplicate action request।

প্রতিটি scenario-তে expected tool sequence ও allowed side effect define করো।

## Deliverables

- Runnable tool-calling agent।
- Tool schema ও permission tests।
- State diagram ও architecture diagram।
- Failure/observability test।
- Dockerized application।
- Agent evaluation report।
- Security limitation note।

## Self-review

- [ ] Model arbitrary code execute করতে পারে না।
- [ ] Tool arguments validate করি।
- [ ] Server-side authorization আছে।
- [ ] Step/time/token budget limit আছে।
- [ ] Irreversible action-এ explicit confirmation আছে।
- [ ] Approval payload-bound ও state-checked।
- [ ] Retry bounded এবং idempotent।
- [ ] State/checkpoint persist করি।
- [ ] Logs-এ sensitive data নেই।
- [ ] Cost ও latency measure করি।
- [ ] Prompt injection ও tool output injection test করেছি।

## Completion Criteria

1. Unknown/invalid/unauthorized tool call reject হয়।
2. Agent loop budget-এর মধ্যে থামে।
3. Approval ছাড়া sensitive action execute হয় না।
4. Retry-তে duplicate side effect হয় না।
5. Process restart-এর পর safe resume হয়।
6. Tool trajectory ও task completion evaluate করা যায়।
7. Dockerized app non-root user-এ run করে।
