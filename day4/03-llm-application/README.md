# LLM Application Engineering

একটি production LLM application শুধু model API call নয়। Input validation, prompt construction, provider request, timeout/retry, output parsing, authorization, observability এবং cost control সব মিলিয়ে application তৈরি হয়। Model-এর response probabilistic হওয়ায় failure path ও evaluation design শুরু থেকেই ভাবতে হয়।

## ১. LLM API call-এর lifecycle কী?

সাধারণ lifecycle:

1. User input receive ও validate।
2. Authentication/authorization।
3. Prompt এবং relevant context তৈরি।
4. Token budget ও model configuration নির্ধারণ।
5. Provider API call।
6. Response parse ও validate।
7. Domain rule / safety check।
8. Result return এবং metadata record।

~~~mermaid
flowchart TD
    A["User request"] --> B["Validate input and permission"]
    B --> C["Build prompt / context"]
    C --> D["Check token and cost budget"]
    D --> E["Call model provider"]
    E --> F["Parse and validate output"]
    F --> G["Apply business / safety rules"]
    G --> H["Return response"]
    E --> I["Record latency, usage, status"]
    F --> I
    H --> I
~~~

প্রতিটি boundary-তে failure handling দরকার। Invalid input provider-এ পাঠানোর আগে reject করো।

## ২. Application architecture কীভাবে ভাগ করা যায়?

একটি maintainable application-এ provider SDK-কে domain logic থেকে আলাদা রাখো।

~~~mermaid
flowchart TD
    A["API / UI layer"] --> B["Application service"]
    B --> C["Prompt and context builder"]
    B --> D["Validation / policy"]
    C --> E["LLM provider adapter"]
    E --> F["External model API"]
    B --> G["Observability / cost tracking"]
~~~

Provider abstraction useful, কিন্তু premature generic framework বানিয়ে complexity বাড়িও না। একাধিক provider বা testability requirement থাকলে interface তৈরি করো।

## ৩. Timeout ও Retry কীভাবে design করব?

Transient network error, 429 rate limit বা temporary provider failure-এ bounded retry করা যায়। Exponential backoff ও jitter ব্যবহার করো।

~~~python
import random
import time

def backoff_delay(attempt: int, base: float = 0.5, cap: float = 8.0) -> float:
    upper = min(cap, base * (2 ** attempt))
    return random.uniform(0, upper)
~~~

সব error retry করা উচিত নয়:
- Invalid request / authentication error: সাধারণত retry নয়।
- Rate limit / temporary server error: provider guidance অনুযায়ী retry।
- Timeout: operation execute হয়েছে কি না জানা নাও থাকতে পারে।

~~~mermaid
flowchart TD
    A["Provider request"] --> B{"Response"}
    B -->|Success| C["Return result"]
    B -->|Transient failure| D{"Retry budget left?"}
    D -->|হ্যাঁ| E["Backoff + jitter"]
    E --> A
    D -->|না| F["Graceful failure / fallback"]
    B -->|Permanent failure| G["Do not retry blindly"]
~~~

## ৪. Idempotency কেন গুরুত্বপূর্ণ?

একই logical request retry হলে duplicate side effect যেন না হয়। Tool/action execute করার সময় idempotency key বা deduplication দরকার হতে পারে।

LLM generation itself অনেক সময় non-deterministic; একই prompt retry করলে ভিন্ন output আসতে পারে। Payment, email send, account update-এর মতো side effect-এ generation result-এর চেয়ে action execution idempotency বেশি গুরুত্বপূর্ণ।

## ৫. Rate Limit, Concurrency এবং Backpressure কীভাবে manage করব?

Provider quota, per-user limit ও application concurrency limit মানতে হবে। Queue/backpressure ব্যবহার করা যায়।

~~~mermaid
flowchart LR
    A["Incoming requests"] --> B["Bounded queue"]
    B --> C["Concurrency limiter"]
    C --> D["Provider API"]
    D --> E["Results"]
    B --> F["Reject / defer when overloaded"]
~~~

শুধু provider rate limit নয়, database connection, token budget, user quota এবং downstream service capacity-ও বিবেচনা করো।

## ৬. Streaming কী? এর trade-off কী?

Streaming partial token/event পাঠায়, ফলে perceived latency কমতে পারে। কিন্তু partial output-কে validated final structured result ধরে নেওয়া যাবে না।

- UI-তে progressive response দেখাতে উপকারী।
- Client disconnect হলে upstream request cancel করা যায় কি না ভাবতে হবে।
- Partial response মাঝপথে থেমে যেতে পারে।
- Structured JSON পুরোপুরি complete হওয়ার আগে parse করা নিরাপদ নয়।
- Streaming response-এ moderation/validation strategy আলাদা হতে পারে।

~~~mermaid
sequenceDiagram
    participant C as Client
    participant A as Application
    participant M as Model
    C->>A: Request
    A->>M: Generate stream
    loop Partial chunks
        M-->>A: Token / event
        A-->>C: Stream chunk
    end
    M-->>A: Stream complete
    A->>A: Final validation / completion handling
~~~

## ৭. Cost Estimation কীভাবে করব?

Input ও output token আলাদা rate-এ bill হতে পারে। Prompt size, retrieved context, model choice, cache, output limit এবং retry cost-এ প্রভাব ফেলে।

Approximate cost:

~~~text
input_cost  = input_tokens  / 1_000_000 × input_rate_per_million
output_cost = output_tokens / 1_000_000 × output_rate_per_million
total_cost  = input_cost + output_cost
~~~

Provider pricing বদলায়; production budget-এ current provider rate ব্যবহার করো। শুধু average cost নয়, p95 request cost এবং retry amplification-ও monitor করো।

## ৮. Context ও Token Budget কীভাবে control করব?

