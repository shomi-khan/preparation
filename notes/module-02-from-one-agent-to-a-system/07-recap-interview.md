# 07. Module 2 Recap ও Interview Drill

এই chapter-এর উদ্দেশ্য শুধু Tool Calling, MCP, Skills, Orchestration ও Gateway-এর definition আবার পড়া নয়। Module 2-তে আমরা দেখেছি, একটি agent যখন বাইরের system-এর সঙ্গে কাজ করে, তখন capability, instruction, protocol, coordination ও policy-এর আলাদা responsibility তৈরি হয়। Interview-তে এই ধারণাগুলোকে এক architecture-এ যুক্ত করে ব্যাখ্যা করতে পারাই আসল দক্ষতা।

## 1. 90-second recap: একটি Agent system-এর কাজের flow

ধরো, একটি customer support agent-কে damaged product-এর refund request resolve করতে হবে।

1. Host/runtime user request গ্রহণ করে এবং model-কে context দেয়।
2. Agent task অনুযায়ী refund investigation skill load করতে পারে।
3. Skill workflow guide করে: order verify, policy retrieve, evidence check, eligibility validate।
4. Model প্রয়োজনীয় tool call propose করে।
5. Runtime schema, allowlist, turn budget ও policy check করে।
6. MCP ব্যবহার করলে client standardized protocol দিয়ে server capability invoke করে।
7. Gateway থাকলে access, routing, credential, quota ও audit mediate করে।
8. Tool/server domain service-এর সঙ্গে কাজ করে; domain service authoritative business rule ও resource permission enforce করে।
9. Result context-এ ফিরে আসে; agent next step বা final response তৈরি করে।
10. Side effect-এর পরে authoritative status verify হয়; ambiguity/high-risk case human review-এ যায়।

এই flow-এর প্রতিটি layer আলাদা problem solve করে। Tool capability, Skill procedure, MCP protocol, Orchestration coordination, Gateway access governance। এগুলোকে একে অন্যের replacement ভাবলে architecture ভুল হবে।

## 2. Quick comparison: কোন concept কোন প্রশ্নের উত্তর দেয়?

| Concept | মূল প্রশ্ন | Responsibility |
|---|---|---|
| Agent | Goal নিয়ে কে decision/coordination করে? | Goal-directed behavior |
| Runtime/Host | Execution কীভাবে control হবে? | Model loop, state, dispatch, stop/budget |
| Tool | কোন operation চালানো যায়? | Executable capability |
| Skill | কাজটি কী পদ্ধতিতে করা উচিত? | Reusable instruction/workflow |
| MCP | Capability standardized ভাবে কীভাবে discover/call হবে? | Protocol |
| Multi-agent orchestration | কাজ কার কাছে যাবে, result কীভাবে join হবে? | Delegation ও coordination |
| AI Gateway | Access কীভাবে govern/route/monitor হবে? | Policy mediation |
| Domain service | Business operation সত্যিই valid কি? | Authoritative state ও invariant |

## 3. Interview answer কীভাবে সাজাবে?

একটি definition বলেই থেমে গেলে follow-up প্রশ্নে আটকে যাওয়ার সম্ভাবনা থাকে। প্রতিটি system-design প্রশ্নে এই কাঠামো অনুসরণ করো:

1. **Definition:** বিষয়টি কী?
2. **Problem:** কোন সমস্যা সমাধান করে?
3. **Mechanism:** ভেতরে কীভাবে কাজ করে?
4. **Boundary:** কোন component-এর responsibility কোথায় শেষ?
5. **Example:** বাস্তব scenario।
6. **Trade-off:** কী সুবিধা ও কী খরচ?
7. **Failure:** কোথায় ভাঙতে পারে?
8. **Mitigation:** কীভাবে safely recover করবে?
9. **Evaluation:** design ভালো হয়েছে কীভাবে মাপবে?

# Part A: Tool Calling

## Q1. Tool calling কীভাবে কাজ করে?

