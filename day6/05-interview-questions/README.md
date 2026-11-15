# Day 6 Interview Questions and Answers

## Agent Fundamentals

### ১. AI Agent কী?
Goal/context অনুযায়ী action নির্বাচন করে tools/environment-এর সঙ্গে interaction করা system। Autonomy ও capability implementation অনুযায়ী বদলায়।

### ২. LLM application ও agent-এর পার্থক্য?
Simple LLM app সাধারণত fixed call-response flow; agent loop-এ observation, decision, tool execution ও iteration থাকে।

### ৩. Tool calling কী?
Model structured tool call propose করে; application validate ও authorize করে function execute করে এবং result model-এ ফেরত দেয়।

### ৪. Model কি সরাসরি tool execute করে?
সাধারণ architecture-এ না। Model call propose করে; application-side orchestrator actual execution ও permission control করে।

### ৫. Workflow ও agent-এর পার্থক্য?
Workflow predetermined steps follow করে; agent runtime-এ next action নির্বাচন করতে পারে। Deterministic অংশ workflow-তে রাখা সহজে testable।

### ৬. Agent loop কীভাবে থামাবে?
Task completion, max steps, timeout, budget, cancellation, no-action state ও human approval boundary দিয়ে।

### ৭. Memory-এর ধরন কী?
Current session/task state এবং cross-session persistent memory। Persistent memory-তে privacy, retention ও deletion policy দরকার।

### ৮. Multi-agent কখন ব্যবহার করবে?
Distinct specialized tasks independentভাবে delegate করলে লাভ হতে পারে। তবে coordination/cost/latency বাড়ে; simple workflow-তে প্রয়োজন নেই।

### ৯. LangGraph কেন?
Stateful graph-based workflow ও agent loop orchestrate করতে। Framework ব্যবহার করলেও state transition ও tool security বুঝতে হবে।

## Tool Security

### ১০. Tool schema কেন?
Input shape ও type constrain করে, parsing ও validation সহজ করে। Schema validation authorization-এর বিকল্প নয়।

### ১১. Least privilege কী?
Tool/credential-কে কাজের জন্য প্রয়োজনীয় সর্বনিম্ন permission দেওয়া।

### ১২. SQL agent কীভাবে secure করবে?
Read-only DB role, table allowlist, AST/query validation, parameterization, timeout, row limit, tenant authorization ও audit log।

### ১৩. Prompt injection agent-এর জন্য কেন ঝুঁকি?
Untrusted content agent-কে tool misuse বা data exfiltration-এর দিকে প্ররোচিত করতে পারে। Tool permission server-side enforce করতে হবে।

### ১৪. Human-in-the-loop কখন?
Irreversible, financial, privacy-sensitive বা high-impact action-এর আগে confirmation/review।

## Production

### ১৫. Agent infinite loop আটকাবে কীভাবে?
Max iteration, tool call budget, wall-clock timeout, repeated-call detection ও explicit termination policy।

### ১৬. Tool failure হলে কী করবে?
Structured error return, retryable classification, bounded retry, fallback বা graceful stop। একই error loop নয়।

### ১৭. Idempotency কেন দরকার?
Retry বা duplicate request-এ side effect একাধিকবার না ঘটাতে।

### ১৮. Observability-তে কী track করবে?
Tool selection, arguments (redacted), execution result, latency, error, model/prompt version, tokens ও task completion।

### ১৯. Agent cost কীভাবে কমাবে?
Unnecessary tool/model call কমানো, step budget, model routing, context compression, caching ও task-level cost monitoring।

### ২০. Caching-এর security risk কী?
Permission scope ছাড়া cache key করলে এক user অন্য user-এর private result পেতে পারে। Authorization-aware cache design দরকার।

### ২১. Model/prompt versioning কেন?
Regression debug, reproducibility, controlled rollout ও rollback-এর জন্য।

### ২২. Docker image-এ API key রাখা যাবে?
না। Secret runtime-এ secure environment/secret manager থেকে inject করতে হবে; image layer-এ নয়।

## Scenario Questions

