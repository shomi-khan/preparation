# Day 4 Interview Questions and Answers

## LLM Fundamentals

### ১. LLM কী?
বড় text corpus-এ train করা neural network, যা context থেকে পরবর্তী token predict করে language task সম্পাদন করে।

### ২. Token কী এবং কেন গুরুত্বপূর্ণ?
Model text token ID হিসেবে process করে। Context limit ও API cost token count-এর উপর নির্ভর করতে পারে।

### ৩. Embedding কী?
Text বা token-এর dense vector representation। Semantic retrieval ও similarity calculation-এ ব্যবহৃত হয়।

### ৪. Transformer কী?
Attention mechanism-ভিত্তিক neural architecture, যা sequence-এর tokenগুলোর সম্পর্ক model করে।

### ৫. Self-attention কী?
একটি token-এর representation তৈরিতে অন্য token-এর information weightedভাবে combine করে। Query, Key ও Value projection ব্যবহৃত হয়।

### ৬. Context window কী?
একটি model request-এ process করতে পারে এমন token context-এর সীমা। Input ও output উভয় budget বিবেচ্য।

### ৭. Temperature কী করে?
Sampling distribution-এর randomness পরিবর্তন করে। এটি factuality control-এর guarantee নয়।

### ৮. Hallucination কেন হয়?
LLM next-token generation করে; সবসময় external fact verify করে না। Fluent output ভুলও হতে পারে।

### ৯. RAG ও Fine-tuning-এর পার্থক্য কী?
RAG inference-এর সময় external knowledge retrieve করে; fine-tuning model weights update করে। Fresh/private knowledge যোগে RAG, behavior/style adaptation-এ fine-tuning উপযোগী হতে পারে।

## Prompt Engineering

### ১০. Zero-shot ও Few-shot prompting কী?
Zero-shot-এ example নেই; few-shot-এ desired input/output example দেওয়া হয়।

### ১১. Structured output কেন দরকার?
Application parsing ও validation predictable হয়। তবে schema-valid output factually correct হবে এমন নিশ্চয়তা নেই।

### ১২. Function calling কী?
Model tool-এর নাম ও argument structure প্রস্তাব করে; application validate করে actual function execute করে।

### ১৩. Prompt injection কী?
Untrusted content model-কে instruction boundary ভাঙাতে বা unauthorized action করাতে চেষ্টা করে।

### ১৪. Prompt injection কীভাবে mitigate করবে?
Least privilege, tool allowlist, input separation, argument validation, human approval, untrusted retrieved text handling এবং sensitive action-এর server-side authorization।

### ১৫. Prompt engineering কি model-এর ভুল পুরোপুরি দূর করতে পারে?
না। Retrieval, validation, verification, abstention ও evaluation দরকার; তবুও residual risk থাকে।

## Application Engineering

### ১৬. LLM API-তে timeout কেন দরকার?
Provider/network slow বা stuck হলে request resource অনির্দিষ্টকাল আটকে না রাখতে।

### ১৭. Retry কীভাবে design করবে?
Transient error-এ bounded exponential backoff ও jitter; invalid request বা permanent auth error retry নয়। Side effect থাকলে idempotency দরকার।

### ১৮. Streaming-এর সুবিধা ও trade-off কী?
User দ্রুত partial output দেখে; কিন্তু partial content validate করা কঠিন এবং connection lifecycle manage করতে হয়।

### ১৯. LLM cost কীভাবে কমাবে?
Prompt/context ছোট করা, প্রয়োজনমতো model নির্বাচন, output limit, caching, retrieval optimization ও redundant call কমানো।

### ২০. কী log করবে?
Request ID, model/version, latency, status, token usage ও error category। Sensitive data redaction করতে হবে।

### ২১. LLM response কীভাবে validate করবে?
Structured schema parse, field constraints, domain rule ও প্রয়োজনমতো source verification। শুধু JSON parse যথেষ্ট নয়।

### ২২. Provider abstraction কেন?
Business logic-কে provider SDK থেকে আলাদা করে testing ও provider change সহজ করতে। অতিরিক্ত abstraction এড়াতে হবে।

## Scenario Questions

### ২৩. Model valid JSON দিল কিন্তু ভুল category দিল। কী করবে?
Schema validation syntax/shape যাচাই করে, semantic correctness নয়। Evaluation dataset, explicit taxonomy, confidence/abstention এবং human review যোগ করব।

