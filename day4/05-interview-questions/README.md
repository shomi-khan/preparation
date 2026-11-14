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
