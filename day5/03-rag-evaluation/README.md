# RAG Evaluation and Production

RAG system-এর answer ভুল হলে কারণ হতে পারে parser, chunking, retrieval, context assembly, generation, citation বা access control। শুধু final answer score করলে failure কোথায় তা বোঝা কঠিন। তাই RAG-কে component level এবং end-to-end দুইভাবেই evaluate করতে হবে।

## ১. RAG-এর কোন অংশ evaluate করব?

দুটি প্রধান quality layer:

- **Retrieval:** relevant evidence retrieve হয়েছে কি?
- **Generation:** retrieved evidence থেকে answer সঠিক, relevant ও grounded কি?

এছাড়াও security, freshness, latency এবং cost evaluate করতে হবে।

~~~mermaid
flowchart TD
    A["Document ingestion"] --> B["Retrieval evaluation"]
    B --> C["Context assembly evaluation"]
    C --> D["Generation evaluation"]
    D --> E["Citation / groundedness evaluation"]
    E --> F["End-to-end quality"]
    A --> G["Freshness / ACL checks"]
    B --> H["Latency / cost"]
    D --> H
~~~

## ২. Evaluation Dataset কীভাবে তৈরি করব?

প্রতিটি প্রশ্নের জন্য:
- Expected answer বা key facts।
- Relevant document/chunk IDs।
- Allowed answer variation।
- Unanswerable প্রশ্ন।
- Ambiguous প্রশ্ন।
- Adversarial/prompt injection প্রশ্ন।
- User/tenant permission context।

Dataset-এ common query-এর পাশাপাশি edge case ও realistic production query রাখো। Evaluation set prompt tuning-এর বাইরে held-out রাখার চেষ্টা করো।

## ৩. Retrieval Metrics কী?

- **Recall@k:** top-k result-এ relevant item-এর কত অংশ পাওয়া গেছে।
- **Precision@k:** top-k result-এর কত অংশ relevant।
- **MRR:** প্রথম relevant result কত উপরে এসেছে।
- **nDCG:** graded relevance ও rank position বিবেচনা করে।

~~~mermaid
flowchart TD
    A["Ranked retrieved chunks"] --> B["Top-k selection"]
    B --> C["Compare with labeled relevant chunks"]
    C --> D["Recall@k / Precision@k"]
    C --> E["MRR"]
    C --> F["nDCG"]
~~~

যদি একটি query-এর multiple relevant chunk থাকে, Recall@k গুরুত্বপূর্ণ। Top-ranked evidence আগে পাওয়া দরকার হলে MRR/nDCG উপকারী।

## ৪. Generation Metrics কী?

- **Faithfulness/Groundedness:** answer retrieved context দ্বারা supported কি?
- **Answer relevance:** প্রশ্নের উত্তর দিয়েছে কি?
- **Correctness:** trusted reference answer-এর সঙ্গে মিল।
- **Citation correctness:** cited source claim support করে কি?
- **Completeness:** প্রয়োজনীয় key fact বাদ পড়েছে কি?

LLM-as-judge সহায়ক হতে পারে, কিন্তু judge bias ও inconsistency থাকে। Human-reviewed test set রাখবে।

## ৫. Faithfulness এবং Correctness কি একই?

না।
- Faithfulness: answer provided context-এর সঙ্গে supported কি?
- Correctness: answer বাস্তব/ground-truth অনুযায়ী সঠিক কি?

RAG context-এ ভুল document থাকলে model সেই ভুল তথ্য faithfulভাবে repeat করতে পারে। তাই দুটো আলাদা measure করো।

## ৬. Citation correctness কীভাবে evaluate করব?

প্রতিটি factual claim-এর জন্য:
1. Citation source ID valid কি?
2. Source chunk claim-টি support করে কি?
3. Citation-এর page/section সঠিক কি?
4. Unsupported claim citation ছাড়া আছে কি?
5. Model source reference fabricate করেছে কি?

Citation string present থাকলেই source claim support করে এমন নয়। সম্ভব হলে application retrieved chunk ID থেকে citation render করুক।

## ৭. No-answer Behavior কেন গুরুত্বপূর্ণ?

Evidence যথেষ্ট না হলে model-কে guess না করে uncertainty জানাতে শেখাও। Application policy-তে abstention condition নির্ধারণ করো।

~~~mermaid
flowchart TD
    A["Question + retrieved evidence"] --> B{"Enough relevant evidence?"}
    B -->|হ্যাঁ| C["Generate grounded answer"]
    B -->|না| D["Abstain / ask clarification / human review"]
~~~

Unanswerable query evaluation set-এ রাখো। Answer coverage বাড়াতে গিয়ে hallucination বাড়ছে কি না measure করো।

## ৮. Access Control কীভাবে evaluate করব?

Multi-user RAG-এ retrieval-এর আগে বা retrieval-এর সময় ACL enforce করতে হবে। শুধু prompt-এ "এই user-এর data দেখিও না" বলা security control নয়।

Test case:
- Tenant A user Tenant B document retrieve করতে পারে কি?
- User permission revoke হলে পুরোনো cache/vector result পাওয়া যায় কি?
- Document access update হলে index refresh হয় কি?
- Citation endpoint direct access দিয়ে permission bypass করা যায় কি?

