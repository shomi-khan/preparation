# RAG Architecture

LLM-এর training knowledge সবসময় current, private বা organization-specific হয় না। ধরো, একটি assistant-কে company policy, internal engineering docs বা latest product catalog থেকে উত্তর দিতে হবে। Model fine-tune না করেও inference-এর সময় relevant document এনে prompt-এ যোগ করা যায়। এই architecture-ই **Retrieval-Augmented Generation (RAG)**।

## ১. RAG কী?

RAG হলো এমন architecture যেখানে LLM answer generate করার আগে external knowledge source থেকে relevant information retrieve করে। এতে model-এর parametric knowledge-এর বাইরে private বা updated data ব্যবহার করা যায়।

~~~mermaid
flowchart TD
    A["User question"] --> B["Retrieve relevant knowledge"]
    B --> C["Build grounded context"]
    C --> D["LLM generates answer"]
    D --> E["Answer with source references"]
~~~

RAG hallucination পুরোপুরি দূর করে না; retrieval ভুল হলে বা model evidence ভুলভাবে interpret করলে answer-ও ভুল হতে পারে।

## ২. RAG কেন ব্যবহার করব?

- Private document থেকে প্রশ্নের উত্তর।
- Frequently updated knowledge।
- Source citation ও traceability।
- Model fine-tuning ছাড়াই domain knowledge access।
- Knowledge update করতে model weights retrain না করা।

RAG উপযুক্ত যখন answer-এর evidence external source-এ থাকে। Model behavior/style adaptation-এর জন্য fine-tuning আলাদা solution হতে পারে।

## ৩. RAG-এর দুইটি প্রধান pipeline কী?

RAG system সাধারণত দুটি flow-তে ভাগ হয়:

1. **Ingestion / indexing:** document process করে searchable index তৈরি।
2. **Query / retrieval:** user query থেকে relevant evidence এনে answer তৈরি।

~~~mermaid
flowchart TD
    A["Documents"] --> B["Ingestion pipeline"]
    B --> C["Search index / vector store"]
    D["User query"] --> E["Query pipeline"]
    C --> E
    E --> F["Grounded answer"]
~~~

Ingestion সাধারণত offline/background job; query path latency-sensitive।

## ৪. Ingestion Pipeline কীভাবে কাজ করে?

~~~mermaid
flowchart TD
    A["Documents / data sources"] --> B["Parse and extract"]
    B --> C["Clean and normalize"]
    C --> D["Split into chunks"]
    D --> E["Attach metadata / ACL"]
    E --> F["Generate embeddings"]
    F --> G["Store text, vectors and metadata"]
    G --> H["Versioned search index"]
~~~

প্রতিটি stage-এর failure track করো। Parsing error হলে silently incomplete document index কোরো না।

## ৫. Query Pipeline কীভাবে কাজ করে?

~~~mermaid
flowchart TD
    A["User question"] --> B["Authenticate and authorize"]
    B --> C["Query rewrite / normalization"]
    C --> D["Retrieve candidates"]
    D --> E["Metadata / ACL filter"]
    E --> F["Rerank / deduplicate"]
    F --> G["Build context within token budget"]
    G --> H["LLM generation"]
    H --> I["Validate answer and citations"]
    I --> J["Return answer / abstain"]
~~~

Retrieval failure এবং generation failure আলাদা debug করতে হবে। LLM prompt improve করলেই poor retrieval ঠিক হবে না।

## ৬. Document Parsing কেন গুরুত্বপূর্ণ?

PDF, DOCX, HTML, Markdown থেকে text extract করতে হয়। Scanned PDF হলে OCR দরকার হতে পারে। Tables, headings, page number ও layout হারিয়ে গেলে retrieval quality কমে।

Document parsing-এ বিবেচনা করো:
- Page/section boundary।
- Table structure।
- Header/footer noise।
- OCR confidence।
- Source document ID ও version।
- Parser failure ও unsupported file type।