### ২৪. API rate limit দিলে?
Provider retry guidance মেনে backoff, request queue, concurrency control ও per-user quota প্রয়োগ করব।

### ২৫. LLM response-এ private data leak হলে?
Data minimization, access control, retrieval permission filtering, output policy, log redaction ও incident response পরীক্ষা করব।

### ২৬. Prompt update-এর পরে quality কমে গেল। কীভাবে বুঝবে?
Versioned evaluation set-এ old/new prompt compare, regression case inspect, task success/latency/cost metrics দেখা এবং gradual rollout/rollback।

### ২৭. Model-এর উত্তর verify করবে কীভাবে?
Trusted sources থেকে retrieval, citation checking, deterministic business rule বা external system verification ব্যবহার করব। Citation উপস্থিত থাকলেই সত্য ধরে নেব না।

## Coding Discussion

### ২৮. Pydantic model কেন?
Untrusted input ও model output-কে expected schema ও constraints অনুযায়ী validate করতে।

### ২৯. Async LLM request-এ কী সতর্কতা?
Async-compatible client, timeout, bounded concurrency, cancellation, retry policy এবং provider quota মানা।

### ৩০. LLM feature-এর unit test কী হবে?
Prompt construction, schema parsing, validation, retry/error mapping, tool authorization এবং deterministic post-processing test করা যায়। Model quality আলাদা evaluation suite-এ মাপতে হবে।


---

## Additional Interview Questions: Transformer Internals and Production Design

### ৩১. Token count word count-এর সমান নয় কেন?

Tokenizer text-কে model-specific subword/token unit-এ ভাগ করে। একটি word একাধিক token হতে পারে, punctuation আলাদা token হতে পারে। Cost ও context budget-এর জন্য target model-এর tokenizer ব্যবহার করো।

### ৩২. Encoder-only, decoder-only এবং encoder-decoder model-এর পার্থক্য কী?

Encoder-only model input sequence representation-এ জোর দেয়; decoder-only model autoregressive next-token generation করে; encoder-decoder model input encode করে output sequence generate করে। Use case architecture অনুযায়ী আলাদা।

### ৩৩. Query, Key এবং Value কী?

Self-attention-এ Query বলে token কী information খুঁজছে, Key similarity score নির্ধারণে ব্যবহৃত হয়, Value হলো weighted combine হওয়া content representation। Attention weights দিয়ে value vector combine করা হয়।

### ৩৪. Causal masking কেন দরকার?

Autoregressive training-এ position t যেন future token দেখতে না পারে। Causal mask future positions block করে, যাতে model শুধু আগের context থেকে next token predict করে।

### ৩৫. Positional encoding কেন দরকার?

Self-attention নিজে token order encode করে না। Positional encoding/embedding বা RoPE-এর মতো mechanism sequence position information যোগ করে।

### ৩৬. Pretraining ও supervised fine-tuning-এর পার্থক্য কী?

Pretraining বড় corpus থেকে general language/statistical pattern শেখে, সাধারণত next-token objective-এ। SFT instruction-response example দিয়ে desired task-following behavior tune করে।

### ৩৭. RLHF এবং DPO কী?

RLHF human preference feedback ব্যবহার করে model behavior align করার training approach; সাধারণ pipeline-এ reward model ও reinforcement learning থাকতে পারে। DPO preference pair থেকে direct preference optimization করে। দুটোই preference alignment-এর পদ্ধতি, কিন্তু objective/optimization আলাদা।

### ৩৮. Context window বড় হলেই answer quality ভালো হয়?

না। বড় context বেশি information নিতে পারে, কিন্তু irrelevant/duplicate text attention ও cost বাড়ায়। Relevant context selection, ordering এবং budget management গুরুত্বপূর্ণ।

### ৩৯. Temperature কি factuality control করে?

না। Temperature sampling randomness/distribution sharpness নিয়ন্ত্রণ করে। কম temperature output বেশি predictable করতে পারে, কিন্তু factual correctness guarantee করে না।

### ৪০. KV cache কীভাবে inference speed up করে?

Autoregressive decoding-এ আগের token-এর Key/Value projection cache করে রাখে। নতুন token generate করার সময় আগের K/V আবার compute করতে হয় না। Trade-off হলো memory usage।

### ৪১. Quantization কী এবং trade-off কী?

