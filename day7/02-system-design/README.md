# AI System Design: Enterprise Document Intelligence

এই system design exercise-এ একটি enterprise-এর document collection থেকে employee natural language-এ প্রশ্ন করবে এবং source/page reference-সহ answer পাবে। লক্ষ্য শুধু RAG component list করা নয়; ingestion lifecycle, multi-tenant security, latency, scaling, cost, failure recovery এবং evaluation design করা।

## ১. Problem Statement এবং Assumptions

একটি enterprise-এ হাজার হাজার PDF/document আছে। Employee প্রশ্ন করবে এবং উত্তর source/page reference-সহ পাবে। একাধিক tenant ও concurrent user থাকবে।

Interview শুরু করার আগে clarification:
- Document type: PDF, DOCX, HTML, scanned image?
- Documents-এর average/max size?
- Update/delete frequency?
- User/tenant permission model?
- Query volume ও concurrency?
- Acceptable latency ও availability?
- Answer citation/page reference দরকার?
- Data residency/retention requirement?
- Unanswerable question-এ abstain করবে?

## ২. Requirements

### Functional
- Document upload ও ingestion।
- Document parsing, indexing, update/delete।
- Natural language Q&A।
- Source citation।
- Tenant/user-based access control।
- Unanswerable question-এ abstention।
- Feedback/report incorrect answer।

### Non-functional
- Predictable latency।
- Secure isolation।
- High availability।
- Cost control।
- Auditability।
- Reproducible evaluation।
- Freshness ও deletion propagation।

## ৩. Capacity Estimation কীভাবে করবে?

ধরি illustrative assumptions:
- ১০০টি tenant।
- মোট ১ মিলিয়ন document।
- Average ২০টি chunk/document।
- মোট ২০ মিলিয়ন chunk/vector।
- Peak ২০০ query/second।

এগুলো sample assumption; interview-তে candidate requirement অনুযায়ী estimate update করবে।

প্রাথমিক হিসাব:
- Vector count = documents × chunks per document।
- Storage = vector storage + text/metadata + original object storage।
- Query load = QPS × average retrieval/generation concurrency।
- Token spend = request volume × average input/output token × provider price।

প্রথমে rough order-of-magnitude estimate করো, পরে benchmark দিয়ে validate।

## ৪. High-level Architecture

~~~mermaid
flowchart TD
    U["Employee / Client"] --> API["API Gateway"]
    API --> AUTH["Authentication / Authorization"]
    AUTH --> Q["Query Service"]
    Q --> EMB["Query Embedding"]
    EMB --> RET["ACL-aware Search / Vector Index"]
    RET --> RERANK["Reranker"]
    RERANK --> CTX["Context Builder"]
    CTX --> LLM["LLM Provider"]
    LLM --> VAL["Citation / Output Validation"]
    VAL --> API

    UP["Upload API"] --> OBJ["Object Storage"]
    OBJ --> QUEUE["Ingestion Queue"]
    QUEUE --> PARSE["Parser / OCR / Cleaner"]
    PARSE --> CHUNK["Chunk + Metadata"]
    CHUNK --> WORKER["Embedding Worker"]
    WORKER --> INDEX["Vector / Search Index"]
    CHUNK --> META["Relational Metadata DB"]
~~~

Responsibilities:
- Object storage: original documents।
- Relational DB: users, tenants, permissions, document metadata, ingestion status।
- Queue: asynchronous ingestion jobs।
- Worker: parse, chunk, embed ও index।
- Search index: vector/keyword retrieval।
- Query service: ACL-aware retrieval ও LLM orchestration।

## ৫. API Contract কেমন হতে পারে?

Example endpoints:

~~~text
POST   /v1/documents
GET    /v1/documents/{document_id}
DELETE /v1/documents/{document_id}
POST   /v1/questions
GET    /v1/ingestion-jobs/{job_id}
POST   /v1/feedback
~~~

Question request-এ:
- question text।
- optional conversation/session ID।
- request ID/idempotency key।
- user identity authentication token থেকে, client-supplied tenant ID blindly trust নয়।

Response-এ:
- answer।
- citations (document ID, page/section, chunk ID)।
- answer status: answered / abstained / needs_review।
- request ID ও relevant metadata।

## ৬. Ingestion Design কীভাবে হবে?

Upload API document metadata ও object storage reference persist করবে। Queue worker asynchronous parse, OCR, chunk ও embedding করবে।

~~~mermaid
stateDiagram-v2
    [*] --> Uploaded
    Uploaded --> Processing
    Processing --> Indexed
    Processing --> Failed
    Failed --> Processing: Retry
    Indexed --> Updating
    Updating --> Indexed
    Indexed --> Deleting
    Deleting --> Deleted
~~~

Requirements:
- Idempotent ingestion।
- Document versioning।
- Duplicate upload detection।
- Retry ও dead-letter queue।
- Partial failure recovery।
- Parser/OCR error tracking।
- Update/delete propagation।
- Index readiness status।

Document indexed হয়েছে বলে পুরো pipeline success ধরে নিও না; chunk count ও expected metadata verify করো।

## ৭. Storage Design কী হবে?

| Store | Data |
|---|---|
| Object storage | Original PDF/DOCX/HTML |
| Relational DB | Tenant, user, ACL, document version, job state |
| Vector/search index | Chunk text/reference, embedding, metadata |
| Queue | Ingestion jobs, retry, DLQ |
| Cache | Permission-scoped safe result/metadata |

