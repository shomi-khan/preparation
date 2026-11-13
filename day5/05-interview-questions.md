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
