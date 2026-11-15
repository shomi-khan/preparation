# Day 5 Interview Questions and Answers

## RAG Fundamentals

### ১. RAG কী?
LLM response generate করার আগে external knowledge retrieve করে prompt context-এ যোগ করার architecture।

### ২. RAG কেন ব্যবহার করবে?
Private বা updated knowledge access, source citation, এবং model retraining ছাড়াই external documents ব্যবহার করতে।

### ৩. RAG কি hallucination দূর করে?
না। Retrieval ভুল/অসম্পূর্ণ হতে পারে এবং model context ভুল interpret করতে পারে। Grounding, verification, abstention ও evaluation দরকার।

### ৪. RAG ও Fine-tuning-এর পার্থক্য?
RAG inference-time knowledge retrieval; fine-tuning model weights update। Knowledge freshness ও traceability-এর জন্য RAG, behavior adaptation-এর জন্য fine-tuning বিবেচ্য।

### ৫. Chunking কেন দরকার?
বড় document থেকে relevant ছোট passage retrieve করতে এবং context budget manage করতে।

### ৬. Chunk size কীভাবে বেছে নেবে?
Document structure, question type, embedding model ও retrieval benchmark দিয়ে experiment করব। Universal optimal size নেই।

### ৭. Overlap কেন ব্যবহার হয়?
Chunk boundary-তে split হওয়া context হারানো কমাতে। বেশি overlap duplicate ও index cost বাড়ায়।

### ৮. Embedding কী?
Text-এর semantic vector representation, যা similarity-based retrieval-এ ব্যবহৃত হয়।

### ৯. Cosine similarity কী?
দুটি vector-এর direction similarity। এটি vector magnitude normalize করে; score-কে সরাসরি probability ভাবা যাবে না।

### ১০. Vector database কেন?
Embedding store, nearest-neighbor search ও metadata filtering efficiently করতে।

## Retrieval

### ১১. Semantic ও keyword search-এর পার্থক্য?
Semantic search meaning similarity ধরতে চেষ্টা করে; keyword search exact term/lexical match-এ ভালো। Hybrid search দুইটির সুবিধা combine করে।

### ১২. Reranking কী?
Initial retrieval-এর candidate query-document relevance অনুযায়ী re-order করা।

### ১৩. top-k বেশি করলে কি সবসময় ভালো?
না। Recall বাড়তে পারে, কিন্তু irrelevant context, latency ও token cost বাড়ে।

### ১৪. ANN কী?
Approximate nearest neighbor algorithm বড় vector index-এ দ্রুত similar candidate খুঁজে দেয়; exact nearest result miss হতে পারে।

### ১৫. Retrieval result খারাপ হলে কী debug করবে?
Query, parsing, chunk boundaries, embedding model, metadata filters, top-k এবং index freshness inspect করব। প্রথমে LLM ছাড়াই retrieved chunk পরীক্ষা করব।

## Evaluation

### ১৬. Recall@k কী?
Top-k result-এ relevant item-এর অনুপাত/coverage। Dataset definition অনুযায়ী denominator relevant ground-truth item count।

### ১৭. MRR কী?
প্রথম relevant result-এর reciprocal rank-এর average। Relevant result যত উপরে, score তত ভালো।

### ১৮. Faithfulness কী?
Generated answer retrieved evidence দ্বারা supported কি না।

### ১৯. Answer relevance ও faithfulness আলাদা কেন?
Answer প্রশ্নের সঙ্গে relevant হলেও unsupported claim থাকতে পারে। আবার context faithful হলেও প্রশ্নের পুরো উত্তর নাও দিতে পারে।

### ২০. LLM-as-judge কি যথেষ্ট?
না। এটি scalable signal দিতে পারে, কিন্তু judge bias, prompt sensitivity ও model error থাকে। Human-reviewed benchmark দরকার।

## Production and Security

### ২১. RAG-এ access control কীভাবে করবে?
Server-side authorization ও tenant/document ACL filter দিয়ে retrieval constrain করব। Prompt instruction security boundary নয়।

### ২২. Retrieved document-এ prompt injection থাকলে?
Document content untrusted data হিসেবে treat করব। Tool access allowlist, least privilege, argument validation ও human approval রাখব।

