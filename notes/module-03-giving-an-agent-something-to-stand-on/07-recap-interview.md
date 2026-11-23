# 07. Module 3 Recap ও Interview Drill

## 90-second recap

LLM-এর parametric knowledge private, changing বা current company knowledge-এর জন্য যথেষ্ট নয়। RAG inference-time-এ external source থেকে evidence retrieve করে model context-এ যোগ করে। Ingestion-এ parsing, normalization, chunking, metadata, embedding ও indexing থাকে; query pipeline-এ query processing, retrieval, filtering, reranking, context construction ও generation থাকে। Dense retrieval semantic similarity খোঁজে; keyword retrieval exact term-এ শক্তিশালী; hybrid retrieval দুটো combine করে। Knowledge Graph entity ও relationship-কে node-edge-property আকারে ধরে, তাই multi-hop query সহজ হয়। GraphRAG graph-aware retrieval ও document evidence একত্র করে। Production system-এ ACL, freshness, provenance, evaluation, prompt injection defense, cache isolation, observability ও safe abstention জরুরি।

## Quick comparison

| Concept | মূল প্রশ্ন |
|---|---|
| Parametric knowledge | Model training থেকে কী শিখেছে? |
| External knowledge | Current/private fact কোন source-এ আছে? |
| Grounding | Answer কোন evidence-এর ওপর দাঁড়ানো? |
| RAG | Evidence কীভাবে retrieve করে generation-এ যোগ হবে? |
| Embedding | Text-এর semantic representation কীভাবে হবে? |
| Vector search | Meaning-এর দিক থেকে কাছের chunk কোনগুলো? |
| Keyword search | Exact term/identifier কোথায় আছে? |
| Reranking | Candidate-এর মধ্যে কোন evidence বেশি relevant? |
| Knowledge Graph | Entity-গুলোর relationship কী? |
| GraphRAG | Graph ও document evidence একত্রে কীভাবে retrieve হবে? |

## Foundational interview questions

**1. RAG কী?**  
RAG হলো inference-time pattern যেখানে query অনুযায়ী external source থেকে evidence retrieve করে LLM context-এ যোগ করা হয়। Model evidence ব্যবহার করে answer তৈরি করে; model weights update হয় না।

**2. RAG pipeline-এর দুই অংশ কী?**  
Indexing: source → parse → chunk → metadata → embedding → index। Query-time: query → retrieval → filtering/reranking → context → generation/citation।

**3. Chunking কেন দরকার?**  
Focused retrieval unit, context budget ও precise citation-এর জন্য। খুব ছোট chunk context হারায়; খুব বড় chunk noise বাড়ায়। Evaluation দিয়ে size/overlap tune করতে হয়।

**4. Embedding কী?**  
Text-কে dense numeric vector-এ map করা representation, যেখানে semantic similarity আনুমানিক geometric proximity দিয়ে মাপা যায়। Model/version বদলালে re-embedding দরকার হতে পারে।

**5. Vector search আর keyword search?**  
Vector semantic similarity ধরতে পারে; keyword/BM25 exact term, ID, acronym ও rare code-এ ভালো। Hybrid দুই signal combine করে।

**6. Reranker কী করে?**  
Retriever candidate আনে; reranker query ও candidate একসঙ্গে বিচার করে relevance অনুযায়ী সাজায়। Candidate-এ evidence না থাকলে reranker নিজে খুঁজে পায় না।

**7. Knowledge Graph কী?**  
Entity/node এবং relationship/edge/property দিয়ে knowledge represent করা। Multi-hop query ও dependency analysis-এ কাজে লাগে।

**8. Entity resolution কেন?**  
এক entity বিভিন্ন alias-এ থাকতে পারে; একই নামের ভিন্ন entity-ও থাকতে পারে। ভুল merge false relation তৈরি করে। Stable ID, domain constraint ও review দরকার।

**9. GraphRAG কী?**  
Knowledge Graph ও document retrieval একত্রে ব্যবহার করে LLM-কে relationship, multi-hop context বা corpus-level pattern-সহ evidence দেওয়া। এটি একক universal algorithm নয়।