## ৭. Chunking কী এবং কেন দরকার?

Document ছোট passage-এ ভাগ করা হয়, যাতে retrieval relevant অংশ ফেরত দিতে পারে। পুরো document এক chunk হলে query-specific evidence খুঁজে পাওয়া কঠিন; অতিরিক্ত ছোট chunk হলে context হারায়।

Common strategy:
- **Fixed-size:** সহজ; কিন্তু sentence/section ভাঙতে পারে।
- **Recursive:** separator hierarchy দিয়ে natural boundary preserve করার চেষ্টা।
- **Semantic:** content meaning/structure অনুযায়ী split।
- **Parent-child:** ছোট child chunk retrieve, বড় parent context ব্যবহার।

~~~mermaid
flowchart TD
    A["Long document"] --> B["Chunking strategy"]
    B --> C["Chunk 1"]
    B --> D["Chunk 2"]
    B --> E["Chunk 3"]
    C --> F["Metadata + embedding"]
    D --> F
    E --> F
~~~

Chunk size ও overlap dataset/task অনুযায়ী evaluate করতে হবে। বেশি overlap duplicate context ও token cost বাড়াতে পারে।

## ৮. Metadata কেন দরকার?

প্রতিটি chunk-এ source ID, document name, page, section, tenant/ACL, timestamp, version রাখা যেতে পারে।

Metadata কাজে লাগে:
- Source citation।
- Filtering by document type/date/tenant।
- Access control।
- Freshness ও re-indexing।
- Debugging retrieval result।

Metadata filter security boundary-এর অংশ হতে পারে, কিন্তু authorization server-side enforce করতে হবে।

## ৯. Embedding কী এবং compatibility কেন জরুরি?

Embedding model text-কে vector-এ রূপান্তর করে। Query ও document embedding compatible embedding space/model ব্যবহার করা দরকার। Model/version বদলালে পুরোনো vector-এর সঙ্গে নতুন query vector compare করা invalid বা degraded হতে পারে; re-embedding প্রয়োজন হতে পারে।

## ১০. Vector Store কী?

Vector database বা search engine embedding store ও nearest-neighbor retrieval করতে পারে। PostgreSQL pgvector, FAISS, Milvus, Qdrant ইত্যাদি option আছে।

Selection বিবেচনা:
- Dataset size ও update frequency।
- Metadata filtering।
- Tenant isolation / ACL.
- Exact বনাম approximate search।
- Backup, replication ও operational complexity।
- Cost, latency ও ecosystem।

Vector store মানেই full RAG নয়; parsing, chunking, retrieval policy ও answer validation-ও দরকার।

## ১১. Hybrid Search কেন ব্যবহার করব?

Vector search semantic similarity ধরে; lexical search exact keyword/identifier match করতে পারে। Hybrid search দুটো combine করে।

~~~mermaid
flowchart TD
    A["User query"] --> B["Vector retrieval"]
    A --> C["Keyword / BM25 retrieval"]
    B --> D["Merge candidates"]
    C --> D
    D --> E["Rerank"]
    E --> F["Top relevant chunks"]
~~~

Exact product ID, code symbol, policy number ও semantic paraphrase একসঙ্গে ধরতে hybrid search সহায়ক।

## ১২. Reranking কী?

প্রথম retrieval stage দ্রুত বেশি candidate আনে। Reranker query-document pair-এর relevance score দিয়ে candidate re-order করে। এটি quality বাড়াতে পারে, তবে latency ও cost বাড়ায়।

Reranker top-k retrieval-এর বিকল্প নয়; সাধারণত candidate retrieval-এর পর second-stage ranking।

## ১৩. Context Construction কীভাবে করব?

Retrieved chunks prompt-এ যোগ করার সময়:
- Context token budget।
- Source ordering।
- Duplicate removal।
- Chunk adjacency/parent context।
- Conflicting document version।
- Untrusted content handling।
- Source attribution।