### ২৩. Document update হলে কী করবে?
Versioned ingestion, stale chunk/vector invalidation এবং delete lifecycle implement করব।

### ২৪. RAG latency কমাবে কীভাবে?
Embedding/retrieval parallelize যেখানে সম্ভব, index tune, candidate count/reranker balance, caching ও context size optimize। Permission-aware caching জরুরি।

### ২৫. Citation কীভাবে validate করবে?
Chunk metadata থেকে source reference তৈরি করব এবং claim-source support পরীক্ষা করব। Citation link থাকা মানেই claim correct নয়।

### ২৬. RAG system কীভাবে monitor করবে?
Retrieval score/result, stage latency, token cost, answer quality, citation correctness, no-answer rate ও user feedback track করব।

## Scenario Questions

### ২৭. Model বলে document-এ তথ্য নেই, কিন্তু document-এ আছে। কী করবে?
Document parsing, chunking, embedding coverage, query formulation, filters ও top-k inspect করব। তারপর retrieval benchmark-এ regression test যোগ করব।

### ২৮. Answer সঠিক কিন্তু citation ভুল। কী সমস্যা?
Generation source attribution ভুল করেছে বা context/source metadata mapping হারিয়েছে। Citation generation-কে retrieved chunk ID-এর সঙ্গে bind করতে হবে।

### ২৯. User A-এর private document User B-কে দেখাচ্ছে। কী করবে?
এটি security incident। Retrieval ACL, tenant scoping, cache key, index filter ও logs inspect করে exposure বন্ধ, incident response এবং regression tests করব।

### ৩০. কখন RAG ব্যবহার করবে না?
যখন external knowledge দরকার নেই, task ছোট fixed taxonomy, deterministic logic যথেষ্ট, অথবা latency/cost/quality trade-off RAG-কে অযৌক্তিক করে।


---

## Additional Interview Questions: Retrieval Quality and RAG Operations

### ৩১. Recall@k এবং Precision@k-এর পার্থক্য কী?

Recall@k top-k result-এ মোট relevant evidence-এর কত অংশ পাওয়া গেছে তা মাপে। Precision@k top-k result-এর মধ্যে কত অংশ relevant তা মাপে। High recall-এর জন্য বড় k লাগতে পারে, কিন্তু noise/context cost বাড়ে।

### ৩২. MRR এবং nDCG কী মাপে?

MRR প্রথম relevant result-এর reciprocal rank-এর average। nDCG graded relevance ও ranking position বিবেচনা করে। একাধিক relevant chunk এবং relevance grade থাকলে nDCG উপকারী।

### ৩৩. Dense ও sparse retrieval-এর পার্থক্য কী?

Dense retrieval embedding vector similarity ব্যবহার করে semantic relation ধরতে চেষ্টা করে। Sparse/lexical retrieval token match ও term weighting (যেমন BM25) ব্যবহার করে। Exact identifier ও semantic paraphrase উভয় দরকার হলে hybrid search ব্যবহার করা যায়।

### ৩৪. RRF কী?

Reciprocal Rank Fusion একাধিক ranked list-এর rank position combine করে। Raw score scale আলাদা হলেও rank-based fusion করা যায়। এটি hybrid retrieval-এ useful, কিন্তু fusion parameter ও candidate coverage evaluate করতে হয়।

### ৩৫. ANN retrieval-এর trade-off কী?

Approximate nearest neighbor search latency কমাতে পারে, কিন্তু exact nearest candidate miss করতে পারে। Recall, index memory, build time ও query latency balance করতে হয়।

### ৩৬. Reranker কেন first-stage retrieval-এর বিকল্প নয়?

Reranker শুধু retrieved candidate re-order করে। Relevant document first-stage candidate-এ না এলে reranker সেটি recover করতে পারে না। তাই candidate recall আগে যথেষ্ট হওয়া দরকার।

### ৩৭. MMR কখন ব্যবহার করবে?

Top result-এ near-duplicate chunk থাকলে relevance ও diversity balance করতে MMR ব্যবহার করা যায়। তবে diversity বেশি prioritize করলে highly relevant evidence বাদ পড়তে পারে।

### ৩৮. Query rewrite বা HyDE কীভাবে সাহায্য করে?