- Conversation history truncate/summarize policy।
- Retrieved chunk relevance ও maximum count।
- Prompt template token overhead।
- Output token reserve।
- Model context window।
- User-level token/cost quota।

Context বেশি দিলেই response ভালো হবে এমন নয়। Relevant context নির্বাচন করো এবং duplicate/irrelevant text বাদ দাও।

## ৯. Caching কখন ব্যবহার করব?

Exact prompt/result cache, embedding cache বা retrieval cache latency ও cost কমাতে পারে। কিন্তু:
- User-specific data cross-user leak হতে পারে।
- Prompt/model/version change হলে cache invalidation দরকার।
- Freshness-sensitive result-এর TTL দরকার।
- Non-deterministic generation cache করলে behavior বদলে যায়।

Cache key-তে tenant/user scope, prompt version, model version এবং relevant input fingerprint বিবেচনা করো।

## ১০. Logging ও Privacy কীভাবে balance করব?

Useful metadata:
- Request ID / trace ID।
- Model/provider version।
- Prompt template version।
- Latency ও token usage।
- Status ও error class।
- Retry count।
- Validation outcome।

Password, API key, personal data বা sensitive prompt indiscriminately log করবে না। Data minimization, redaction, retention policy ও access control রাখো।

## ১১. Pydantic validation boundary কীভাবে ব্যবহার করব?

~~~python
from pydantic import BaseModel, Field, field_validator

class GenerateRequest(BaseModel):
    prompt: str = Field(min_length=1, max_length=10_000)
    max_tokens: int = Field(default=300, ge=1, le=2000)

    @field_validator("prompt")
    @classmethod
    def trim_prompt(cls, value: str) -> str:
        value = value.strip()
        if not value:
            raise ValueError("prompt cannot be blank")
        return value
~~~

Validation request-এর size ও shape constrain করে। কিন্তু prompt injection, authorization, rate limit ও business permission আলাদাভাবে enforce করতে হবে।

## ১২. Model abstraction কী?

Provider SDK-কে application domain থেকে আলাদা interface-এর পেছনে রাখলে provider swap ও testing সহজ হয়। তবে abstraction-এর scope বাস্তব requirement অনুযায়ী রাখো।

~~~python
from typing import Protocol

class TextGenerator(Protocol):
    async def generate(self, prompt: str) -> str:
        ...
~~~

Different provider-এর streaming, tool call, token usage ও structured output capability এক নয়। Interface-এ common capability expose করো; provider-specific feature লুকিয়ে ফেলো না।

## ১৩. Fallback এবং Graceful Degradation কী?

Provider unavailable বা quota exhausted হলে:
- Cached safe response ব্যবহার করা।
- Smaller/secondary model-এ fallback।
- Human review queue।
- Feature temporarily disable।
- Clear retry-later response।

Fallback-এ security, quality ও cost trade-off evaluate করো। Lower-quality model-এ silent fallback করে high-risk decision নেওয়া উচিত নয়।

## ১৪. Testing কীভাবে করব?

| Test type | কী পরীক্ষা করবে |
|---|---|
| Unit test | Prompt construction, validation, parser |
| Contract test | Provider request/response shape |
| Integration test | Provider call behavior |
| Evaluation test | Answer quality ও task success |
| Load test | Latency, concurrency ও rate limit |
| Failure test | Timeout, malformed output, provider unavailable |
| Security test | Prompt injection, data leakage, unauthorized tool |

LLM output non-deterministic হতে পারে, তাই exact string equality-এর বদলে schema, rubric, semantic assertion ও representative eval dataset ব্যবহার করো।

## ১৫. Production checklist

- Input/output validation
- Authentication ও authorization
- Timeout
- Bounded retry with backoff/jitter
- Idempotency for side effects
- Rate limiting ও bounded concurrency
- Token/cost budget
- Sensitive data handling
- Observability
- Prompt/model versioning
- Evaluation regression test
- Fallback বা graceful failure
- Monitoring ও rollback

## Quick Interview Revision

| প্রশ্ন | সংক্ষিপ্ত উত্তর |
|---|---|
| LLM API lifecycle? | Validate → prompt/context → call → parse → policy → respond |
| Retry সব error-এ? | না, transient error-এ bounded retry |
| Idempotency? | Retry-তে duplicate side effect আটকানো |
| Backpressure? | Overload-এ incoming work control |
| Streaming risk? | Partial output final validated data নয় |
| Cost কী দিয়ে হয়? | Input/output token, model rate, retry/context |
| Cache risk? | Stale result ও cross-user data leak |
| Logging privacy? | Minimize, redact, access-control |
| Provider abstraction? | SDK ও domain logic আলাদা |
| Fallback? | Failure-এ safe degraded path |
| LLM test? | Unit, contract, eval, load, failure, security |

## Practice

একটি small support-ticket classifier বানাও:
1. Input Pydantic দিয়ে validate করো।
2. Prompt template version করো।
3. LLM response schema-তে parse করো।
4. Timeout ও bounded retry policy implement করো।
5. Token usage/latency metadata log করো, sensitive text নয়।
6. Rate limit ও concurrency limit যোগ করো।
7. Invalid schema ও provider timeout simulate করো।
8. Duplicate action request-এ idempotency behavior test করো।
9. Prompt injection test case যোগ করো।
10. Fallback/human review path document করো।

## Practice করার পদ্ধতি

1. Happy path ছাড়াও timeout, invalid output, cancellation ও rate limit test করো।
2. Provider error code অনুযায়ী retry policy নির্ধারণ করো।
3. Token/cost budget request পাঠানোর আগেই estimate করো।
4. LLM output-কে untrusted input হিসেবে validate করো।
5. Observability-তে useful metadata রাখো, sensitive content নয়।