Host model-কে conversation ও available tool schema দেয়। Model প্রয়োজন বুঝে structured tool-call proposal তৈরি করে। Host/runtime সেটি parse করে, tool allowlist ও schema validate করে, identity/permission/approval check করে এবং trusted implementation invoke করে। Result বা structured error conversation context-এ যোগ হয়; model এরপর final response দিতে পারে বা আরেকটি tool call প্রস্তাব করতে পারে।

গুরুত্বপূর্ণ বিষয়: **model proposal দেয়, runtime execute ও enforce করে।** এই separation না থাকলে model-এর ভুল বা malicious instruction সরাসরি side effect তৈরি করতে পারে।

## Q2. Schema-valid call কেন unsafe হতে পারে?

Schema শুধু structure, type ও required field যাচাই করে। যেমন `amount_minor: 500` integer হিসেবে valid, কিন্তু user-এর refundable balance 100 হলে business rule ভঙ্গ। একইভাবে `order_id` string হলেও সেটি অন্য tenant-এর order হতে পারে।

তাই schema validation-এর পরে domain/business validation এবং resource-level authorization দরকার। Authoritative service-এও check রাখতে হবে, কারণ model-facing layer bypass হতে পারে।

## Q3. Tool timeout-এর পরে duplicate side effect কীভাবে আটকাবে?

Timeout মানে operation ব্যর্থ হয়েছে এমন নয়। Server transaction commit করলেও response হারিয়ে যেতে পারে। Side-effecting operation-এ idempotency key ব্যবহার, operation status lookup ও reconciliation দরকার। একই key-তে retry এলে নতুন transaction না করে আগের result ফেরত দিতে হবে। Blind retry duplicate refund বা payment তৈরি করতে পারে।

## Q4. Tool description বনাম authorization policy?

Description model-কে tool-এর উদ্দেশ্য, input ও constraint বুঝতে সাহায্য করে। Authorization trusted runtime/server-এ enforce হয়। Description-এ “শুধু নিজের order পড়বে” লিখলে malicious বা ভুল tool call আটকায় না। Server-কে authenticated principal, tenant ও resource ownership যাচাই করতে হবে।

## Q5. ১০০টি tool দিলে কী সমস্যা হতে পারে?

Tool selection ambiguity, context overhead, wrong selection ও evaluation complexity বাড়তে পারে। Domain অনুযায়ী tool discovery/filtering, concise description, task-specific allowlist ও grouping ব্যবহার করা যায়। Wrong-tool rate, task success, latency ও token cost দিয়ে evaluate করতে হবে।

# Part B: MCP

## Q6. MCP কী এবং কেন দরকার?

MCP হলো AI host/client ও capability server-এর মধ্যে standardized protocol। এটি tools, resources ও prompts discover/invoke করার common pattern দেয়। ফলে compatible host একাধিক server-এর capabilities একই ধরনের interaction flow দিয়ে ব্যবহার করতে পারে। এটি model বা reasoning engine নয়; integration interoperability-এর সমস্যা সমাধান করে।

## Q7. MCP Host, Client ও Server-এর দায়িত্ব কী?

Host হলো AI application/runtime, যা user interaction, model এবং orchestration পরিচালনা করে। MCP Client host-এর অংশ, যা server-এর সঙ্গে protocol connection ও request/response lifecycle পরিচালনা করে। MCP Server tools/resources/prompts expose করে এবং request handle করে। Model সাধারণত host-এর মাধ্যমে capability সম্পর্কে জানে; model নিজে server-এর network connection manage করে, এমন ধরে নেওয়া উচিত নয়।

## Q8. Tool, Resource ও Prompt-এর পার্থক্য কী?

Tool action বা operation invoke করে; Resource data/context read করার উপায়; Prompt reusable prompt template বা interaction starter। Tool side effect তৈরি করতে পারে বলে permission গুরুত্বপূর্ণ। Resource content untrusted instruction বহন করতে পারে। Prompt workflow guide করে, কিন্তু authorization enforce করে না।

## Q9. MCP কি Agent Framework বা Orchestrator?

