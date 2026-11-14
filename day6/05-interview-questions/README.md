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
