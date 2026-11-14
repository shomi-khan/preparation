# LLM Application Engineering

## ১. LLM API call-এর lifecycle
Application input নেয়, prompt/context তৈরি করে, provider API call করে, response parse ও validate করে, তারপর result user-কে দেয়। প্রতিটি ধাপে failure handling দরকার।

## ২. Timeout ও Retry
Transient network error বা rate limit-এ bounded retry করা যায়। Exponential backoff ও jitter ব্যবহার করো। সব error retry করা উচিত নয়; invalid request retry করলে লাভ নেই।

## ৩. Idempotency
একই logical request retry হলে duplicate side effect যেন না হয়। Tool/action execute করার সময় idempotency key বা deduplication দরকার হতে পারে।

## ৪. Rate Limit ও Concurrency
Provider quota, per-user limit ও application concurrency limit মানতে হবে। Queue/backpressure ব্যবহার করা যায়।

## ৫. Streaming
Streaming partial token পাঠায়, ফলে perceived latency কমতে পারে। তবে partial output-কে validated final structured result ধরে নেওয়া যাবে না।

## ৬. Cost Estimation
Input ও output token আলাদা rate-এ bill হতে পারে। Prompt size, retrieved context, model choice, cache ও output limit cost-এ প্রভাব ফেলে।

## ৭. Logging ও Privacy
Request ID, model version, latency, token usage, status ও error class log করা উপকারী। Password, API key, personal data বা sensitive prompt indiscriminately log করবে না।

## ৮. Example: Pydantic validation boundary
```python
from pydantic import BaseModel, Field

class GenerateRequest(BaseModel):
    prompt: str = Field(min_length=1, max_length=10_000)
    max_tokens: int = Field(default=300, ge=1, le=2000)

def prepare_prompt(request: GenerateRequest) -> str:
    return request.prompt.strip()
```

## ৯. Model abstraction
Provider SDK-কে application domain থেকে আলাদা interface-এর পেছনে রাখলে provider swap ও testing সহজ হয়। তবে premature abstraction-এ অতিরিক্ত complexity আনবে না।

## ১০. Testing
- Unit test: prompt construction, validation, parser।
- Integration test: provider call contract।
- Evaluation test: answer quality ও task success।
- Load test: latency, concurrency ও rate limit।
- Failure test: timeout, malformed output, provider unavailable।

## ১১. Production checklist
- Timeout
- Bounded retry with backoff
- Rate limiting
- Input/output validation
- Sensitive data handling
- Observability
- Cost/latency monitoring
- Prompt/model versioning
- Fallback বা graceful failure

## Practice
একটি small support-ticket classifier বানাও। Input Pydantic দিয়ে validate করো, LLM response schema-তে parse করো, timeout/retry policy implement করো এবং token/latency metadata log করো।
