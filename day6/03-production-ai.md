# Production AI Engineering

## ১. Reliability
LLM provider, database, vector store ও external API সবই fail করতে পারে। Timeout, bounded retry, circuit breaker, fallback ও graceful degradation বিবেচনা করো।

## ২. Retry ও Backoff
Transient error-এ exponential backoff ও jitter। Permanent 4xx/validation error blindly retry নয়। Side-effecting tool-এ idempotency key ব্যবহার করো।

## ৩. Rate Limiting
Provider quota ও per-user abuse control করতে token bucket/leaky bucket বা queue ব্যবহার করা যায়। Concurrency limit ও backpressure রাখো।

## ৪. Caching
Prompt/result cache cost ও latency কমাতে পারে। Cache key-তে model version, prompt version, relevant context ও permission scope বিবেচনা করো। Private user data cross-user cache leak হতে দেবে না।

## ৫. Observability
- Logs: structured events।
- Metrics: latency, error rate, token usage, cost।
- Traces: request-এর stage-by-stage path।
Correlation/request ID ব্যবহার করো।

## ৬. LLM-specific Metrics
Input/output tokens, model name, prompt version, tool calls, refusal rate, structured output validation failure, retrieval quality ও task success track করো।

## ৭. Latency Optimization
Streaming, smaller model for simple task, parallel independent I/O, retrieval tuning, caching ও prompt/context reduction। Quality regression যেন না হয়, evaluation চালাও।

## ৮. Cost Control
Per-user budget, max output tokens, model routing, cache, duplicate request suppression ও usage alert। Cost per successful task track করা token count-এর চেয়ে বেশি useful হতে পারে।

## ৯. Security
- API key secret manager/environment থেকে।
- Least privilege credentials।
- Input/output validation।
- PII redaction।
- Tenant isolation।
- Audit log।
- Dependency patching।
- Tool execution sandboxing যেখানে দরকার।

## ১০. Docker
Application ও dependency reproducible image-এ package করা যায়। Non-root user, small base image, pinned dependencies, health check ও secret injection বিবেচনা করো। Secret image layer-এ bake করবে না।

## ১১. Model/Prompt Versioning
Model ID, prompt template, embedding model, vector index version ও dataset version track করো। Rollback path রাখো।

## ১২. Deployment Strategy
Canary, shadow traffic বা gradual rollout দিয়ে নতুন prompt/model release evaluate করো। Failure হলে rollback।

## ১৩. Human Review
High-impact decision, low confidence, policy boundary বা irreversible action-এ human review রাখো।

## Practice
একটি FastAPI LLM endpoint-এর production checklist তৈরি করো। Timeout, rate limit, structured logging, Pydantic validation ও Dockerfile যোগ করো।
