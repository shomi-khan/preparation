# Production AI Engineering

Prototype-এ model call কাজ করলেই production-ready হয় না। Production AI system-এ provider, database, vector store, tool, network ও user input সবই failure source। Reliability, security, observability, cost এবং deployment lifecycle একসঙ্গে design করতে হয়।

## ১. Reliability কীভাবে design করব?

Consider করো:
- Timeout।
- Bounded retry।
- Exponential backoff ও jitter।
- Circuit breaker।
- Graceful degradation।
- Fallback।
- Idempotency।
- Health check।

~~~mermaid
flowchart TD
    A["Request"] --> B["Dependency call"]
    B --> C{"Healthy response?"}
    C -->|হ্যাঁ| D["Return result"]
    C -->|না| E["Retry if transient and budget allows"]
    E --> F{"Still failing?"}
    F -->|না| D
    F -->|হ্যাঁ| G["Circuit open / fallback"]
    G --> H["Graceful failure"]
~~~

Retry সব error-এ নয়। Permanent validation/auth error retry কোরো না; side effect-এর ক্ষেত্রে idempotency দরকার।

## ২. Circuit Breaker কী?

একটি dependency বারবার fail করলে circuit breaker সাময়িকভাবে call বন্ধ করে দেয়, যাতে failing service-কে আরও চাপ না দেওয়া হয়।

States:
- **Closed:** request pass করে; failure count monitor।
- **Open:** request short-circuit; dependency call নয়।
- **Half-open:** সীমিত probe request দিয়ে recovery test।

~~~mermaid
stateDiagram-v2
    [*] --> Closed
    Closed --> Open: Failure threshold reached
    Open --> HalfOpen: Cooldown elapsed
    HalfOpen --> Closed: Probe succeeds
    HalfOpen --> Open: Probe fails
~~~

## ৩. Rate Limiting এবং Backpressure কী?

Provider quota ও per-user abuse control করতে token bucket/leaky bucket বা queue ব্যবহার করা যায়। Concurrency limit ও backpressure রাখো।

- Request-per-minute limit।
- Token-per-minute limit।
- Per-user/tenant budget।
- Max concurrent provider calls।
- Bounded queue ও overload rejection।

Queue unbounded হলে memory শেষ হতে পারে। Queue length, wait time ও rejection rate monitor করো।

## ৪. Caching কীভাবে design করব?

Prompt/result cache cost ও latency কমাতে পারে। Cache key-তে model version, prompt version, relevant context ও permission scope বিবেচনা করো।

Risks:
- Stale result।
- Private user data cross-user cache leak।
- Prompt/model version mismatch।
- Non-deterministic response cache করে behavior বদলে যাওয়া।
- Invalidation ও TTL ভুল হওয়া।

Permission-aware key ও cache invalidation policy ছাড়া sensitive output cache কোরো না।

## ৫. Observability কী?

- **Logs:** structured event ও error detail।
- **Metrics:** latency, error rate, token usage, cost।
- **Traces:** request-এর stage-by-stage path।

Correlation/request ID ব্যবহার করো, যাতে API request থেকে provider call ও tool execution trace করা যায়।

~~~mermaid
flowchart TD
    A["Incoming request ID"] --> B["API span"]
    B --> C["Prompt / retrieval span"]
    C --> D["Provider span"]
    D --> E["Tool / database span"]
    E --> F["Response span"]
    B --> G["Logs and metrics"]
    D --> G
    E --> G
    F --> G
~~~

## ৬. LLM-specific Metrics কী track করবে?

- Input/output tokens।
- Model name/version।
- Prompt version।
- Tool call count ও success rate।
- Refusal rate।
- Structured output validation failure।
- Retrieval quality।
- Task success / user feedback।
- Cost per request ও cost per successful task।
- p50/p95 latency।

শুধু average latency নয়; tail latency user experience-এ বড় প্রভাব ফেলতে পারে।

## ৭. Latency Optimization কীভাবে করবে?

- Streaming।
- Simple task-এ smaller/faster model।
- Independent I/O parallelize।
- Retrieval index ও reranking tune।
- Caching।
- Prompt/context reduction।
- Output token limit।
- Repeated tool call deduplicate।

Quality regression যেন না হয়, evaluation চালাও। Fast কিন্তু incorrect response production success নয়।

## ৮. Cost Control কীভাবে করবে?

- Per-user/tenant budget।
- Max output tokens।
- Model routing।
- Cache।
- Duplicate request suppression।
- Context/token budget।
- Usage alert।
- Cost per successful task।

Token count-এর পাশাপাশি retry amplification, failed request cost ও long-tail request monitor করো।

## ৯. Security কীভাবে enforce করবে?