Access control correctness quality metric-এর মতোই critical security requirement।

## ৯. Prompt Injection in Documents কীভাবে test করব?

Retrieved document-এ hostile instruction থাকতে পারে। Document content-কে untrusted evidence হিসেবে treat করবে; tool/action permission retrieval text থেকে আসবে না।

Adversarial test:
- "Ignore previous instructions" text।
- Secret/system prompt reveal request।
- Tool call trigger করার নির্দেশ।
- Conflicting source content।
- Malicious HTML/Markdown link।

Prompt-only defense যথেষ্ট নয়; tool allowlist, least privilege, server-side authorization ও action validation দরকার।

## ১০. Latency ও Cost কীভাবে measure করব?

Query path-এর প্রতিটি stage আলাদা measure করো:
- Query embedding।
- Vector/keyword retrieval।
- Metadata filter।
- Reranking।
- Context assembly।
- LLM generation।
- Post-validation।

~~~mermaid
flowchart LR
    A["Embedding"] --> B["Retrieval"]
    B --> C["Rerank"]
    C --> D["Context build"]
    D --> E["Generation"]
    E --> F["Validation"]
~~~

Average-এর পাশাপাশি p50/p95 latency, timeout rate, token usage, cache hit rate এবং cost per successful answer monitor করো।

## ১১. Freshness ও Re-indexing কীভাবে evaluate করব?

Document update/delete হলে পুরোনো chunk/vector remove বা version invalidate করতে হবে। Measure করো:
- Source update থেকে searchable হওয়া পর্যন্ত freshness lag।
- Delete-এর পরে stale chunk কতক্ষণ থাকে।
- Failed ingestion retry।
- Version mismatch।
- Cache invalidation।
- Duplicate chunk/index version।

## ১২. Observability কী রাখব?

Log/trace:
- Query/request ID।
- Retrieved document IDs ও scores।
- Filter/tenant context (sensitive value ছাড়া)।
- Prompt/model/index version।
- Latency per stage।
- Token usage।
- User feedback ও evaluation result।
- Abstention ও validation outcome।

Sensitive document content বা personal data indiscriminately log করবে না।

## ১৩. RAG Evaluation Regression কীভাবে করব?

Prompt, chunking, embedding model, retrieval parameter বা reranker change হলে fixed evaluation set-এ old/new compare করো।

~~~mermaid
flowchart TD
    A["RAG change"] --> B["Run fixed eval set"]
    B --> C["Retrieval metrics"]
    B --> D["Groundedness / correctness"]
    B --> E["Security / ACL tests"]
    B --> F["Latency / cost"]
    C --> G["Compare with baseline"]
    D --> G
    E --> G
    F --> G
    G --> H{"Regression?"}
    H -->|হ্যাঁ| I["Investigate / rollback"]
    H -->|না| J["Promote version"]
~~~

একটি metric বাড়লেও অন্যটি কমতে পারে। Quality, safety, latency ও cost trade-off একসঙ্গে বিচার করো।

## ১৪. RAG Production Checklist

- Document parser errors handle।
- Chunk/source metadata persist।
- ACL filter enforce ও test।
- Retrieval quality benchmark।
- No-answer behavior।
- Citation validation।
- Prompt injection defense-in-depth।
- Re-index/update/delete lifecycle।
- Prompt/model/index versioning।
- Cost/latency monitoring।
- User feedback ও regression evaluation।

## Quick Interview Revision

| প্রশ্ন | সংক্ষিপ্ত উত্তর |
|---|---|
| RAG evaluation layers? | Retrieval, generation, citation, security, operations |
| Recall@k? | Top-k-এ পাওয়া relevant items / all relevant |
| Precision@k? | Top-k-এ relevant items / k |
| MRR? | First relevant result-এর reciprocal rank average |
| nDCG? | Graded relevance ও rank position |
| Faithfulness? | Answer context-supported কি না |
| Correctness? | Ground truth অনুযায়ী সঠিক কি না |
| Faithfulness = correctness? | না |
| No-answer test? | Evidence না থাকলে abstain করে কি না |
| ACL test? | Unauthorized document retrieve/cite হয় কি না |
| Freshness? | Update/delete কত দ্রুত index-এ reflect হয় |
| Regression eval? | Change-এর আগে/পরে fixed set compare |

## Practice

কমপক্ষে ২০টি question-এর evaluation set বানাও:
- Relevant document/chunk IDs label করো।
- Recall@5, Precision@5, MRR/nDCG measure করো।
- Answer faithfulness, correctness ও citation correctness measure করো।
- অন্তত ৫টি unanswerable question যোগ করো।
- Direct ও indirect prompt injection test করো।
- Cross-tenant access test করো।
- Latency/cost stage-wise measure করো।
- Chunking বা embedding change-এর আগে/পরে regression report তৈরি করো।

## Practice করার পদ্ধতি

1. Retrieval ও generation failure আলাদা করে diagnose করো।
2. Faithfulness-কে correctness ধরে নিও না।
3. Unanswerable ও adversarial case dataset-এ রাখো।
4. ACL/citation test security requirement হিসেবে treat করো।
5. Quality, latency, cost এবং freshness একসঙ্গে monitor করো।