সব retrieved text prompt-এ ঢোকানো উচিত নয়। Relevant evidence নির্বাচন করো এবং answer-এর সঙ্গে source metadata preserve করো।

## ১৪. Citation কীভাবে তৈরি হবে?

Chunk-এর source metadata সংরক্ষণ করলে answer-এর supporting source/page reference দেওয়া যায়। Citation সত্যিই claim support করে কি না evaluate করতে হবে। Model citation বানিয়ে দিতে পারে; তাই application-এর retrieved source ID থেকে citation render করা safer।

## ১৫. Access Control কীভাবে enforce করব?

Multi-tenant RAG-এ user-এর authorization অনুযায়ী retrieval filter enforce করতে হবে। শুধু prompt-এ "এই user-এর data দেখিও না" বলা security control নয়।

~~~mermaid
flowchart TD
    A["Authenticated user"] --> B["Resolve tenant and permissions"]
    B --> C["Authorized retrieval filter"]
    C --> D["Retrieve allowed chunks only"]
    D --> E["LLM context"]
    E --> F["Answer"]
~~~

Access filter query-র আগে বা retrieval engine-এর ভিতরে enforce করো এবং result-এ post-check করো। Unauthorized chunk LLM context-এ ঢুকতে দেওয়া যাবে না।

## ১৬. Document update ও delete কীভাবে handle করব?

Document পরিবর্তিত হলে পুরোনো chunk/vector stale থাকতে পারে। Version-aware ingestion রাখো:
- Document ID ও version।
- Ingestion timestamp।
- Chunk IDs।
- Delete/tombstone event।
- Re-index status।
- Failed ingestion retry।

Delete request হলে source document ছাড়াও derived chunks, embeddings ও cache invalidation policy বিবেচনা করো।

## ১৭. RAG-এর common failure mode কী?

| Failure | সম্ভাব্য কারণ |
|---|---|
| Relevant chunk নেই | Parsing/chunking/embedding/query issue |
| Wrong chunk | Retrieval metric/index/filter issue |
| Correct chunk, wrong answer | Prompt/generation issue |
| Citation mismatch | Source mapping/claim attribution issue |
| Unauthorized data | ACL/tenant filter failure |
| Stale answer | Re-index/cache invalidation issue |
| High latency | Rerank, vector search, provider latency |
| High cost | Too many/large chunks, oversized prompt |

## Quick Interview Revision

| প্রশ্ন | সংক্ষিপ্ত উত্তর |
|---|---|
| RAG কী? | Retrieve external evidence, then generate |
| RAG কেন? | Private/fresh knowledge, citations |
| Ingestion vs query pipeline? | Index creation vs online retrieval/generation |
| Chunking কেন? | Query-relevant passage retrieve করতে |
| Overlap বেশি হলে? | Duplicate context ও token cost |
| Metadata কেন? | Filtering, citation, ACL, version |
| Embedding compatibility? | Query/doc vector একই compatible space-এ |
| Hybrid search? | Vector + lexical retrieval |
| Reranking? | Candidate-কে query relevance অনুযায়ী reorder |
| RAG hallucination eliminate করে? | না |
| ACL কোথায় enforce? | Server/retrieval boundary-তে |
| Document delete? | Derived chunks/vector/cache-ও invalidate |

## Practice

1. একটি PDF থেকে text extract করে ১০-২০টি chunk তৈরি করো।
2. প্রতিটি chunk-এ source, page, section ও chunk index metadata রাখো।
3. Query দিয়ে top-k retrieval করে result inspect করো।
4. Vector ও keyword search compare করো।
5. Reranker যোগ করে ranking change evaluate করো।
6. Unanswerable query দিয়ে abstention behavior test করো।
7. Tenant metadata filter implement করে unauthorized retrieval test করো।
8. Document update/delete করে stale vector cleanup verify করো।