Relational DB-তে document status ও version source of truth হতে পারে। Vector index derived data; re-build/re-index strategy থাকতে হবে।

## ৮. Retrieval ও Generation Design

1. Authenticated user identity থেকে tenant ও permission resolve।
2. Query embedding।
3. ACL-aware retrieval।
4. Dense/keyword hybrid search।
5. Candidate reranking।
6. Duplicate ও irrelevant chunk removal।
7. Context token budget।
8. LLM generation।
9. Citation/output validation।
10. Evidence না থাকলে abstain।

Authorization prompt-এ ছেড়ে দেওয়া যাবে না। Unauthorized chunk LLM context-এ ঢুকতে দেওয়া যাবে না।

## ৯. Security Design

- Tenant isolation।
- Document-level ACL।
- Encryption at rest/in transit।
- Secret management।
- Prompt injection defense।
- Tool allowlist ও least privilege।
- Audit trail।
- Data retention/delete propagation।
- Cache permission scoping।
- Citation endpoint authorization।

~~~mermaid
flowchart TD
    A["Authenticated request"] --> B["Resolve tenant / user"]
    B --> C["Check document permissions"]
    C --> D["Apply ACL to retrieval"]
    D --> E["Verify each returned chunk"]
    E --> F["Build allowed context only"]
    F --> G["Generate answer"]
    G --> H["Validate citations and access"]
~~~

## ১০. Scalability কীভাবে design করবে?

- Ingestion worker horizontally scale করা যায়।
- Queue burst traffic absorb করে।
- Query API stateless রাখলে horizontal scaling সহজ।
- Vector index partition/shard strategy volume ও tenant pattern অনুযায়ী।
- Embedding batch processing throughput বাড়াতে পারে।
- Cache only permission-aware data।
- Backpressure ও provider rate limit রাখতে হবে।

একটি বড় tenant-এর workload অন্য tenant-এর latency বা quota consume করছে কি না monitor করো।

## ১১. Reliability ও Failure Recovery

সম্ভাব্য failure:
- Object upload success, metadata write fail।
- Queue duplicate delivery।
- Parser/OCR crash।
- Embedding provider rate limit।
- Partial vector indexing।
- LLM provider timeout।
- Permission update-এর পর stale cache।
- Delete event processing fail।

Design:
- Idempotency key ও versioned job।
- Retry with backoff/jitter।
- Dead-letter queue।
- Circuit breaker।
- Reconciliation job।
- Delete tombstone ও cache invalidation।
- Graceful abstention/fallback।

## ১২. Monitoring ও SLO কীভাবে define করবে?

Monitor:
- Ingestion success/failure ও freshness lag।
- Index chunk count/version consistency।
- Retrieval Recall@k।
- Citation correctness ও answer faithfulness।
- p50/p95 latency per stage।
- Token cost per query।
- No-answer rate।
- Access denied events।
- Queue depth ও worker lag।
- Provider error/timeout rate।

SLO example:
- Query p95 latency target।
- Ingestion freshness target।
- Availability target।
- ACL violation target: zero tolerated।

Exact numbers workload ও business requirement থেকে define করবে; arbitrary number মুখস্থ নয়।

## ১৩. Evaluation কীভাবে করবে?

- Golden question set।
- Relevant chunk IDs।
- Recall@k, MRR/nDCG।
- Answer correctness/faithfulness।
- Citation correctness।
- Unanswerable question abstention।
- Cross-tenant isolation tests।
- Prompt injection tests।
- Latency/cost benchmark।
- Document update/delete freshness tests।

Retrieval ও generation আলাদা evaluate করো। শুধু demo answer দেখে quality claim কোরো না।

## ১৪. Trade-offs

| Decision | Benefit | Cost / Risk |
|---|---|---|
| Larger chunk | More context | Less precise retrieval |
| Higher top-k | Recall potential | More noise/token cost |
| Reranker | Better ordering | Latency/cost |
| Stronger model | Quality potential | Cost/latency |
| Hybrid retrieval | Keyword + semantic | More index/complexity |
| Strict ACL filtering | Better isolation | Metadata/index complexity |
| Cache | Lower latency/cost | Staleness/privacy risk |
| Async ingestion | Query path responsive | Eventual consistency |

## ১৫. Interview Drill

1. ১০ হাজার document থেকে ১০ মিলিয়নে গেলে কী বদলাবে?
2. Document delete হলে সব vector কীভাবে remove করবে?
3. User permission change হলে stale cache কীভাবে invalidate করবে?
4. RAG latency ৮ সেকেন্ড হলে কোথা থেকে optimize শুরু করবে?
5. Citation hallucination কীভাবে detect করবে?
6. Provider outage-এ system কী করবে?
7. Duplicate ingestion event এলে কী হবে?
8. একটি tenant অন্য tenant-এর resource exhaust করলে কীভাবে isolate করবে?
9. Index partially updated হলে query কীভাবে behave করবে?
10. Model quality regression হলে rollback কীভাবে করবে?

## System Design Answer Structure

1. Clarify requirements ও assumptions।
2. Estimate scale/traffic।
3. Define APIs ও data model।
4. Draw high-level architecture।
5. Explain ingestion ও query path।
6. Security/authorization boundary।
7. Failure/recovery ও consistency।
8. Scaling, latency ও cost।
9. Evaluation/monitoring।
10. Trade-off ও future evolution।
