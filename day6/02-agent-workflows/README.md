# Agent Workflows, State and Safety

একটি agent যত বেশি tool call ও decision নেয়, তার state, failure recovery এবং safety তত গুরুত্বপূর্ণ হয়ে ওঠে। Production system-এ সবকিছু model-এর উপর ছেড়ে দেওয়া উচিত নয়। Deterministic workflow যেখানে সম্ভব সেখানে ব্যবহার করো; model decision কেবল যেখানে ambiguity বা flexible reasoning দরকার সেখানে রাখো।

## ১. Workflow এবং Agent-এর পার্থক্য কী?

- **Workflow:** steps ও transitions আগে থেকে নির্ধারিত।
- **Agent:** context দেখে dynamically next action নির্বাচন করতে পারে।

~~~mermaid
flowchart TD
    A["Task"] --> B["Workflow"]
    A --> C["Agent"]
    B --> D["Predefined steps and branches"]
    C --> E["Model selects next action"]
    D --> F["Predictable execution"]
    E --> G["Flexible but less predictable"]
~~~

Production-এ deterministic workflow ও সীমিত agent decision combine করা প্রায়ই ভালো। Agent loop দরকার না হলে simple workflow-ই রাখো।

## ২. Planning এবং Execution কীভাবে আলাদা?

Agent task-কে subtask-এ ভেঙে plan করতে পারে। কিন্তু plan ভুল বা stale হতে পারে; প্রতিটি action-এর আগে policy ও current state যাচাই করতে হবে।

~~~mermaid
flowchart TD
    A["User goal"] --> B["Create plan"]
    B --> C["Validate plan against policy"]
    C --> D["Execute next step"]
    D --> E["Observe result"]
    E --> F{"Plan still valid?"}
    F -->|হ্যাঁ| G{"Task complete?"}
    G -->|না| D
    G -->|হ্যাঁ| H["Return result"]
    F -->|না| B
~~~

Plan নিজেই permission নয়। Plan তৈরি হয়েছে বলে সব action execute করা যাবে না।

## ৩. Agent State-এ কী রাখা উচিত?

State-এ থাকতে পারে:
- Original user request ও authenticated identity reference।
- Current task/step।
- Validated plan।
- Intermediate result।
- Tool call/result summary।
- Retry count ও budget।
- Approval status।
- Error/status।

State schema explicit ও versioned রাখো। Raw conversation dump-কে একমাত্র source of truth বানিও না।

## ৪. State Machine কেন দরকার?

Task-এর lifecycle state হিসেবে model করলে invalid transition আটকানো যায়।

~~~mermaid
stateDiagram-v2
    [*] --> Pending
    Pending --> Running
    Running --> WaitingForApproval
    WaitingForApproval --> Running: Approved
    WaitingForApproval --> Cancelled: Rejected
    Running --> Completed
    Running --> Failed
    Running --> Cancelled
    Completed --> [*]
    Failed --> [*]
    Cancelled --> [*]
~~~

যেমন WaitingForApproval state-এ থাকা অবস্থায় approval ছাড়া side-effecting tool execute হওয়া উচিত নয়।

## ৫. Checkpoint এবং Durable State কেন দরকার?

Long-running task-এ process restart, provider timeout বা worker crash হতে পারে। Checkpoint-এ progress persist করলে resume করা যায়।

Persist করো:
- Task ID ও state version।
- Completed step ও validated outputs।
- Pending action/approval reference।
- Retry/budget counters।
- Idempotency key।

Sensitive content encrypt, access-control ও retention policy মেনে store করো। Checkpoint-এ secret বা unnecessary PII রাখবে না।

## ৬. Human-in-the-loop Workflow কীভাবে কাজ করে?

Sensitive action-এর আগে workflow pause করে approval চাইতে পারে।

~~~mermaid
flowchart TD
    A["Agent prepares action"] --> B["Validate policy and arguments"]
    B --> C["Persist pending approval"]
    C --> D["Show exact action to user"]
    D --> E{"Explicit confirmation?"}
    E -->|হ্যাঁ| F["Re-check permission and state"]
    F --> G["Execute idempotently"]
    E -->|না| H["Cancel action"]
~~~

Approval stale হতে পারে। User confirm করার সময় target resource বা amount বদলে গেলে execute করার আগে state আবার verify করো। Confirmation specific action-এর জন্য bind হওয়া উচিত।

## ৭. Memory কী? Short-term এবং Long-term Memory-এর পার্থক্য কী?

- **Short-term memory:** current task/session-এর context ও intermediate state।
- **Long-term memory:** sessions-এর বাইরে persist করা user/project knowledge।

Memory-কে unquestionable truth হিসেবে ধরো না। Source, freshness, user permission, retention, privacy এবং deletion policy দরকার। Memory write-ও controlled action হওয়া উচিত।

## ৮. Multi-agent কখন ব্যবহার করব?

একাধিক specialized agent collaborate করতে পারে, কিন্তু coordination, duplicated work, latency, cost ও security complexity বাড়ে।

