# 07. Module 2 Recap ও Interview Drill

## 90-second recap

Tool calling external capability ব্যবহার করতে দেয়। Model structured call propose করে; host/runtime validate, authorize ও execute করে। MCP host/client/server-এর মধ্যে capability discover ও access করার common protocol; এটি tools/resources/prompts expose করতে পারে। Agent Skill tool access-এর বদলে reusable workflow ও instruction দেয়, progressive loading দিয়ে relevant context-এ আসে। Multi-agent system responsibility ভাগ করে, কিন্তু coordination, handoff, latency, cost ও failure propagation বাড়ায়। AI Gateway tool access-এর সামনে routing, auth, policy, quota, credential ও observability centralize করতে পারে। সব component থাকলেও security/correctness trusted server-side layer-এ enforce করতে হবে।

## Quick comparison

| Concept | মূল প্রশ্ন |
|---|---|
| Agent | Goal নিয়ে কে সিদ্ধান্ত/coordination করে? |
| Tool | কোন কাজ execute করতে পারে? |
| Skill | কাজটি কীভাবে ভালোভাবে করবে? |
| MCP | Capability standardized ভাবে কীভাবে discover/call হবে? |
| Multi-agent | Responsibility কীভাবে ভাগ/coordinate হবে? |
| Gateway | Access কীভাবে govern/route/monitor হবে? |

## Foundational interview

**1. Tool calling কীভাবে কাজ করে?**  
Schema model-কে দেওয়া হয়; model call propose করে; runtime parse/validate/authorize/execute করে; result context-এ ফেরত দিয়ে model continue বা final answer দেয়।

**2. MCP কেন?**  
AI host/client ও external capability server-এর মধ্যে reusable standard interface দিতে, bespoke integration কমাতে।

**3. MCP Host, Client, Server?**  
Host application/runtime; client protocol connection; server capability expose ও request handle করে।

**4. Tool vs Skill?**  
Tool executable capability; Skill reusable instruction/workflow।

**5. Multi-agent কখন?**  
Task decomposition/specialization/parallelism বাস্তব value দিলে; baseline-এর তুলনায় measurable improvement থাকলে।

**6. AI Gateway?**  
Tools/services-এর access intermediary, policy, routing, credential, quota ও observability manage করে।

## Deep interview questions

1. Model schema-valid arguments দিলেও business rule violate করলে কোথায় reject?
2. Tool timeout-এর পর duplicate side effect কীভাবে আটকাবে?
3. Tool description বনাম authorization policy?
4. Compromised MCP server-এর blast radius কীভাবে limit?
5. Skill outdated হলে version/review/evaluation?
6. 100 tools দিলে selection degradation কীভাবে measure?
7. Parallel agent partial failure কীভাবে handle?
8. Handoff context/contract কীভাবে নিশ্চিত?
9. Gateway ও downstream service দুটিতে authorization কেন?
10. Gateway outage-এর failure mode?
11. Tool-call audit-এ কী রাখবে, কী redact?
12. Tool output prompt injection defense?
13. User, skill, retrieved doc-এর trust boundary?
14. MCP কি orchestration, memory, reasoning replace করে?
15. Parent agent child-agent contract কীভাবে define করবে?

## System design exercise

Design a Refund Agent Platform. Diagram-এ:
- Host/runtime, LLM
- Skill catalog
- Tool catalog
- MCP client/server
- Gateway/policy
- Order/refund API
- Memory/state
- Human approval
- Audit/tracing

Component ownership ও trust boundary explain করো। Failure paths: timeout, policy deny, approval pending, duplicate refund, service outage।

## Self-review checklist

- [ ] Tool call proposal বনাম execution
- [ ] Schema ও business validation
- [ ] Authorization ও approval
- [ ] MCP Host/Client/Server
- [ ] Tools/Resources/Prompts
- [ ] Transport/deployment trade-offs
- [ ] Skill vs Tool
- [ ] Progressive disclosure
- [ ] Single vs Multi-agent
- [ ] Sequential/parallel/supervisor/handoff
- [ ] Gateway responsibilities/limitations
- [ ] Least privilege, idempotency, timeout, trace
- [ ] Prompt injection ও untrusted tool output

## Mock interview

Interviewer: “আমাদের agent-কে Slack, Jira, GitHub, database ও deployment system access দিতে হবে। সব tools এক agent-কে দেব, নাকি multiple agents? MCP ও Gateway কোথায় বসবে? Skills দরকার? Production security কীভাবে করব?”

উত্তর:
1. Requirements/risk clarify।
2. Single-agent baseline।
3. Tool boundary/schema।
4. Skill workflow।
5. MCP integration decision।
6. Multi-agent-এর measurable justification।
7. Gateway/security।
8. Observability/evaluation।
9. Failure handling/human approval।

Framework-এর নাম বলার চেয়ে component-এর responsibility, trade-off, failure mode ও enforcement point ব্যাখ্যা করাই বেশি গুরুত্বপূর্ণ।

## Deep interview answers: যুক্তিসহ উত্তর

### 1. Tool calling-এর lifecycle ব্যাখ্যা করো