- API key secret manager/environment থেকে।
- Least privilege credentials।
- Input/output validation।
- PII redaction ও data minimization।
- Tenant isolation।
- Audit log।
- Dependency patching।
- Tool execution sandboxing যেখানে দরকার।
- Prompt injection defense-in-depth।
- Sensitive action-এ human approval।

~~~mermaid
flowchart TD
    A["Untrusted input"] --> B["Authentication"]
    B --> C["Authorization"]
    C --> D["Validate and minimize data"]
    D --> E["LLM / tool boundary"]
    E --> F["Validate action and output"]
    F --> G["Audit and respond"]
~~~

Prompt-এ secret রাখা, model output-কে trusted command ধরে execute করা বা retrieved content-কে instruction হিসেবে follow করা নিরাপদ নয়।

## ১০. Docker deployment-এ কী বিবেচনা করবে?

Application ও dependency reproducible image-এ package করা যায়।

- Non-root user।
- Small trusted base image।
- Pinned dependencies।
- Health check।
- Secret injection at runtime।
- Minimal OS packages।
- Image scanning।
- Graceful shutdown।
- CPU/memory limit।

Secret image layer-এ bake করবে না। Build-time ও runtime secret আলাদা রাখো।

## ১১. Model, Prompt এবং Data Versioning কেন দরকার?

Track করো:
- Model/provider ID ও version।
- Prompt template version।
- Embedding model version।
- Vector index version।
- Dataset/evaluation set version।
- Feature/schema version।
- Runtime dependency version।

একটি model বদলালেও embedding space বা prompt behavior বদলাতে পারে। Rollback path রাখো।

## ১২. Deployment Strategy কী?

Canary, shadow traffic বা gradual rollout দিয়ে নতুন prompt/model release evaluate করো।

~~~mermaid
flowchart TD
    A["New model / prompt version"] --> B["Offline evaluation"]
    B --> C{"Pass quality and safety gates?"}
    C -->|না| D["Revise"]
    C -->|হ্যাঁ| E["Shadow / canary traffic"]
    E --> F["Monitor metrics"]
    F --> G{"Regression?"}
    G -->|হ্যাঁ| H["Rollback"]
    G -->|না| I["Gradual rollout"]
~~~

Shadow mode-এ production request copy করে নতুন model evaluate করা যায়, কিন্তু side effect execute করা যাবে না। Canary-তে limited real traffic; rollback criteria আগেই define করো।

## ১৩. Human Review কখন দরকার?

- High-impact decision।
- Low confidence বা insufficient evidence।
- Policy boundary।
- Irreversible action।
- Conflicting source।
- Sensitive user request।

Human review-এর জন্য exact evidence, proposed action ও consequence দেখাও। Reviewer action অনুমোদন করলেও permission/state আবার verify করো।

## ১৪. Production Incident হলে কীভাবে respond করবে?

1. Impact scope ও affected tenant/user নির্ধারণ।
2. Unsafe action বা data exposure বন্ধ।
3. Model/prompt/index version rollback বা feature disable।
4. Logs/traces থেকে root cause।
5. Data/security incident হলে escalation।
6. Regression test ও remediation।
7. Post-incident review ও monitoring improvement।

## Quick Interview Revision

| প্রশ্ন | সংক্ষিপ্ত উত্তর |
|---|---|
| Circuit breaker? | Repeated failure-এ dependency call সাময়িক বন্ধ |
| Backpressure? | Overload-এ incoming work control |
| Cache risk? | Stale data, version mismatch, privacy leak |
| Observability? | Logs, metrics, traces |
| Tail latency? | p95/p99 response time |
| Cost per successful task? | Useful outcome-এর প্রতি total cost |
| Secret management? | Runtime secret injection, never bake into image |
| Shadow deployment? | Copy traffic, no side effects |
| Canary? | Limited real traffic rollout |
| Rollback? | Known-good version restore |
| Human review? | High-impact/uncertain action approval |

## Practice

একটি FastAPI LLM endpoint-এর production checklist তৈরি করো।

Requirements:
1. Pydantic input validation।
2. Authentication/authorization।
3. Timeout ও bounded retry।
4. Rate limit ও concurrency limit।
5. Structured logging ও request ID।
6. Token usage ও latency metrics।
7. Secret environment/secret manager থেকে।
8. Dockerfile: non-root, pinned dependencies, health check।
9. Prompt/model version record।
10. Canary/rollback strategy।
11. Failure injection test।
12. Privacy ও data retention policy।

## Practice করার পদ্ধতি

1. Dependency failure ও overload test করো।
2. Retry, timeout, circuit breaker এবং fallback আলাদা concern হিসেবে design করো।
3. p95 latency ও cost per successful task measure করো।
4. Prompt/model/index version traceable রাখো।
5. Secret ও personal data log/image-এ leak হচ্ছে কি না verify করো।
6. Rollback ও incident response drill করো।