**10. RAG কি hallucination দূর করে?**  
না। Retrieval miss, stale/conflicting source ও model misinterpretation হতে পারে। Evidence quality, abstention, citation validation ও evaluation দরকার।

## Deep interview questions with answer outlines

### 1. “Internal policy প্রতিদিন বদলায়। Fine-tune করবে, নাকি RAG?”
RAG baseline বেছে নেব, কারণ policy update-এর সঙ্গে source/index update করা যায় এবং citation রাখা যায়। Fine-tuning factual update-এর primary mechanism নয়। Style/task behavior-এর জন্য fine-tuning আলাদাভাবে বিবেচনা করা যায়। Source-of-truth, version ও freshness SLA নির্ধারণ করব।

### 2. “RAG ভুল উত্তর দিয়েছে। Root cause কীভাবে বের করবে?”
Stage ধরে trace: source-এ fact ছিল? parser ঠিক? index current? query rewrite intent ঠিক? retriever relevant chunk এনেছে? reranker/context builder বাদ দিয়েছে? model ভুল বুঝেছে? citation claim support করেছে? Retrieval ও generation evaluation আলাদা রাখব।

### 3. “Chunk size কীভাবে ঠিক করবে?”
Universal size বলব না। Corpus structure, query, embedding max input ও context budget দিয়ে fixed, structure-aware ও semantic chunking compare করব। Recall@k, nDCG, answer/citation correctness, latency/cost মাপব।

### 4. “Hybrid retrieval কেন?”
Dense paraphrase ও semantic similarity ধরতে পারে; BM25 exact term, ID, acronym ও rare keyword-এ ভালো। দুই ধরনের query থাকলে hybrid recall বাড়াতে পারে। Rank fusion ও reranking evaluate করব।

### 5. “Top-k ১০ থেকে ৫০ করলে quality বাড়বে?”
অবশ্যই নয়। Recall বাড়তে পারে, কিন্তু irrelevant chunk, token cost, latency ও distraction বাড়ে। Candidate pool, reranker ও final context budget আলাদা tune করব।

### 6. “User অন্য department-এর document দেখতে চাইলে?”
Authenticated identity থেকে tenant/group/role resolve করব। Retrieval-এর আগেই trusted server-side ACL filter enforce। Vector, keyword, graph, cache, reranker ও citation path-এ একই scope। Client-provided filter বিশ্বাস করব না।

### 7. “Retrieved document-এ prompt injection থাকলে?”
Document content untrusted data; system instruction থেকে আলাদা। অপ্রয়োজনীয় secret context-এ দেব না। Tool permission server-side enforce; high-risk action-এ approval; adversarial corpus test।

### 8. “Knowledge Graph কেন, relational DB কেন নয়?”
Workload দেখি। Deep/variable traversal, dependency বা multi-hop path-এ graph natural হতে পারে। Transactional tabular data-তে relational DB উপযুক্ত। Hybrid architecture সম্ভব; graph maintenance cost justify করতে হবে।

### 9. “GraphRAG কি সবসময় Vector RAG-এর চেয়ে ভালো?”
না। GraphRAG entity/relation extraction, resolution, storage ও refresh-এর extra cost আনে। Relationship-heavy বা corpus-level question-এ value দিতে পারে। Simple FAQ-তে Vector/Hybrid RAG যথেষ্ট হতে পারে।

### 10. “Graph relation trustworthy করবে কীভাবে?”
Ontology/schema, provenance, extraction confidence, entity resolution, temporal validity, validation rule ও human review। LLM-extracted edge authoritative নয়।

### 11. “Source update হলে answer current কীভাবে থাকবে?”
Version/hash, event/batch incremental ingestion, update/delete propagation, freshness monitoring ও atomic index publish। Query-তে effective date/status filter।

### 12. “No relevant document পাওয়া গেলে?”
Abstain, clarification বা authoritative source/tool-এ escalate। Company-specific policy parametric knowledge দিয়ে অনুমান করানো উচিত নয়।

### 13. “RAG cache কীভাবে design করবে?”
Cache key-তে normalized query ছাড়াও tenant/permission scope, corpus/index version ও locale বিবেচনা। Permission revoke/document update-এ invalidate। Personalized answer indiscriminately share নয়।

