# Day 4 Practical Exercises

এই chapter-এর লক্ষ্য একটি ছোট LLM feature design, implement এবং evaluate করা। শুধু model response পাওয়া success নয়; structured correctness, security boundary, reliability, latency এবং cost-ও বিবেচনা করতে হবে।

## Exercise 1: Tokenization and Context Budget

একটি বাংলা ও ইংরেজি paragraph target model-এর tokenizer দিয়ে encode করো।

Tasks:
- Token count, character count ও word count compare করো।
- একটি prompt-এ system instruction, history, retrieved context, user input ও output reserve-এর budget ভাগ করো।
- Context limit ছাড়ালে কোন অংশ truncate/summarize করবে policy লিখো।

## Exercise 2: Prompt Design

একটি support ticket classify করার zero-shot prompt তৈরি করো। এরপর ৩-৫টি representative few-shot example যোগ করে output consistency compare করো।

Requirements:
- Allowed category ও priority define করো।
- Missing evidence হলে human review flag করো।
- User message-কে untrusted data হিসেবে treat করো।
- Prompt version record করো।

## Exercise 3: Structured Output

Ticket classification-এর Pydantic schema বানাও।

Test:
- Invalid category।
- Missing field।
- Wrong field type।
- Malformed JSON।
- Schema-valid কিন্তু semantically invalid response।

Schema validation ও domain validation আলাদা implement করো।

## Exercise 4: Prompt Injection

Customer message ও retrieved document-এ "ignore previous instructions" এবং secret বের করার request যোগ করো।

Tasks:
- Direct ও indirect injection case তৈরি করো।
- Tool allowlist ও permission check implement করো।
- Prompt-only defense কেন যথেষ্ট নয় explain করো।
- Sensitive action-এর আগে human confirmation যোগ করো।

~~~mermaid
flowchart TD
    A["Untrusted message / document"] --> B["LLM"]
    C["Trusted application policy"] --> B
    B --> D["Tool call proposal"]
    D --> E["Server-side validation"]
    E --> F{"Authorized?"}
    F -->|হ্যাঁ| G["Execute limited tool"]
    F -->|না| H["Reject / require approval"]
~~~

## Exercise 5: LLM API Wrapper

একটি wrapper implement করো:
- Input validation।
- Timeout।
- Bounded retry with backoff/jitter।
- Error classification।
- Response schema validation।
- Request ID।
- Token/latency metadata।
- Cancellation handling।

Retry সব error-এ করবে না। Provider-specific rate limit এবং transient error আলাদা করো।

## Exercise 6: Evaluation Set

কমপক্ষে ২০টি representative ticket লিখে expected category/priority annotate করো।

Measure:
- Task accuracy।
- Per-class precision/recall।
- Schema validity rate।
- Human-review flag behavior।
- Prompt injection robustness।
- Latency ও token cost।

Prompt change-এর আগে/পরে compare করো। Test example দিয়ে prompt tune করলে evaluation contamination হতে পারে; separate development ও held-out eval set রাখো।

## Exercise 7: Streaming

একটি streaming response UI-তে দেখাও।

Test:
- Partial chunk arrival।
- Client disconnect।
- Provider error midway।
- Stream completion।
- Incomplete JSON response।

Partial output-কে final validated object হিসেবে ব্যবহার কোরো না।

## Exercise 8: Cost and Context Optimization

একই task-এর জন্য:
- Long prompt বনাম compact prompt।
- Full history বনাম summarized history।
- Different model tier।
- Output token limit।
- Retrieval context size।

Token cost, latency ও task quality compare করো। শুধু cheapest output বেছে নিও না; quality constraint বজায় রাখতে হবে।

## Exercise 9: Caching

একটি response cache design করো।

Consider:
- Cache key-তে user/tenant scope।
- Prompt ও model version।
- TTL এবং invalidation।
- User-specific/private content।
- Non-deterministic response cache করলে behavior।
- Stale knowledge risk।

## Exercise 10: Mini Support-Ticket Classifier

একটি runnable Python project বানাও:

~~~mermaid
flowchart TD
    A["HTTP request"] --> B["Pydantic validation"]
    B --> C["Authentication / authorization"]
    C --> D["Prompt builder"]
    D --> E["LLM provider adapter"]
    E --> F["Parse structured output"]
    F --> G["Semantic / policy validation"]
    G --> H["Response or human review"]
    E --> I["Latency / token / error telemetry"]
    F --> I
~~~

Requirements:
- Provider SDK-কে domain logic থেকে isolate করো।
- Unit test-এ fake provider ব্যবহার করো।
- Timeout, retry ও concurrency limit রাখো।
- Sensitive text indiscriminately log কোরো না।
- Prompt/model version record করো।
- Human review/fallback path রাখো।

## Deliverables

- Runnable Python project।
- Prompt template ও schema।
- Unit/contract/security test cases।
- Evaluation dataset ও report।
- Cost/latency comparison।
- Architecture diagram।
- ৩-৫ মিনিটের architecture walkthrough।

## Self-review

- [ ] Output schema validate করেছি।
- [ ] Semantic correctness আলাদা যাচাই করেছি।
- [ ] Hallucination-এর সীমাবদ্ধতা বুঝেছি।
- [ ] Prompt injection-কে security problem হিসেবে নিয়েছি।
- [ ] Tool permission সীমিত করেছি।
- [ ] Sensitive data log করিনি।
- [ ] Timeout/retry implement করেছি।
- [ ] Retry idempotency বিবেচনা করেছি।
- [ ] Prompt evaluation করেছি।
- [ ] Token/cost budget estimate করেছি।
- [ ] Streaming partial output handle করেছি।

## Completion Criteria

1. Valid ও invalid request behavior predictable।
2. Model output schema এবং business rule pass করে।
3. Unauthorized tool call reject হয়।
4. Timeout/retry/failure behavior test করা হয়েছে।
5. Quality, latency ও cost report আছে।
6. Prompt/model version ও evaluation dataset track করা হয়েছে।
