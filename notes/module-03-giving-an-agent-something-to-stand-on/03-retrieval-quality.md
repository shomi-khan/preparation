# 03. Retrieval Quality: Finding the Right Evidence

একটি RAG system-এর answer quality অনেকাংশে নির্ভর করে retriever সঠিক evidence খুঁজে পাচ্ছে কি না। Model-কে ভুল document দিলে prompt যত সুন্দরই হোক, ভালো উত্তর পাওয়ার সম্ভাবনা কম।

## Recall এবং Precision

ধরো, একটি প্রশ্নের উত্তর দিতে corpus-এ ৪টি relevant chunk আছে। Retriever ৩টি পেয়েছে এবং মোট ৫টি chunk এনেছে।

- **Recall**: সব relevant item-এর কত অংশ retrieve হয়েছে?
- **Precision**: retrieve করা item-এর কত অংশ relevant?

~~~text
Precision = Relevant Retrieved / Total Retrieved
Recall    = Relevant Retrieved / Total Relevant
~~~

High recall-এ evidence বাদ পড়ার ঝুঁকি কম; high precision-এ noise কম। দুটোই দরকার।

## Dense retrieval / Vector search

Query ও chunk embedding vector-এ রূপান্তরিত হয়। Cosine similarity, dot product বা distance metric দিয়ে কাছাকাছি vector খোঁজা হয়।

Query “How do I get my money back?” হলে “Refund requests must be submitted within 30 days” relevant হতে পারে, যদিও exact word এক নয়।

সীমাবদ্ধতা:
- Exact ID, product code, error code miss হতে পারে।
- Similar wording কিন্তু different meaning ভুলভাবে কাছাকাছি আসতে পারে।
- Embedding model-এর domain/language performance ভিন্ন হয়।

## Keyword / Sparse retrieval

BM25-এর মতো lexical retrieval term frequency, inverse document frequency ও document length বিবেচনা করে। Exact term, identifier, acronym, code বা rare phrase-এর জন্য কার্যকর।

ERR_AUTH_4037, SKU-AX19 বা policy clause R-17 খুঁজতে keyword retrieval ভালো হতে পারে। Synonym/paraphrase-এ dense retrieval বেশি সাহায্য করতে পারে।

## Hybrid retrieval

Dense ও keyword search একত্রে ব্যবহার করাকে hybrid retrieval বলে।

~~~mermaid
flowchart TD
  Q[User Query] --> D[Dense Vector Search]
  Q --> K[Keyword / BM25 Search]
  D --> F[Fusion / Deduplication]
  K --> F
  F --> R[Reranker]
  R --> C[Top Evidence Chunks]
~~~

দুই search-এর raw score একই scale-এ নাও থাকতে পারে। Normalization বা Reciprocal Rank Fusion (RRF)-এর মতো rank-based fusion ব্যবহার করা যায়।

RRF-এর ধারণা:

~~~text
RRF_score(document) = Σ 1 / (k + rank_in_list)
~~~

Rank যত উপরে, contribution তত বেশি। k smoothing constant। Fusion strategy-ও evaluate করতে হবে।

## Reranking

প্রথম retriever দ্রুত অনেক candidate আনে। Reranker query ও candidate একসঙ্গে দেখে relevance ranking উন্নত করে।

- Bi-encoder: query ও document আলাদা encode; search দ্রুত।
- Cross-encoder: query-document pair একসঙ্গে evaluate; সাধারণত বেশি accurate, কিন্তু প্রতি candidate-এ compute লাগে।

~~~mermaid
flowchart LR
  Q[Query] --> R[Fast Retriever]
  I[(Index)] --> R
  R --> C[Top Candidates]
  C --> X[Cross-Encoder Reranker]
  X --> T[Top Evidence]
~~~

Retriever relevant chunk একেবারেই না আনলে reranker সেটি উদ্ধার করতে পারবে না।

## Query rewriting ও decomposition

User query conversational বা অস্পষ্ট হতে পারে:
- Conversation: “ওটা কত দিনে ফেরত পাওয়া যায়?”
- Search query: “Company refund policy: purchase-এর কত দিনের মধ্যে refund request করা যায়?”

Conversation context দিয়ে standalone query বানানো যায়। Complex question sub-query-তে ভাগ হতে পারে। যেমন “গত বছরের তুলনায় এ বছরের refund rate কেন বেড়েছে?” প্রশ্নে দুই বছরের rate, policy change ও incident report আলাদা খুঁজতে হতে পারে।