না। MCP capability discovery ও invocation standardize করে। Agent runtime model loop, state, planning, tool execution policy, retries, orchestration ও stopping rule পরিচালনা করতে পারে। MCP framework-এর integration layer হিসেবে ব্যবহৃত হতে পারে, কিন্তু পুরো runtime বা workflow engine-এর বিকল্প নয়।

## Q10. MCP কি security নিশ্চিত করে?

না। MCP interoperability protocol, security architecture নয়। Host-এ user authentication/consent, client-এ connection trust, server-এ authorization ও input validation, downstream service-এ resource-level permission, transport-এ TLS/authentication এবং পুরো flow-তে audit দরকার। Tool description-এ “শুধু নিজের data পড়বে” লিখলে malicious request আটকায় না।

## Q11. stdio বনাম Streamable HTTP কখন বেছে নেবে?

Local child process integration-এ stdio উপযোগী হতে পারে; remote/service deployment-এ Streamable HTTP প্রাসঙ্গিক হতে পারে। সিদ্ধান্তে process lifecycle, network exposure, authentication, TLS, filesystem permission, deployment ও availability বিবেচনা করতে হবে। Transport বদলালে trust boundary-ও বদলায়। Implementation-এর ক্ষেত্রে ব্যবহৃত protocol/SDK version-এর current behavior যাচাই করতে হবে।

# Part C: Agent Skills

## Q12. Skill এবং Tool কেন আলাদা?

Tool executable capability; Skill reusable procedure/instruction। `get_order` tool order details আনে। Refund-investigation skill বলে কখন order lookup করবে, কোন policy version পড়বে, কী evidence চাইবে এবং কখন escalate করবে। Skill tools sequence করতে পারে, কিন্তু নিজে external operation execute করে না।

## Q13. Progressive disclosure কীভাবে context বাঁচায়?

প্রথমে skill-এর ছোট metadata/description discoverable থাকে। Task-এর সঙ্গে match করলে core instruction load হয়; দরকার হলে reference বা asset on-demand load হয়। এতে সব skill-এর full text প্রতিটি request-এ context-এ রাখতে হয় না। তবে metadata দুর্বল হলে relevant skill নির্বাচন ব্যর্থ হতে পারে।

## Q14. Skill কি deterministic behavior নিশ্চিত করে?

না। Skill natural-language instruction; model সেটি interpret করে। Model ভুল বুঝতে বা step বাদ দিতে পারে। Exact validation, permission, calculation ও state transition deterministic code-এ enforce করো। Skill adherence representative evaluation দিয়ে মাপো।

## Q15. Malicious skill instruction কীভাবে risk তৈরি করে?

Skill model-কে unsafe action propose করতে, sensitive data প্রকাশ করতে বা policy bypass করতে বলতে পারে। Script থাকলে code execution risk-ও থাকে। Trusted source, review, version pinning, sandboxing, permission restriction ও backend authorization দরকার।

# Part D: Multi-Agent Orchestration

## Q16. Multi-agent কখন ব্যবহার করবে?

যখন task-এ independent subtask parallel করা যায়, আলাদা bounded expertise দরকার, অথবা responsibility isolation measurable value দেয়। আগে single-agent/deterministic baseline তৈরি করব। একই test set-এ correctness, success, policy violation, latency, cost ও failure rate তুলনা করব। Multi-agent-এর extra coordination overhead benefit-এর চেয়ে বেশি হলে ব্যবহার করব না।

## Q17. Parallel agent flow-তে কী failure হতে পারে?

কোনো branch timeout, partial result, stale state, contradictory evidence, rate limit বা cancellation হতে পারে। Join point-এ per-branch deadline, partial-result policy, evidence provenance, conflict resolution ও escalation দরকার। Shared state-এর authoritative source নির্দিষ্ট করতে হবে। সব branch success না হলে final decision নেওয়া নিরাপদ কি না business requirement নির্ধারণ করবে।

## Q18. Handoff contract-এ কী থাকবে?