### ২৩. Agent ভুল customer-এর data দেখাচ্ছে। কী করবে?
Tool-level authorization, identity binding, tenant filter, argument validation ও cache isolation investigate করব। এটি security incident হিসেবে treat করব।

### ২৪. Agent একই tool বারবার call করছে। কী করবে?
Max step, duplicate call detection, state transition ও tool result error contract পরীক্ষা করব। প্রয়োজন হলে workflow-কে deterministic করব।

### ২৫. Agent refund execute করেছে কিন্তু user confirmation নেয়নি। কী সমস্যা?
Authorization/approval boundary ভাঙা। Refund tool-কে pending approval ছাড়া execution না করতে বাধ্য করতে হবে; prompt instruction যথেষ্ট নয়।

### ২৬. Agent final answer সঠিক, কিন্তু অনেক বেশি খরচ হয়েছে। কীভাবে improve?
Tool trajectory inspect, unnecessary planning/calls কমানো, step budget, smaller model routing, caching ও task success per cost measure করব।

### ২৭. Production provider outage হলে?
Timeout/circuit breaker, graceful error, queue/defer বা approved fallback। Silent incorrect answer দেওয়া যাবে না।

### ২৮. Agent quality কীভাবে evaluate করবে?
Task completion, tool choice, argument correctness, policy compliance, unnecessary actions, latency ও cost। Golden scenario এবং adversarial test রাখব।

## Coding Discussion

### ২৯. Tool call-এর আগে কী validation?
Schema, argument bounds, user authorization, tenant scope, resource ownership ও action policy।

### ৩০. Retryable error-এর উদাহরণ?
Transient network failure বা কিছু rate-limit response; provider guidance অনুযায়ী retry। Invalid schema বা permission denied সাধারণত retry নয়।


---

## Additional Interview Questions: Agent Reliability and Orchestration

### ৩১. Tool calling এবং MCP-এর পার্থক্য কী?

Tool calling হলো model/application interaction pattern, যেখানে model structured tool call propose করে। MCP (Model Context Protocol) tools/resources/prompts expose করার protocol। MCP server tool expose করতে পারে, কিন্তু tool calling MCP ছাড়াও সম্ভব।

### ৩২. Agent-এর state কেন explicit রাখা দরকার?

Conversation text থেকে implicit state infer করলে restart, retry, debugging ও validation কঠিন হয়। Explicit state-এ current step, validated result, budget, approval ও retry count রাখা যায় এবং state transition test করা যায়।

### ৩৩. Checkpoint কী?

Long-running workflow-এর progress persist করা snapshot/state record। Process crash/restart হলে completed step পুনরায় না চালিয়ে checkpoint থেকে resume করা যায়। Side-effecting step-এর idempotency তবুও দরকার।

### ৩৪. Human approval কি শুধু prompt-এ instruction দিলেই হবে?

না। Application state machine-এ pending approval enforce করতে হবে। Tool handler নিজেই approved action token/state ছাড়া execute না করার নিশ্চয়তা দেবে। Confirmation নির্দিষ্ট payload-এর সঙ্গে bind করতে হবে।

### ৩৫. Approval stale হতে পারে কেন?

Approval দেওয়ার পরে resource state, amount, permission বা target বদলে যেতে পারে। Execute করার আগে current state ও authorization re-check করতে হবে; payload বদলালে নতুন approval চাইতে হবে।

### ৩৬. Multi-agent system সবসময় ভালো কেন নয়?

Agent coordination, handoff, shared state, duplicated work, latency, token cost ও security surface বাড়ে। Parallel specialization measurable benefit না দিলে single agent/workflow সহজ ও reliable।

### ৩৭. Agent-এর plan কি execution permission?

না। Plan হলো proposed sequence। প্রতিটি action-এর আগে tool allowlist, schema, user permission, current state ও approval policy যাচাই করতে হবে।

### ৩৮. Tool output injection কী?

Tool বা retrieved content-এর ভিতরে malicious instruction থাকতে পারে। Model সেটিকে instruction হিসেবে follow করলে tool misuse হতে পারে। Tool output untrusted data হিসেবে treat করো; server-side permission enforce করো।

