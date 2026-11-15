# Day 5 Practical Exercises

এই chapter-এ end-to-end RAG prototype তৈরি করবে। লক্ষ্য শুধু question-এর answer generate করা নয়; retrieval relevance, source citation, access control, no-answer behavior এবং production trade-off validate করা।

## Exercise 1: Document Ingestion

একটি PDF, একটি Markdown file ও একটি HTML page থেকে text extract করো।

Requirements:
- Source ID ও document version।
- Page/section metadata।
- Table ও heading extraction behavior inspect।
- Parsing failure record।
- Scanned PDF হলে OCR path document করো।

## Exercise 2: Chunking Experiment

Chunk size ও overlap-এর অন্তত ৩টি configuration পরীক্ষা করো।

- একই query set ব্যবহার করো।
- Chunk boundary-তে answer split হয়েছে কি না inspect করো।
- Recall@k compare করো।
- Duplicate/overlapping context measure করো।
- Token/index size report করো।

~~~mermaid
flowchart TD
    A["Source document"] --> B["Chunk strategy A"]
    A --> C["Chunk strategy B"]
    A --> D["Chunk strategy C"]
    B --> E["Same evaluation queries"]
    C --> E
    D --> E
    E --> F["Compare recall, duplication, cost"]
~~~

## Exercise 3: Embedding Search

Embedding তৈরি করে vector store-এ রাখো। Query embedding দিয়ে top-5 chunk retrieve করো।

- Similarity score inspect করো।
- Cosine, dot product বা Euclidean metric compare করো (embedding model guidance মেনে)।
- Embedding model/version record করো।
- Query ও document embedding compatibility verify করো।

## Exercise 4: Hybrid Retrieval

Keyword search ও vector search-এর result combine করো।

- Identifier-heavy query।
- Natural language paraphrase।
- Acronym ও product code।
- RRF দিয়ে rank fusion।
- Optional reranker দিয়ে final order compare।

## Exercise 5: RAG Answering

Retrieved chunk prompt-এ দিয়ে answer generate করো।

Requirements:
- Source citation metadata preserve।
- প্রতিটি factual claim-এর evidence reference।
- Evidence না থাকলে abstain।
- Conflicting source version হলে behavior define।
- Retrieved document-এর instruction-কে untrusted data হিসেবে treat।

## Exercise 6: Retrieval Evaluation

২০টি প্রশ্নের জন্য relevant chunk label করো। Calculate:
- Recall@k।
- Precision@k।
- MRR।
- nDCG (graded relevance থাকলে)।

Retriever বদলালে fixed evaluation set-এ compare করো।

## Exercise 7: Generation Evaluation

প্রতিটি question-এর expected key facts লিখো। Measure:
- Answer correctness।
- Faithfulness/groundedness।
- Answer relevance।
- Citation correctness।
- No-answer accuracy।

Schema-valid output এবং factually correct output আলাদা করে report করো।

## Exercise 8: Access Control

দুইটি user/tenant-এর document index করো।

- Tenant A যেন Tenant B-এর chunk retrieve না করতে পারে।
- User permission revoke করলে পুরোনো cache/vector result পাওয়া যাবে না।
- Direct citation URL permission bypass করবে না।
- ACL filter server-side enforce করো।

~~~mermaid
flowchart TD
    A["User identity"] --> B["Authorization service"]
    B --> C["Tenant + document ACL filter"]
    C --> D["Retrieval"]
    D --> E{"All chunks authorized?"}
    E -->|হ্যাঁ| F["Context builder"]
    E -->|না| G["Reject / remove unauthorized chunks"]
~~~

## Exercise 9: Prompt Injection in Retrieved Documents

কমপক্ষে ৫টি malicious document তৈরি করো:
- Ignore previous instructions।
- Secret/system prompt reveal request।
- Unauthorized tool call request।
- User/tenant data exfiltration request।
- Conflicting instruction in quoted text।

Test করো model যেন document content-কে evidence হিসেবে নেয়, trusted instruction হিসেবে নয়। Tool permission application-side enforce করো।

## Exercise 10: Failure Analysis

কমপক্ষে ১০টি ভুল answer inspect করে failure classify করো:
- Parsing/OCR।
- Chunking।
- Embedding mismatch।
- Retrieval/ranking।
- Metadata filter।
- Context assembly/truncation।
- Generation/grounding।
- Citation।
- Permission।
- Stale index।

প্রতিটি failure-এর root cause ও proposed fix লিখো।

## Exercise 11: Freshness and Deletion

একটি document update ও delete করো।

- Versioned ingestion।
- Old chunk/vector invalidation।
- Cache invalidation।
- Failed re-index retry।
- Delete propagation time measure।
- Stale answer test।

## Exercise 12: Latency and Cost

Query path-এর প্রতিটি stage-এর latency measure করো:
- Query embedding।
- Retrieval।
- Reranking।
- Context construction।
- Generation।
- Validation।

p50/p95 latency, token cost এবং cost per successful answer report করো।

## Deliverables

- Runnable PDF Q&A prototype।
- Ingestion ও query architecture diagram।
- Evaluation dataset ও metric report।
- Access control ও prompt injection test।
- Latency/cost breakdown।
- Known limitation document।
- ৩-৫ মিনিটের architecture walkthrough।

## Self-review

- [ ] Chunk metadata ও version রাখি।
- [ ] Retrieval result inspect করি।
- [ ] Retrieval ও generation আলাদা evaluate করি।
- [ ] No-answer case test করি।
- [ ] Citation source verify করি।
- [ ] ACL enforce ও test করি।
- [ ] Prompt injection threat বিবেচনা করি।
- [ ] Document update/delete lifecycle test করি।
- [ ] Latency ও token cost measure করি।
- [ ] Failure root cause document করি।

## Completion Criteria

1. Relevant source chunk consistently retrieve হয়।
2. Answer claim-এর citation valid ও supportive।
3. Evidence না থাকলে system abstain করে।
4. Cross-tenant retrieval test pass করে।
5. Prompt injection থেকে unauthorized action হয় না।
6. Update/delete stale index invalidate করে।
7. Quality, latency ও cost report করা হয়েছে।
