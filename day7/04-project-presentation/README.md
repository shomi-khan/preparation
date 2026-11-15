# Project Presentation Guide

Interview-এ project presentation-এর উদ্দেশ্য শুধু feature দেখানো নয়। Interviewer বুঝতে চায় তুমি problem বুঝেছ কি না, architecture কেন বেছেছ, কী trade-off নিয়েছ, কীভাবে quality মেপেছ এবং failure হলে কী করবে।

Suggested project: **Document Q&A with RAG**। নিজের বাস্তব project অনুযায়ী content adapt করবে। Implement না করা feature, metric বা production scale claim করবে না।

## ১. ৫-১০ মিনিটের Presentation Structure

| Section | Time | কী বলবে |
|---|---:|---|
| Problem | 45 sec | User problem ও why it matters |
| Requirements | 45 sec | Functional/non-functional constraints |
| Architecture | 90 sec | Ingestion ও query flow |
| Key decisions | 90 sec | Chunking, embedding, retrieval, model |
| Reliability/security | 60 sec | ACL, timeout, failure, injection |
| Evaluation | 60 sec | Dataset, metrics, evidence |
| Limitations/next steps | 45 sec | Honest gap ও improvement |

## ২. Problem কীভাবে explain করবে?

উত্তর দাও:
- কার problem solve করছ?
- Existing workflow-এ কী pain point?
- কেন keyword search বা manual process যথেষ্ট নয়?
- Success কীভাবে measure করবে?

Problem statement user outcome-এ focus করো, technology list দিয়ে শুরু কোরো না।

## ৩. Requirements কীভাবে present করবে?

উদাহরণ:
- Document type: PDF/Markdown/HTML।
- Source citation/page reference।
- Tenant/user access control।
- Unanswerable query-তে abstention।
- Latency/cost constraints।
- Document update/delete support।

তোমার implementation-এ যা আছে আর future requirement যা, আলাদা করে বলবে।

## ৪. Architecture কীভাবে দেখাবে?

~~~mermaid
flowchart TD
    U["User"] --> API["API / Auth"]
    API --> Q["Query service"]
    Q --> RET["ACL-aware retrieval"]
    RET --> CTX["Context builder"]
    CTX --> LLM["LLM"]
    LLM --> VAL["Output / citation validation"]
    VAL --> U

    DOC["Documents"] --> ING["Ingestion worker"]
    ING --> PARSE["Parse / chunk"]
    PARSE --> EMB["Embedding"]
    EMB --> IDX["Vector / search index"]
~~~

প্রতিটি component-এর responsibility explain করো:
- API/authentication।
- Ingestion worker।
- Object storage ও metadata store।
- Parser/chunker।
- Embedding/index।
- Retrieval/reranker।
- Context builder।
- LLM ও output validation।

## ৫. Key Design Decisions কীভাবে justify করবে?

প্রতিটি decision-এর জন্য:
1. Problem/constraint কী ছিল?
2. Alternative কী ছিল?
3. কেন এই option বেছেছ?
4. Trade-off কী?
5. কীভাবে measure/test করেছ?

Example:
- Chunking strategy কেন?
- Embedding model কেন?
- Vector বনাম hybrid retrieval কেন?
- top-k কীভাবে select?
- Reranker দরকার কি?
- Context budget কীভাবে manage?
- Citation metadata কোথায় preserve?

## ৬. Reliability and Security কীভাবে explain করবে?

Discuss:
- Input validation।
- Timeout ও bounded retry।
- Provider rate limit।
- Tenant ACL।
- Prompt injection।
- Secret management।
- Sensitive log redaction।
- Document update/delete।
- Cache permission scope।
- Human review/abstention।

Security শুধু prompt instruction দিয়ে enforce হয় না। Application-side authorization ও tool boundary দেখাও।

## ৭. Evaluation কীভাবে present করবে?

নিজস্ব test question set তৈরি করো। Report করো:
- Retrieval Recall@k / Precision@k।
- MRR/nDCG (যদি label থাকে)।
- Answer correctness/faithfulness।
- Citation correctness।
- No-answer behavior।
- Latency p50/p95।
- Token/cost per query।