Model weight lower-bit precision-এ represent করে memory footprint ও inference cost কমানো যায়। Speed/memory improvement-এর বিনিময়ে quality/accuracy কমতে পারে; hardware ও method অনুযায়ী ফল বদলায়।

### ৪২. JSON mode এবং JSON Schema structured output-এর পার্থক্য কী?

JSON mode syntactically valid JSON output encourage/guarantee করতে পারে (provider-specific)। JSON Schema structured output field/type/shape constrain করে। দুই ক্ষেত্রেই application-side semantic validation দরকার।

### ৪৩. Schema-valid output কি trusted?

না। Schema শুধু structure/type/constraint যাচাই করে। Factual correctness, authorization, business invariant ও source grounding আলাদাভাবে যাচাই করতে হবে।

### ৪৪. Indirect prompt injection কী?

Retrieved webpage, document, email বা tool output-এর ভিতরে malicious instruction থাকা এবং model সেটিকে instruction হিসেবে follow করার চেষ্টা করা। Retrieved content-কে untrusted data ধরে application-side tool permission ও action validation enforce করতে হবে।

### ৪৫. Prompt injection-এর বিরুদ্ধে delimiter যথেষ্ট নয় কেন?

Delimiter text boundary স্পষ্ট করে, কিন্তু model-এর instruction-following behavior-এর বিরুদ্ধে cryptographic/security boundary নয়। Least privilege, tool allowlist, server-side authorization, argument validation ও human confirmation দরকার।

### ৪৬. Tool calling-এ model কি নিজে tool execute করে?

সাধারণ application flow-তে model tool name ও arguments propose করে। Application validates permission ও arguments, তারপর tool execute করে এবং result model-কে দেয়। Execution boundary application-এর হাতে থাকা উচিত।

### ৪৭. LLM tool action-এ idempotency কেন জরুরি?

Timeout বা retry-তে একই action একাধিকবার execute হতে পারে। Payment, email বা account update-এর মতো side effect-এ idempotency key/deduplication duplicate action আটকায়।

### ৪৮. Streaming output-এর partial chunk কি সঙ্গে সঙ্গে trusted?

না। Stream মাঝপথে fail হতে পারে, output incomplete বা malformed হতে পারে। Final completion ও schema/semantic validation না হওয়া পর্যন্ত structured result হিসেবে consume কোরো না।

### ৪৯. LLM application-এর cost কীভাবে estimate করবে?

Input ও output token আলাদা provider rate-এ multiply করে, সঙ্গে retry, model choice, context length, cache hit এবং request volume হিসাব করবে। Current provider pricing ব্যবহার করতে হবে।

### ৫০. LLM response caching-এর ঝুঁকি কী?

Stale response, user/tenant data leak, prompt/model version mismatch এবং non-deterministic response behavior। Cache key-তে tenant scope ও version রাখো; TTL/invalidation policy define করো।

### ৫১. LLM evaluation dataset কেমন হওয়া উচিত?

Representative real-world input, edge case, adversarial/prompt injection case, expected output/rubric এবং held-out regression set থাকা উচিত। শুধু happy path বা কয়েকটি anecdotal prompt যথেষ্ট নয়।

### ৫২. LLM-as-a-judge-এর সীমাবদ্ধতা কী?

Judge model bias, prompt sensitivity, position bias ও একই ধরনের model-এর shared blind spot থাকতে পারে। Human review, deterministic checks এবং multiple evaluation methods দিয়ে judge reliability validate করো।

### ৫৩. LLM application-এ কী observability রাখবে?

Request/trace ID, provider/model version, prompt version, latency, token usage, retry count, status/error category এবং validation outcome। Sensitive prompt বা personal data indiscriminately log করা যাবে না।

## LLM Application Flow

~~~mermaid
flowchart TD
    A["User input"] --> B["Validate and authorize"]
    B --> C["Prompt / context construction"]
    C --> D["Model API"]
    D --> E["Structured parsing"]
    E --> F["Semantic and policy validation"]
    F --> G["Response / human review"]
    D --> H["Usage, latency, error telemetry"]
    E --> H
~~~

## Oral Practice

প্রতিটি প্রশ্নের উত্তরে mechanism, practical use, failure mode এবং security/cost trade-off explain করো। Tool calling বা structured output নিয়ে কথা বললে application-side validation boundary স্পষ্ট করো।