### ৩৯. Agent loop কীভাবে bounded করবে?

Max steps/tool calls, wall-clock timeout, token/cost budget, repeated-call detection এবং explicit terminal state দিয়ে। Budget শেষ হলে safe failure বা human handoff।

### ৪০. Agent-এর tool selection accuracy কীভাবে evaluate করবে?

Golden scenario-তে expected tool/arguments define করো। Correct tool selection, argument validity, unnecessary call, policy violation, retry behavior এবং task completion measure করো।

### ৪১. Agent final answer correct হলেও কেন trajectory evaluate করতে হবে?

Agent ভুল বা unauthorized tool call করে পরে correct answer দিতে পারে। Side effect, data exposure বা unnecessary cost final answer-এ দেখা নাও যেতে পারে। Intermediate action audit করতে হবে।

### ৪২. Tool idempotency কীভাবে implement করবে?

Client/application generated idempotency key unique scope-এ persist করো। Same key পুনরায় এলে previous result return করো; action দ্বিতীয়বার execute কোরো না। Key persistence ও side effect atomicity বিবেচনা করো।

### ৪৩. Structured tool error কেন দরকার?

Agent retry, clarify, stop বা human escalation বেছে নিতে পারে। Error category যেমন validation_error, permission_denied, not_found, rate_limited ও temporary_unavailable আলাদা হওয়া উচিত।

### ৪৪. Circuit breaker কীভাবে agent workflow-এ সাহায্য করে?

Repeated dependency failure হলে call সাময়িক বন্ধ করে, যাতে resource waste ও cascading failure কমে। Cooldown শেষে limited probe দিয়ে recovery test করা হয়।

### ৪৫. Agent memory কি সবসময় trusted?

না। Memory stale, incorrect বা unauthorized হতে পারে। Source, timestamp, user scope, permission ও retention track করো। Memory content থেকে permission derive কোরো না।

### ৪৬. Durable execution-এর জন্য কী persist করবে?

Task ID, state version, completed step output, pending approval, idempotency key reference, retry count, deadline/budget এবং trace reference। Secrets ও unnecessary sensitive data persist কোরো না।

### ৪৭. Agent-এর জন্য workflow framework কখন ব্যবহার করবে?

যখন durable state, checkpoint, human pause/resume, graph branching, streaming বা complex orchestration দরকার। Simple task-এ raw function/tool loop যথেষ্ট হতে পারে। Framework complexity ও operational dependency বিবেচনা করো।

### ৪৮. Agent system-এ least privilege কীভাবে apply করবে?

প্রতিটি tool-কে নির্দিষ্ট permission দাও; user/tenant context bind করো; read/write tool আলাদা করো; output সীমিত করো; action-এর আগে authorization check করো। Model-এর prompt-এ permission লিখে রাখাই যথেষ্ট নয়।

### ৪৯. Agent cost কীভাবে monitor করবে?

Per-task token, model call, tool call, retry, latency এবং cost per successful task measure করো। Unnecessary loop, repeated retrieval ও oversized context identify করো।

### ৫০. Agent production incident হলে কী করবে?

Unsafe tool/action বন্ধ, affected user/tenant scope identify, model/prompt/tool version trace, state/audit logs inspect, প্রয়োজন হলে rollback/disable, security escalation এবং regression scenario add করো।

## Agent Execution Flow

~~~mermaid
flowchart TD
    A["Goal and authenticated context"] --> B["Load versioned state"]
    B --> C["Model proposes next step"]
    C --> D["Validate tool and arguments"]
    D --> E["Authorize / approval check"]
    E --> F["Execute with timeout and idempotency"]
    F --> G["Persist result / checkpoint"]
    G --> H{"Complete or budget exhausted?"}
    H -->|না| C
    H -->|হ্যাঁ| I["Return / handoff"]
~~~

## Oral Practice

একটি agent scenario বেছে নাও এবং explain করো: state কোথায় থাকে, tool permission কে enforce করে, failure হলে কী হয়, approval কীভাবে bind হয় এবং task কীভাবে terminate করে।