~~~mermaid
flowchart TD
    A["Evaluation queries + labels"] --> B["Retrieval metrics"]
    A --> C["Answer quality metrics"]
    A --> D["Security / ACL tests"]
    A --> E["Latency / cost"]
    B --> F["Baseline comparison"]
    C --> F
    D --> F
    E --> F
~~~

Metric না মেপে "highly accurate" বা "production-ready" দাবি কোরো না। Sample size, evaluation limitation ও manual review-এর ভূমিকা বলো।

## ৮. Limitations কীভাবে বলবে?

সৎভাবে বলো:
- OCR বা complex table extraction limitation।
- Ambiguous/unanswerable query।
- Multilingual retrieval quality।
- Latency ও provider dependency।
- Index freshness delay।
- Citation support gap।
- ACL edge case।
- Cost/scale limitation।

Limitation বলার সঙ্গে mitigation বা next experiment প্রস্তাব করো।

## ৯. Future Improvements কীভাবে prioritize করবে?

Possible improvements:
- Better OCR/layout parsing।
- Hybrid retrieval ও reranker।
- Larger evaluation dataset।
- Better no-answer calibration।
- Permission-aware cache।
- Observability/tracing।
- Re-index/delete workflow।
- Human review dashboard।

সব feature একসঙ্গে roadmap-এ দিও না। Quality/security/latency evidence অনুযায়ী priority দাও।

## ১০. Demo Plan

Live demo-এর আগে:
- Sample documents ready।
- Index complete ও version verified।
- At least 3 answerable questions।
- At least 2 unanswerable questions।
- Citation/page reference visible।
- Permission test।
- Failure case।
- Architecture diagram।
- Evaluation result।

Backup plan:
- Live provider/network failure হলে recorded response বা local fixture দেখাও।
- Demo environment-এ real secret/PII ব্যবহার কোরো না।
- Data unavailable হলে architecture ও test evidence explain করো।

## ১১. Sample Presentation Opening

"আমি একটি document question-answering system তৈরি করেছি, যার লক্ষ্য হলো user-কে বড় document collection থেকে relevant evidence খুঁজে source reference-সহ উত্তর দেওয়া। আমি ingestion এবং query path আলাদা রেখেছি, যাতে indexing asynchronous এবং question answering responsive থাকে।"

এটি নিজের implementation অনুযায়ী edit করবে। বাস্তবে implement না করা feature নিজের project-এ আছে বলে দাবি করবে না।

## ১২. Common Interview Follow-up

- কেন RAG, fine-tuning নয়?
- Retrieval ভুল হলে কীভাবে diagnose করবে?
- Citation correctness কীভাবে validate করবে?
- User permission change হলে cache কীভাবে invalidate করবে?
- Document delete হলে stale vector কীভাবে remove হবে?
- Provider outage হলে কী হবে?
- ১০০ গুণ query volume এলে bottleneck কোথায়?
- কোন metric দিয়ে quality prove করবে?
- কোন অংশ নিজে implement করেছ?
- কোন decision পরে বদলাতে চাইবে?

## Demo Checklist

- [ ] Problem ও target user স্পষ্ট।
- [ ] Architecture diagram ready।
- [ ] Ingestion ও query flow explain করতে পারি।
- [ ] At least 3 answerable questions।
- [ ] At least 2 unanswerable questions।
- [ ] Citation/page reference visible।
- [ ] Permission test।
- [ ] Failure case demo।
- [ ] Evaluation result ও sample size।
- [ ] Latency/cost evidence (যদি measure করা থাকে)।
- [ ] Limitations ও next steps।
- [ ] Backup demo plan।
- [ ] Secret/PII exposed নয়।

## Practice

1. ৫ মিনিটে project explain করে record করো।
2. Architecture diagram ছাড়া whiteboard-এ flow আঁকো।
3. তিনটি design decision-এর alternative ও trade-off বলো।
4. Evaluation metric ও actual evidence explain করো।
5. একটি failure scenario root cause পর্যন্ত trace করো।
6. নিজের contribution ও limitation স্পষ্টভাবে বলো।