Goal, relevant context/evidence, tenant/resource scope, constraints, allowed tools, expected output schema, deadline, uncertainty ও escalation condition। পুরো transcript পাঠানোর বদলে minimum necessary context পাঠাতে হবে। Child agent-এর output validate করতে হবে; parent-এর কাছে ফিরে আসা message authoritative database state-এর বিকল্প নয়।

## Q19. দুই agent পরস্পরবিরোধী result দিলে কী করবে?

Source, evidence, freshness ও scope যাচাই করব। Authoritative domain service বা official policy source দিয়ে conflict resolve করব। Majority vote যথেষ্ট নয়, কারণ দুই agent একই ভুল source ব্যবহার করতে পারে। High-impact ambiguity unresolved থাকলে human review-এ পাঠাব।

# Part E: AI Gateway

## Q20. AI Gateway কী এবং কেন দরকার?

একাধিক agent ও tool থাকলে credential, policy, routing, quota ও logging আলাদা implement করলে inconsistency তৈরি হয়। Gateway এসব cross-cutting concern centralize করতে পারে। এতে access governance, observability ও credential isolation সহজ হয়। তবে ছোট application-এ Gateway বাধ্যতামূলক নয়; application ও domain service-এ responsibility যথেষ্টভাবে থাকলে আলাদা gateway overhead অপ্রয়োজনীয় হতে পারে।

## Q21. Gateway এবং MCP server-এর পার্থক্য কী?

MCP server capability expose/implement করে। Gateway একাধিক tool/service-এর সামনে intermediary হিসেবে routing, credential, policy, quota ও observability centralize করতে পারে। Gateway MCP server-এর সামনে থাকতে পারে, কিন্তু MCP protocol আর Gateway architectural role এক নয়। MCP interoperability; Gateway governance/control layer।

## Q22. Gateway থাকলেও downstream authorization কেন?

Gateway centralized access control দেয়, কিন্তু bypass path, misconfiguration বা internal service call থাকতে পারে। Domain service-কে resource ownership, business invariant ও state transition নিজে enforce করতে হবে। Gateway coarse policy gate; domain service authoritative business guard। এটাই defense in depth।

## Q23. Gateway outage হলে fail-open নাকি fail-closed?

Risk অনুযায়ী। Refund, payment, deployment বা delete-এর মতো sensitive write action-এ policy service unavailable হলে সাধারণত fail-closed। Low-risk read-only action-এ short-lived cached policy দিয়ে সীমিত degraded mode সম্ভব হতে পারে, যদি scope ও expiry নিশ্চিত থাকে। Decision explicit risk policy-তে লিখতে হবে।

## Q24. Logging-এ কী রাখবে, কী redact করবে?

Trace ID, actor/tenant, tool/version, policy decision, latency, status/error category, retry count ও downstream request ID রাখব। Password, token, full payment data, unnecessary PII ও raw sensitive content redact বা বাদ দেব। Retention, access control ও audit integrity নিশ্চিত করব।

# Part F: Integrated System Design

## Q25. Tool, Skill, MCP, Gateway ও Domain Service-এর responsibility এক flow-তে ব্যাখ্যা করো।

Refund workflow-এ Skill procedure guide করে। Model tool call propose করে। Runtime schema ও allowlist validate করে। MCP থাকলে client protocol request server-এ পাঠায়। Gateway coarse access, quota ও routing enforce করতে পারে। Tool server domain API invoke করে। Domain service resource ownership, refundable balance ও transaction invariant যাচাই করে। Result runtime-এ ফেরে, যা authoritative status verify করে user-কে জানায়।

প্রতিটি layer-এর কাজ আলাদা। Gateway business state-এর owner নয়; Skill permission দেয় না; MCP reasoning করে না; model নিজে tool execute করে না।

## Q26. Prompt injection-এর বিরুদ্ধে কী করবে?

User/retrieved/tool content-কে trusted instruction হিসেবে ধরব না। Model context-এ secret কম রাখব, tool scope least privilege করব, runtime ও service-side authorization enforce করব, outbound side effect validate করব, high-risk action-এ approval রাখব এবং adversarial test চালাব। Prompt wording একা যথেষ্ট নয়; লক্ষ্য হলো injection হলেও unauthorized action যেন সম্ভব না হয়।

