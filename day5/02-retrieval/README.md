# Embeddings, Vector Search and Retrieval

RAG-এর answer quality অনেকটাই নির্ভর করে model-কে সঠিক evidence দেওয়া হচ্ছে কি না তার উপর। Retrieval stage-এর কাজ হলো user query-এর জন্য relevant document/chunk খুঁজে আনা। এখানে similarity metric, index, metadata filter, hybrid search এবং reranking গুরুত্বপূর্ণ।

## ১. Similarity Search কী?

Query embedding ও document embedding compare করে nearest candidate retrieve করা হয়। Similarity metric embedding model-এর training objective-এর সঙ্গে সামঞ্জস্যপূর্ণ হওয়া দরকার।

~~~mermaid
flowchart LR
    A["User query"] --> B["Query embedding"]
    C["Indexed document chunks"] --> D["Document embeddings"]
    B --> E["Similarity search"]
    D --> E
    E --> F["Top-k candidates"]
~~~

## ২. Cosine Similarity কী?

Cosine similarity vector-এর direction similarity মাপে:

~~~text
cosine(a,b) = (a·b) / (||a|| ||b||)
~~~

Magnitude normalize করে। Normalized vector-এ dot product cosine similarity-এর সমান হতে পারে।

## ৩. Dot Product এবং Euclidean Distance-এর পার্থক্য কী?

- **Dot product:** vector alignment ও magnitude দুটোতেই প্রভাবিত।
- **Cosine similarity:** vector direction-এর similarity।
- **Euclidean distance:** absolute geometric distance।

কোন metric ভালো তা embedding model ও index configuration অনুযায়ী নির্ভর করে। Embedding model documentation-এর recommended metric অনুসরণ করো এবং retrieval benchmark করো।

## ৪. Top-k Retrieval কী?

সবচেয়ে relevant kটি candidate ফেরত দেয়। k বেশি হলে recall বাড়তে পারে, কিন্তু noise, rerank cost ও context token-ও বাড়ে।

~~~mermaid
flowchart TD
    A["All indexed chunks"] --> B["Score against query"]
    B --> C["Sort by relevance"]
    C --> D["Select top-k"]
    D --> E["Candidate context"]
~~~

k একটি tuning parameter; dataset ও task অনুযায়ী Recall@k, Precision@k এবং answer quality measure করো।

## ৫. Exact Search এবং Approximate Nearest Neighbor (ANN) কী?

Exact search সব vector-এর সঙ্গে compare করে nearest result খুঁজে পেতে পারে, কিন্তু বড় collection-এ expensive। ANN index দ্রুত candidate দেয়, তবে কিছু nearest result miss করতে পারে।

Trade-off:
- Recall বনাম latency।
- Memory বনাম search speed।
- Index build time বনাম update cost।

HNSW ও IVF-এর মতো index family প্রচলিত। Database/vector engine অনুযায়ী implementation ও parameter আলাদা।

## ৬. Metadata Filtering কেন দরকার?

Tenant, document type, ACL, date, version ইত্যাদি filter দিয়ে candidate সীমিত করা যায়।

~~~python
# Pseudocode: actual API vector database অনুযায়ী আলাদা
results = vector_store.search(
    query_vector=query_embedding,
    top_k=10,
    filter={
        "tenant_id": current_user.tenant_id,
        "status": "published",
    },
)
~~~

Multi-tenant system-এ user authorization filter retrieval path-এ বাধ্যতামূলক হতে হবে। শুধু prompt-এ instruction দেওয়া security control নয়।

Pre-filtering retrieval candidate set-এর ভিতরে ACL enforce করতে পারে। Post-filtering-এ unauthorized result LLM context-এ যাওয়ার আগেই remove করতে হবে; post-filtering top-k কমিয়ে দিতে পারে, তাই over-fetch/recall strategy দরকার হতে পারে।

## ৭. Hybrid Search কী?

Semantic vector search ও lexical keyword search (যেমন BM25) combine করে। Exact identifier, code, product name ও semantic paraphrase একসঙ্গে ধরতে সহায়ক।

~~~mermaid
flowchart TD
    A["Query"] --> B["Dense vector search"]
    A --> C["Sparse / BM25 search"]
    B --> D["Candidate fusion"]
    C --> D
    D --> E["Reranking"]
    E --> F["Final top-k"]
~~~

দুই retriever-এর raw score সরাসরি যোগ করা সবসময় ঠিক নয়, কারণ score scale আলাদা হতে পারে। Rank-based fusion যেমন Reciprocal Rank Fusion (RRF) ব্যবহার করা যায়।

## ৮. Reranking কী?

প্রথম retrieval stage বেশি candidate আনে; reranker query-document pair-এর relevance score দিয়ে candidate re-order করে। Cross-encoder reranker query ও document একসঙ্গে দেখে score করতে পারে।

~~~mermaid
flowchart TD
    A["Query"] --> B["First-stage retrieval"]
    B --> C["Top 50 candidates"]
    C --> D["Cross-encoder reranker"]
    D --> E["Top 5 relevant chunks"]
~~~

Reranker latency/cost বাড়ায়, তাই candidate count ও final top-k evaluate করো। Reranking poor candidate recall ঠিক করতে পারে না; relevant document first-stage candidate-এ না থাকলে reranker সেটি খুঁজে পাবে না।

## ৯. Query Transformation কী?

Original query retrieval-এর জন্য ambiguous বা short হলে rewrite, expansion বা decomposition করা যায়।

