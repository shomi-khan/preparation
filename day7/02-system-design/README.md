# AI System Design: Enterprise Document Intelligence

## Problem
একটি enterprise-এর হাজার হাজার PDF/document আছে। Employee natural language-এ প্রশ্ন করবে এবং উত্তর source/page reference-সহ পাবে। একাধিক tenant ও concurrent user থাকবে।

## ১. Requirements

Functional:
- Document upload ও ingestion।
- Document parsing, indexing ও update/delete।
- Natural language Q&A।
- Source citation।
- Tenant/user-based access control।
- Unanswerable question-এ abstention।

Non-functional:
- Low and predictable latency।
- Secure isolation।
- High availability।
- Cost control।
- Auditability।
- Reproducible evaluation।

## ২. High-level Architecture

```text
                 Upload API
                     |
              Object Storage
                     |
              Ingestion Queue
                     |
          Parser / OCR / Cleaner
                     |
             Chunk + Metadata
                     |
              Embedding Worker
                     |
           Vector / Search Index

User -> API Gateway -> AuthN/AuthZ
                    -> Query Embedding
                    -> ACL-aware Retrieval
                    -> Reranker
                    -> Context Builder
                    -> LLM
                    -> Citation / Output Validation
                    -> User
```

## ৩. Ingestion Design
Upload API document metadata ও object storage reference persist করবে। Queue worker asynchronous parse, OCR, chunk ও embedding করবে। Status tracking থাকবে: uploaded, processing, indexed, failed।

Idempotent ingestion ও document versioning দরকার। Re-upload হলে পুরোনো chunk invalidate করতে হবে।

## ৪. Storage
- Object storage: original documents।
- Relational DB: users, tenants, permissions, document metadata, ingestion status।
- Vector/search index: chunks, embeddings, metadata।
- Queue: asynchronous ingestion jobs।

## ৫. Retrieval
User identity থেকে authorized tenant/document scope resolve হবে। Retrieval query সেই ACL scope দিয়ে filter হবে। Top-k candidate retrieve, optional hybrid search ও rerank, তারপর context build।

Authorization prompt-এ ছেড়ে দেওয়া যাবে না।

## ৬. Generation
System instruction, user question ও retrieved evidence আলাদা করে prompt করা হবে। Evidence source ID/page metadata সহ থাকবে। Unsupported answer-এ abstain করতে হবে। Output schema validate হবে।

## ৭. Security
- Tenant isolation।
- Document ACL।
- Encryption at rest/in transit।
- Secret management।
- Prompt injection defense।
- Tool allowlist।
- Audit trail।
- Data retention/delete propagation।

## ৮. Scalability
Ingestion worker horizontally scale করা যায়। Query path stateless API service দিয়ে scale করা যায়। Vector index partition/shard strategy volume ও tenant pattern অনুযায়ী নির্ধারণ। Queue backpressure এবং rate limit রাখতে হবে।

## ৯. Reliability
Provider timeout, retry with backoff, circuit breaker, queue retry/DLQ, idempotency, graceful degradation ও re-index recovery।

## ১০. Monitoring
- Ingestion success/failure।
- Index freshness।
- Retrieval Recall@k।
- Citation correctness।
- Answer faithfulness।
- p50/p95 latency per stage।
- Token cost per query।
- No-answer rate।
- Access denied events।

## ১১. Trade-offs
- Larger chunk: more context, less precise retrieval।
- Higher top-k: better recall potential, more noise/cost।
- Reranker: quality improve, latency increase।
- Stronger model: quality potential higher, cost/latency higher।
- Strict ACL filter: secure, but metadata/index design complexity বাড়ে।

## Interview Drill
1. ১০ হাজার document থেকে ১০ মিলিয়নে গেলে কী বদলাবে?
2. Document delete হলে সব vector কীভাবে remove করবে?
3. User permission change হলে stale cache কীভাবে invalidate করবে?
4. RAG latency ৮ সেকেন্ড হলে কোথা থেকে optimize শুরু করবে?
5. Citation hallucination কীভাবে detect করবে?
6. Provider outage-এ system কী করবে?