Host model-কে conversation ও available tool schema দেয়। Model প্রয়োজন বুঝে structured tool-call proposal তৈরি করে। Host/runtime সেটি parse করে, tool allowlist ও schema validate করে, identity/permission/approval check করে এবং trusted implementation invoke করে। Result বা structured error conversation context-এ যোগ হয়; model এরপর final response দিতে পারে বা আরেকটি tool call প্রস্তাব করতে পারে। গুরুত্বপূর্ণ বিষয়: model proposal দেয়, runtime execute ও enforce করে। এই separation না থাকলে model-এর ভুল বা malicious instruction সরাসরি side effect তৈরি করতে পারে।

### 2. Schema-valid call কেন unsafe হতে পারে?

Schema শুধু structure/type/required field যাচাই করে। যেমন `amount: 500` integer হিসেবে valid, কিন্তু user-এর refundable balance 100 হলে business rule ভঙ্গ। একইভাবে order_id string হলেও সেটি অন্য tenant-এর order হতে পারে। তাই schema validation-এর পরে domain/business validation এবং resource-level authorization দরকার। Authoritative service-এও check রাখতে হবে, কারণ model-facing layer bypass হতে পারে।

### 3. MCP কী এবং এটি কী নয়?

MCP হলো host/client ও server-এর মধ্যে capability discovery ও interaction-এর common protocol। Server tools, resources ও prompts expose করতে পারে। এটি model, agent framework, memory store, reasoning engine বা complete workflow orchestrator নয়। এটি integration interface standardize করে; business semantics, permission policy, retries ও high-level coordination host/server/application-এর দায়িত্বে থাকে।

### 4. MCP Host, Client ও Server আলাদা কেন?

Host user interaction, model ও orchestration-এর owner। Client protocol connection/session পরিচালনা করে। Server capability expose করে এবং request handle করে। এই separation এক host-কে একাধিক server-এ connect করতে দেয় এবং protocol communication-কে application logic থেকে আলাদা রাখে। Model সরাসরি server-এ যুক্ত থাকতেই হবে এমন নয়; host/client মধ্যস্থতা করতে পারে।

### 5. Tool, Resource ও Prompt-এর পার্থক্য কী?

Tool action বা operation invoke করে, যেমন issue create। Resource data/context পড়ার উপায়, যেমন document content। Prompt reusable template বা interaction starter। Tool side effect তৈরি করতে পারে; resource content untrusted instruction বহন করতে পারে; prompt workflow guide করে কিন্তু authorization enforce করে না। Capability অনুযায়ী permission ও trust handling আলাদা হওয়া উচিত।

### 6. Skill এবং Tool কেন আলাদা?

Tool executable capability, Skill reusable procedure/instruction। “Order lookup” tool কাজটি করে; refund-investigation skill বলে কখন lookup করবে, কোন policy পড়বে, কী evidence চাইবে এবং কখন escalate করবে। Skill tools-কে sequence করতে পারে, কিন্তু নিজে trusted security boundary নয়। Runtime ও domain service permission enforce করে।

### 7. Progressive disclosure কীভাবে context বাঁচায়?

প্রথমে skill-এর ছোট metadata/description discoverable থাকে। Task-এর সঙ্গে মিললে core instruction load হয়; দরকার হলে reference বা asset on-demand load হয়। এতে সব skill-এর full text প্রতিটি request-এ context-এ রাখতে হয় না। তবে discovery metadata ভুল হলে relevant skill নির্বাচন ব্যর্থ হতে পারে, আর loaded instruction-এর conflict-ও evaluate করতে হবে।

### 8. Multi-agent কখন ব্যবহার করবে?

যখন task-এ independent subtask parallel করা যায়, আলাদা bounded expertise দরকার, অথবা responsibility isolation measurable value দেয়। আগে single-agent/deterministic baseline তৈরি করব। একই test set-এ correctness, success, policy violation, latency, cost ও failure rate তুলনা করব। Multi-agent-এর extra coordination overhead benefit-এর চেয়ে বেশি হলে ব্যবহার করব না।

### 9. Parallel agent flow-তে কী কী failure?

কোনো branch timeout, partial result, stale state, contradictory evidence, rate limit বা cancellation হতে পারে। Join point-এ per-branch deadline, partial-result policy, evidence provenance, conflict resolution ও escalation দরকার। Shared state-এর authoritative source নির্দিষ্ট করতে হবে। সব branch success না হলে final decision নেওয়া নিরাপদ কি না business requirement নির্ধারণ করবে।

### 10. Handoff contract-এ কী থাকবে?

Goal, relevant context/evidence, tenant/resource scope, constraints, allowed tools, expected output schema, deadline, uncertainty ও escalation condition। পুরো transcript পাঠানোর বদলে minimum necessary context পাঠাতে হবে। Child agent-এর output validate করতে হবে; parent-এর কাছে ফিরে আসা message authoritative database state-এর বিকল্প নয়।

### 11. AI Gateway এবং MCP server-এর পার্থক্য?