সতর্কতা: rewriting-এ intent বদলে যেতে পারে। Original এবং rewritten query trace করো; প্রয়োজন হলে দুটোই search করো।

## Metadata filtering

Filter হতে পারে tenant_id, allowed_groups, document_type, language, effective_date, version/status, product বা region।

**Security filter retrieval-এর আগেই trusted identity দিয়ে enforce করতে হবে।** পরে unauthorized result বাদ দিলে intermediate logs, reranker বা model context-এ leak হতে পারে।

## Chunk size, overlap ও top-k trade-off

| Setting | খুব কম হলে | খুব বেশি হলে |
|---|---|---|
| Chunk size | Context ভেঙে যায় | Topic মিশে যায়, noise |
| Overlap | Boundary context হারায় | Duplicate ও cost |
| Top-k | Evidence বাদ | Irrelevant context |
| Candidate pool | Recall কম | Rerank cost/latency |
| Context budget | Evidence কম | Model distraction |

কোনো universally best chunk size বা top-k নেই। Query set ও corpus দিয়ে experiment করো।

## Context selection

সব retrieved chunk model-কে পাঠানো উচিত নয়। Context builder:
1. Duplicate/near-duplicate বাদ।
2. একই document-এর redundant passage কমানো।
3. সব sub-question cover হয়েছে কি না দেখা।
4. Contradictory version detect।
5. Source diversity বিবেচনা।
6. Token budget অনুযায়ী evidence নির্বাচন।
7. Citation metadata preserve।

Long context মানেই unlimited attention নয়। Evidence-এর ordering ও placement-ও answer quality-তে প্রভাব ফেলে।

## Retrieval evaluation

| Metric | কী বোঝায় |
|---|---|
| Recall@k | Top-k-তে relevant evidence কতটা |
| Precision@k | Top-k-এর কতটা relevant |
| MRR | প্রথম relevant result কত উপরে |
| nDCG@k | Relevance grade ও ranking position |
| Context precision | Generation context-এ relevant evidence-এর অনুপাত |
| Context recall | প্রয়োজনীয় evidence context-এ এসেছে কি না |

Evaluation set-এ বাস্তব query, expected source/chunk, relevant document, answerable/unanswerable status এবং permission scope রাখো।

## Practical experiment

একই evaluation set-এ compare করো:
1. Dense retrieval top-5
2. Dense top-20 + rerank top-5
3. BM25 top-20
4. Hybrid top-20 + rerank top-5

Measure: Recall@k, nDCG, answer correctness, citation correctness, p95 latency ও cost per query। ACL কখনো off করে production data-তে quality experiment করবে না।

## Failure analysis

| Symptom | সম্ভাব্য কারণ | তদন্ত |
|---|---|---|
| Relevant doc নেই | Ingestion/index gap | Source coverage, parser |
| Similar but wrong result | Dense ambiguity | Hybrid, metadata |
| Correct doc, wrong passage | Chunking | Structure-aware chunk |
| Top result ভালো, answer খারাপ | Context/prompt/generation | Evidence ordering |
| Old policy answer | Version/freshness | Effective date |
| Unauthorized content | ACL bug | Identity propagation |

## Interview drill

**Dense বনাম sparse retrieval?**  
Dense semantic similarity ধরতে পারে; sparse exact lexical match-এ শক্তিশালী। Identifier-heavy corpus-এ hybrid useful।

**Reranker কেন?**  
Retriever broad candidate set আনে; reranker query ও candidate pair বিচার করে। Candidate-এ evidence না থাকলে reranker খুঁজে আনতে পারে না।

**Top-k কীভাবে বাছবে?**  
Recall@k, precision/nDCG, final answer quality, latency ও cost একসঙ্গে মেপে। বেশি k মানেই ভালো নয়।

**Metadata filter কি security boundary?**  
শুধু trusted identity থেকে server-side enforce হলে এবং সব retrieval path একই policy মানলে। Client-provided filter বিশ্বাস করা যাবে না।

**Retrieval ভালো হলেও hallucination কেন?**  
Model evidence ভুল বুঝতে পারে, context-এ conflict থাকতে পারে, citation mismatch হতে পারে। Retrieval ও generation আলাদা evaluate করতে হবে।

## অনুশীলন

একটি policy corpus-এ exact clause ID ও natural-language question আছে:
1. Dense-only search কেন clause ID miss করতে পারে?
2. BM25 ও vector result কীভাবে fuse করবে?
3. Reranker কোথায় বসবে?
4. Draft ও active version থাকলে active version কীভাবে নিশ্চিত করবে?
5. Query answerable না হলে system কী করবে?