Multi-agent বিবেচনা করো যখন:
- Task genuinely parallelizable।
- Specialized roles measurable benefit দেয়।
- Shared state ও handoff contract স্পষ্ট।
- Coordination overhead acceptable।

Single agent/workflow যথেষ্ট হলে multi-agent ব্যবহার কোরো না।

~~~mermaid
flowchart TD
    A["Complex task"] --> B{"Can independent subtasks run in parallel?"}
    B -->|না| C["Single agent / workflow"]
    B -->|হ্যাঁ| D["Specialized workers"]
    D --> E["Coordinator validates outputs"]
    E --> F["Combine results"]
~~~

## ৯. LangChain এবং LangGraph কী?

- **LangChain:** model/tool integration ও common LLM application building blocks দেয়।
- **LangGraph:** stateful graph/loop-based agent workflow orchestrate করতে পারে।

Framework concept বোঝার বিকল্প নয়। Raw tool-call loop দিয়েও basic agent তৈরি করা যায়। Framework বেছে নাও persistence, streaming, human approval, observability এবং complexity requirement দেখে।

## ১০. Termination Conditions কী কী?

Agent loop-এর hard stop দরকার:
- Task complete।
- Max step/tool call reached।
- No valid tool/action।
- Token/cost budget exhausted।
- Timeout/cancellation।
- Human approval required।
- Repeated identical failure।

~~~mermaid
flowchart TD
    A["Agent step"] --> B{"Stop condition reached?"}
    B -->|না| C["Continue with next decision"]
    C --> A
    B -->|হ্যাঁ| D["Stop safely"]
~~~

Infinite loop ও runaway cost আটকাতে max iteration, wall-clock timeout এবং token/tool budget enforce করো।

## ১১. Error Handling এবং Retry কীভাবে design করব?

Tool error structuredভাবে return করা যায়, যাতে agent retry বা graceful failure সিদ্ধান্ত নিতে পারে।

- Validation/permission error: retry নয়।
- Not found: query refine বা user clarification।
- Rate limit/transient error: bounded backoff।
- Timeout on side effect: idempotency status check।
- Repeated same failure: stop/escalate।

একই failing tool বারবার call করতে দেওয়া যাবে না।

## ১২. Prompt Injection ও Tool Abuse কীভাবে handle করবে?

Retrieved content বা user input malicious instruction দিতে পারে। Tool allowlist, permission checks, argument validation, output sanitization ও approval boundary রাখতে হবে।

Agent-এর plan বা memory থেকে permission inherit করা যাবে না। প্রতিটি tool execution-এর সময় current user/context অনুযায়ী authorization check করো।

## ১৩. Auditability কীভাবে নিশ্চিত করবে?

Trace করো:
- কোন model/version decision দিয়েছে।
- কোন tool call propose হয়েছে।
- Application কী validation/authorization করেছে।
- কী execute হয়েছে।
- Result/status কী ছিল।
- Approval কে এবং কখন দিয়েছে।

Sensitive data redaction ও access control জরুরি।

## ১৪. Agent Evaluation কীভাবে করব?

শুধু final answer নয়, intermediate behavior-ও evaluate করো:
- Task completion rate।
- Tool selection accuracy।
- Argument validity।
- Unnecessary tool call।
- Policy violation।
- Human approval compliance।
- Retry/loop behavior।
- Latency ও cost।
- Recovery from tool failure।

## Quick Interview Revision

| প্রশ্ন | সংক্ষিপ্ত উত্তর |
|---|---|
| Workflow vs Agent? | Predefined steps vs dynamic action selection |
| Planning কি permission? | না |
| State-এ কী রাখবে? | Task, step, outputs, retries, approval |
| State machine কেন? | Valid transition enforce |
| Checkpoint? | Crash/restart-এর পর resume |
| Human approval? | Specific consequential action authorize |
| Memory? | Session বা cross-session context |
| Multi-agent trade-off? | Parallel specialization বনাম coordination cost |
| Termination? | Max steps/time/cost, completion, approval |
| Agent evaluation? | Completion + tool behavior + safety + operations |

## Practice

একটি bounded workflow বানাও:

~~~mermaid
flowchart TD
    A["User question"] --> B["Validate request"]
    B --> C["Read-only customer lookup"]
    C --> D["Summarize result"]
    D --> E["Return answer"]
~~~

Requirements:
1. Max ৩টি tool call।
2. Read-only access।
3. Structured error handling।
4. State ও step count persist।
5. Timeout ও total budget।
6. Tool failure হলে bounded retry।
7. Unauthorized request reject।
8. একই request replay করলে duplicate side effect না হয়।
9. Agent loop termination test।
10. Tool selection ও final task completion evaluate।

## Practice করার পদ্ধতি

1. Workflow step এবং agent decision আলাদা করো।
2. State schema versioned রাখো।
3. Approval-এর আগে/পরে state persist করো।
4. প্রতিটি tool call-এর আগে permission পুনরায় verify করো।
5. Max steps, time ও cost budget enforce করো।
6. Failure, restart, duplicate event ও cancellation test করো।