### 14. “RAG system quality কীভাবে মাপবে?”
Retrieval: Recall@k, Precision@k, MRR, nDCG। Generation: correctness, faithfulness, citation, completeness, abstention। System: p95 latency, cost, freshness, availability, ACL leakage tests। Gold set, human review, deterministic checks ও LLM judge combine।

### 15. “Vector DB কি RAG-এর অপরিহার্য অংশ?”
না। RAG pattern; Vector DB implementation option। Search engine, relational extension, keyword index, graph store বা hybrid retrieval হতে পারে। Workload অনুযায়ী নির্বাচন।

## System design exercise: Company Knowledge Assistant

Requirements: HR, Finance, Engineering-এর private documents; employee-specific permission; daily policy update; natural-language search; citation সহ answer; insufficient evidence-এ abstain।

~~~mermaid
flowchart TD
  S[Docs / Wiki / APIs] --> I[Ingestion and Parsing]
  I --> C[Chunk + Metadata + ACL]
  C --> E[Embedding / Keyword Index]
  C --> G[Optional Knowledge Graph]
  U[Employee] --> A[Authentication]
  A --> Q[Query Service]
  Q --> P[Permission-aware Retrieval]
  E --> P
  G --> P
  P --> R[Rerank and Context Build]
  R --> L[LLM]
  L --> V[Citation / Grounding Check]
  V --> O[Answer or Abstain]
~~~

Ownership:
- Source systems authoritative content।
- Ingestion pipeline parse/version/index।
- Identity service permission।
- Retriever ACL enforce।
- LLM answer draft।
- Citation validator claim-source support check।
- Runtime tool authorization আলাদা enforce।

Failure paths:
- ACL service down → sensitive retrieval fail closed।
- Index stale → freshness policy/fallback।
- No evidence → abstain।
- Prompt injection → retrieved content untrusted।
- Cache stale → version/permission-aware invalidation।

## Self-review checklist

- [ ] Parametric vs external knowledge
- [ ] Grounding ও RAG definition
- [ ] Indexing বনাম query-time
- [ ] Parsing/normalization
- [ ] Chunking/overlap
- [ ] Embedding/version compatibility
- [ ] Vector index/ANN
- [ ] Dense বনাম keyword retrieval
- [ ] Hybrid retrieval/RRF
- [ ] Reranking
- [ ] Metadata/ACL filtering
- [ ] Recall@k, Precision@k, MRR, nDCG
- [ ] Knowledge Graph node/edge/property
- [ ] Entity extraction/resolution
- [ ] Provenance/temporal validity
- [ ] GraphRAG local/global retrieval
- [ ] GraphRAG vs Vector RAG
- [ ] Prompt injection
- [ ] Freshness, deletion, caching
- [ ] Citation validation ও abstention

## Mock interview

**Interviewer:** “আমাদের company-তে হাজার হাজার document, service dependency, incident report ও policy আছে। একটি agent বানাতে হবে, যাতে employee প্রশ্ন করলে সঠিক source থেকে answer দেয়। Architecture কী হবে?”

উত্তর সাজাও:
1. Query type ও data source classify।
2. Document RAG baseline।
3. Structured/current state-এর জন্য API/SQL।
4. Relationship-heavy question-এর জন্য Knowledge Graph/GraphRAG।
5. Ingestion, metadata, ACL, freshness।
6. Hybrid retrieval ও reranking।
7. Context/citation/abstention।
8. Security ও prompt injection defense।
9. Evaluation, observability, cost।
10. Incremental rollout ও baseline comparison।

Framework-এর নামের চেয়ে data lifecycle, trust boundary, ownership, failure mode ও evaluation strategy ব্যাখ্যা করা বেশি গুরুত্বপূর্ণ।

## Oral practice rubric

প্রতিটি উত্তরে যাচাই করো:
1. Concept সঠিকভাবে define করেছি?
2. End-to-end flow বুঝিয়েছি?
3. Trade-off বা কখন ব্যবহার করব না বলেছি?
4. Failure mode ও mitigation বলেছি?
5. বাস্তব উদাহরণ বা metric দিয়েছি?

শুধু “RAG মানে vector database + LLM” বললে উত্তর অসম্পূর্ণ।