- Query rewrite: clearer search query তৈরি।
- Multi-query: একই question-এর alternate formulation।
- Query decomposition: complex question sub-question-এ ভাগ।
- HyDE: hypothetical answer/document তৈরি করে তার embedding দিয়ে retrieval চেষ্টা।

এগুলো recall বাড়াতে পারে, কিন্তু generated query ভুল direction-এও যেতে পারে। Evaluation ছাড়া যোগ কোরো না; original query-ও retrieval candidate হিসেবে রাখা যায়।

## ১০. MMR (Maximal Marginal Relevance) কী?

Top result-এ একই ধরনের near-duplicate chunk থাকলে context redundant হয়। MMR relevance ও diversity balance করে candidate select করে।

~~~mermaid
flowchart TD
    A["Retrieved candidates"] --> B["Relevance score"]
    A --> C["Similarity to already selected chunks"]
    B --> D["Balance relevance and diversity"]
    C --> D
    D --> E["Diverse selected context"]
~~~

MMR helpful হতে পারে যখন document-এ repeated/overlapping information থাকে। তবে অতিরিক্ত diversity relevant evidence বাদ দিতে পারে।

## ১১. Contextual Retrieval কী?

Chunk-এর আশেপাশের heading বা document context যোগ করলে isolated chunk বোঝা সহজ হয়। Parent-child retrieval-এ ছোট child chunk retrieve করে বড় parent section context হিসেবে ব্যবহার করা যায়।

Trade-off:
- Better context ও disambiguation।
- Index size বাড়তে পারে।
- Token cost বাড়ে।
- Duplicate content বাড়তে পারে।

## ১২. Retrieval Failure-এর কারণ কী হতে পারে?

- Poor parsing বা OCR।
- Chunk boundary খারাপ।
- Wrong/incompatible embedding model।
- Query অস্পষ্ট বা acronym-heavy।
- Metadata filter ভুল।
- Relevant document index-এ নেই।
- top-k অপর্যাপ্ত।
- ANN recall কম।
- Permission filter অতিরিক্ত restrictive বা অনুপস্থিত।
- Stale index / deleted document।

## ১৩. Retrieval Debugging কীভাবে করব?

প্রথমে LLM বাদ দিয়ে query-এর top-k chunk inspect করো। Relevant evidence retrieve না হলে prompt পরিবর্তন করে লাভ হবে না।

Debugging sequence:
1. Query ও normalized query inspect।
2. Embedding model/version verify।
3. Metadata/ACL filter inspect।
4. Top-k candidates ও score inspect।
5. Exact/keyword search compare।
6. Reranker আগে/পরে ranking compare।
7. Relevant document index-এ আছে কি না verify।

~~~mermaid
flowchart TD
    A["Wrong final answer"] --> B{"Relevant evidence retrieved?"}
    B -->|না| C["Debug parsing, chunking, embedding, filters, index"]
    B -->|হ্যাঁ| D{"Evidence passed to prompt correctly?"}
    D -->|না| E["Debug context assembly / token truncation"]
    D -->|হ্যাঁ| F["Debug generation / prompt / grounding"]
~~~

## ১৪. Retrieval Benchmark কীভাবে তৈরি করব?

প্রতিটি query-এর জন্য relevant document/chunk ID label করো। এরপর:
- Recall@k।
- Precision@k।
- MRR।
- nDCG।
- Latency।
- Filter correctness।
- Per-tenant isolation test।

Human relevance label imperfect হতে পারে; ambiguous query এবং multiple valid evidence source document করো।

## Quick Interview Revision

| প্রশ্ন | সংক্ষিপ্ত উত্তর |
|---|---|
| Cosine similarity? | Vector direction similarity |
| Dot product? | Direction + magnitude |
| Top-k? | Highest-ranked k candidates |
| ANN? | Faster approximate nearest-neighbor search |
| Metadata filter? | Tenant/type/date/ACL filtering |
| Hybrid search? | Dense vector + lexical/BM25 |
| Reranker? | Candidates query relevance অনুযায়ী reorder |
| RRF? | Rank-based fusion |
| MMR? | Relevance ও diversity balance |
| HyDE? | Hypothetical document embedding দিয়ে retrieval |
| Parent-child? | Small chunk retrieve, larger context use |
| Reranker recall বাড়ায়? | First-stage miss করা candidate ফিরিয়ে আনতে পারে না |
| Retrieval debug? | LLM বাদ দিয়ে top-k inspect |

## Practice

1. একই প্রশ্নে keyword search, vector search ও hybrid search compare করো।
2. Top-k ফলাফলে relevant chunk আছে কি না manually label করো।
3. Cosine ও dot product compare করো normalized/unnormalized vector দিয়ে।
4. ANN index parameter বদলে recall-latency trade-off measure করো।
5. Metadata/ACL filter test করো, unauthorized chunk যেন না আসে।
6. Reranker-এর আগে/পরে ranking compare করো।
7. MMR দিয়ে near-duplicate chunk কমাও।
8. Multi-query বা query rewrite test করো।
9. Retrieval failure-এর top-k evidence inspect করে root cause classify করো।
10. Recall@k, MRR ও nDCG report করো।

## Practice করার পদ্ধতি

1. Retrieval metric optimize করার আগে relevant chunk label করো।
2. First-stage recall এবং reranker quality আলাদা evaluate করো।
3. Score scale না বুঝে dense ও sparse raw score যোগ কোরো না।
4. ACL filter correctness-কে security test হিসেবে ধরো।
5. Latency, recall, context cost ও answer quality একসঙ্গে compare করো।