Query rewrite query-কে retrieval-friendly করে; HyDE hypothetical answer/document তৈরি করে embedding retrieval করতে পারে। এগুলো query intent ভুলভাবে বদলাতেও পারে, তাই original query-এর সঙ্গে evaluate করতে হবে।

### ৩৯. Parent-child retrieval কী?

ছোট child chunk retrieve করে বড় parent section বা surrounding context generation-এ ব্যবহার করা। এটি precise retrieval ও sufficient context-এর trade-off balance করতে পারে।

### ৪০. Embedding model বদলালে কী করতে হবে?

Query ও document embedding একই compatible vector space-এ থাকা জরুরি। Model/version বদলালে index re-embed ও rebuild করতে হতে পারে; পুরোনো vector-এর সঙ্গে নতুন query vector মিশিয়ে retrieval করা ঠিক নয়।

### ৪১. Cosine similarity score কি probability?

না। এটি vector direction similarity measure; score-কে directly answer correctness probability হিসেবে interpret করা যাবে না। Score distribution model/dataset অনুযায়ী বদলায়।

### ৪২. Faithfulness ও correctness-এর পার্থক্য কী?

Faithfulness answer retrieved context দ্বারা supported কি না। Correctness ground truth অনুযায়ী answer সঠিক কি না। ভুল source context থাকলে answer faithful কিন্তু factually wrong হতে পারে।

### ৪৩. RAG system unanswerable question কীভাবে handle করবে?

Retrieved evidence যথেষ্ট কি না assess করে abstain, clarification request বা human review path নিতে হবে। Evaluation set-এ unanswerable query রাখতে হবে; forced answer hallucination বাড়াতে পারে।

### ৪৪. Citation generation কীভাবে reliable করবে?

Application retrieved chunk ID ও metadata preserve করে citation render করতে পারে। Model-generated source name/URL blindly trust না করে claim-source support validate করতে হবে।

### ৪৫. Multi-tenant RAG-এ ACL কোথায় enforce করবে?

Server-side authorization ও retrieval filter-এ। Unauthorized chunk LLM context-এ যাওয়ার আগেই block করতে হবে। Cache key, citation endpoint ও re-index lifecycle-এও tenant permission enforce করতে হবে।

### ৪৬. Document update/delete হলে কী সমস্যা হতে পারে?

Stale chunk/vector বা cached answer পুরোনো information ফেরত দিতে পারে। Version-aware ingestion, delete/tombstone propagation, index invalidation ও cache invalidation দরকার।

### ৪৭. RAG latency কীভাবে diagnose করবে?

Query embedding, retrieval, metadata filtering, reranking, context construction, generation এবং validation stage-wise timing নাও। p50/p95 ও cost per successful answer measure করো।

### ৪৮. RAG-এর context window budget কীভাবে manage করবে?

Top-k blindly বাড়াবে না। Relevance, chunk length, duplicate removal, parent context, source diversity ও output reserve বিবেচনা করবে। Context compression/summary যোগ করলে evidence loss evaluate করতে হবে।

### ৪৯. RAG-এ prompt injection কীভাবে mitigate করবে?

Retrieved content-কে untrusted data হিসেবে treat করবে। Tool allowlist, least privilege, server-side authorization, argument validation এবং sensitive action-এ human approval রাখবে। Prompt wording একা যথেষ্ট নয়।

### ৫০. RAG evaluation regression কীভাবে করবে?

Fixed held-out query set-এ retrieval metrics, answer correctness, faithfulness, citation, ACL, no-answer behavior, latency ও cost compare করবে। Chunking/embedding/prompt/reranker change আলাদা version করবে।

## RAG Debugging Flow

~~~mermaid
flowchart TD
    A["Incorrect RAG answer"] --> B{"Relevant evidence retrieved?"}
    B -->|না| C["Parsing / chunking / embedding / filter / index"]
    B -->|হ্যাঁ| D{"Correct evidence in prompt?"}
    D -->|না| E["Context assembly / token truncation"]
    D -->|হ্যাঁ| F["Generation / citation / grounding"]
    C --> G["Regression evaluation"]
    E --> G
    F --> G
~~~

## Oral Practice

একটি RAG failure scenario নাও এবং retrieval, generation, security, freshness ও cost perspective থেকে root cause বিশ্লেষণ করো। শুধু "prompt improve করব" উত্তর যথেষ্ট নয়।