MCP server capability expose/implement করে। Gateway একাধিক tool/service-এর সামনে intermediary হিসেবে routing, credential, policy, quota ও observability centralize করতে পারে। Gateway MCP server-এর সামনে থাকতে পারে, কিন্তু MCP protocol আর gateway architectural role এক নয়। সব system-এ gateway বাধ্যতামূলক নয়।

### 12. Gateway থাকলেও downstream authorization কেন?

Gateway centralized access control দেয়, কিন্তু bypass path, misconfiguration বা internal service call থাকতে পারে। Domain service-কে resource ownership, business invariant ও state transition নিজে enforce করতে হবে। Gateway coarse policy gate; domain service authoritative business guard। এটাই defense in depth।

### 13. Prompt injection-এর বিরুদ্ধে কী করবে?

User/retrieved/tool content-কে trusted instruction হিসেবে ধরব না। Model context-এ secret কম রাখব, tool scope least privilege করব, runtime ও service-side authorization enforce করব, outbound side effect validate করব, high-risk action-এ approval রাখব এবং adversarial test চালাব। Prompt wording একা যথেষ্ট নয়; লক্ষ্য হলো injection হলেও unauthorized action যেন সম্ভব না হয়।

### 14. Timeout-এর পর side effect retry করবে?

Blind retry নয়। Timeout মানে outcome unknown হতে পারে। Idempotency key ব্যবহার, operation status query, transaction reconciliation এবং bounded retry দরকার। Idempotency guarantee না থাকলে duplicate side effect হতে পারে; তখন human/manual reconciliation লাগতে পারে।

### 15. Production agent system কীভাবে evaluate করবে?

Representative offline test set ও adversarial cases তৈরি করব। Task success, correctness, tool selection/argument accuracy, policy violation, prompt injection success, human escalation, p50/p95 latency, token/API cost, timeout ও duplicate side effect মাপব। Single-agent baseline-এর সঙ্গে তুলনা, canary rollout এবং production trace review করব। শুধু answer fluency quality metric নয়।

## System design: Refund Agent Platform-এর model answer outline

প্রথমে requirements ও threat model: user কি নিজের order refund করতে পারবে, refund limit, approval threshold, latency target, audit/retention এবং tenant model কী? এরপর host/runtime + LLM, skill catalog, narrow tools, optional MCP integration, gateway/policy, order/refund domain service, database, approval service ও audit/trace layer আঁকো।

**Ownership:** LLM intent ও next action propose করে; runtime loop/schema/allowlist নিয়ন্ত্রণ করে; skill procedure guide করে; MCP integration protocol দেয়; gateway centralized access policy দেয়; domain service final authorization/business invariant enforce করে; database authoritative state রাখে; human exception approve করে।

**Critical flow:** read-only lookup → policy/evidence check → current state revalidation → approval (if needed) → idempotent refund command → authoritative status reconciliation → audit → user response।

**Failure paths:** policy deny হলে action নয়; approval pending হলে wait/notify; timeout হলে operation ID দিয়ে reconcile; duplicate request idempotency দিয়ে suppress; gateway outage-এ sensitive write fail-closed; prompt injection-এ untrusted data boundary ও server-side permission বজায় থাকে।

## Mock interview: আরও গভীর follow-up

**Interviewer:** “Agent-কে deployment permission দিলে সে ভুল deploy করলে কীভাবে ঠেকাবে?”

**উত্তরের কাঠামো:** Model-কে production credential দেব না। Deploy tool narrow হবে; runtime user/agent identity ও environment allowlist check করবে; deployment service artifact, target, approval ও change window যাচাই করবে; production-এ human approval লাগতে পারে; idempotent deployment ID, rollback strategy, audit ও canary থাকবে। Skill deployment checklist guide করবে, কিন্তু policy enforce করবে service।

**Interviewer:** “একটি agent অন্য agent-কে secret চাইতে বলল?”

**উত্তরের কাঠামো:** Agent-to-agent message trust boundary। Child agent-কে শুধু minimum context ও scoped capability দেব। Secret model context বা handoff payload-এ দেব না। Credential broker/service identity অনুযায়ী short-lived scoped access দেবে। Parent-child contract-এ allowed action থাকবে; tool server authorization independently enforce করবে।

**Interviewer:** “MCP server tool schema বদলে দিল?”

**উত্তরের কাঠামো:** Capability discovery/cache refresh, schema version compatibility, strict validation ও contract test দরকার। Unknown breaking schema হলে fail safely; production rollout-এ pin/version policy বা compatibility check থাকবে। Tool output-কে untrusted data হিসেবেই ধরব।

## Oral practice rubric

প্রতিটি উত্তরে নিজেকে ৫টি প্রশ্ন করো: (১) concept সঠিকভাবে define করেছি? (২) execution flow ব্যাখ্যা করেছি? (৩) responsibility/authority কোথায় তা বলেছি? (৪) failure ও trade-off উল্লেখ করেছি? (৫) বাস্তব উদাহরণ দিয়েছি? শুধু acronym বা framework-এর নাম বললে উত্তর সম্পূর্ণ নয়।