## Q27. Production agent system কীভাবে evaluate করবে?

Representative offline test set ও adversarial case তৈরি করব। Task success, correctness, tool selection/argument accuracy, policy violation, prompt injection success, human escalation, p50/p95 latency, token/API cost, timeout ও duplicate side effect মাপব। Single-agent baseline-এর সঙ্গে তুলনা, canary rollout এবং production trace review করব। শুধু answer fluency quality metric নয়।

## Q28. Production failure হলে কীভাবে recover করবে?

প্রতিটি workflow-এর durable state রাখব। Timeout-এর পর side effect হয়েছে কি না reconcile করব। Idempotency key দিয়ে duplicate action ঠেকাব। Policy deny হলে action বন্ধ থাকবে। Approval pending হলে state persist করে অপেক্ষা করবে। Gateway outage-এ sensitive write fail-closed হবে। Unrecoverable ambiguity human review-এ যাবে। Recovery path আগে design না করলে agent-এর “আরেকবার চেষ্টা করো” যথেষ্ট নয়।

# Part G: System Design Exercise

## Design a Refund Agent Platform

Diagram-এ দেখাও:
- Host/runtime ও LLM
- Skill catalog
- Tool catalog
- Optional MCP client/server
- Optional Gateway/policy
- Order/refund API
- Memory/workflow state
- Human approval
- Audit/tracing

প্রতিটি component-এর ownership ও trust boundary explain করো।

### Critical flow

Read-only lookup → policy/evidence check → current state revalidation → approval (if needed) → idempotent refund command → authoritative status reconciliation → audit → user response।

### Failure paths

- Policy deny হলে action নয়।
- Approval pending হলে wait/notify।
- Timeout হলে operation ID দিয়ে reconcile।
- Duplicate request idempotency দিয়ে suppress।
- Gateway outage-এ sensitive write fail-closed।
- Prompt injection-এ untrusted data boundary ও server-side permission বজায় থাকবে।

### Interviewer-এর follow-up challenge

**“Agent-কে deployment permission দিলে ভুল deploy কীভাবে আটকাবে?”**  
Model-কে production credential দেব না। Deploy tool narrow হবে; runtime identity ও environment allowlist check করবে; deployment service artifact, target, approval ও change window যাচাই করবে; production-এ human approval লাগতে পারে; deployment ID, rollback, canary ও audit থাকবে। Skill checklist guide করবে, policy enforce করবে service।

**“একটি agent অন্য agent-কে secret চাইতে বলল?”**  
Agent-to-agent message trust boundary। Child agent-কে minimum context ও scoped capability দেব। Secret model context বা handoff payload-এ দেব না। Credential broker/service identity অনুযায়ী short-lived scoped access দেবে। Tool server authorization independently enforce করবে।

**“MCP server tool schema বদলে দিল?”**  
Capability discovery/cache refresh, schema compatibility, strict validation ও contract test দরকার। Unknown breaking schema হলে fail safely; rollout-এ version pinning বা compatibility check থাকবে। Tool output untrusted data হিসেবেই বিবেচিত হবে।

## 4. Oral practice rubric

প্রতিটি উত্তরে নিজেকে ৬টি প্রশ্ন করো:

1. Concept সঠিকভাবে define করেছি?
2. Execution flow ব্যাখ্যা করেছি?
3. Responsibility ও authority কোথায় তা বলেছি?
4. Failure ও trade-off উল্লেখ করেছি?
5. বাস্তব উদাহরণ দিয়েছি?
6. কী metric দিয়ে design যাচাই করব বলেছি?

শুধু acronym বা framework-এর নাম বললে উত্তর সম্পূর্ণ নয়। একজন engineer হিসেবে তোমাকে বোঝাতে হবে, system-টি কেন এমনভাবে design করেছ এবং failure হলে কীভাবে নিরাপদে recover করবে।
