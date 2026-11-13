# RAG Architecture

## ১. RAG কী?
Retrieval-Augmented Generation হলো এমন architecture যেখানে LLM answer generate করার আগে external knowledge source থেকে relevant information retrieve করে। এতে model-এর parametric knowledge-এর বাইরে private বা updated data ব্যবহার করা যায়।

## ২. RAG কেন?
- Private document থেকে প্রশ্নের উত্তর।
- Frequently updated knowledge।
- Source citation ও traceability।
- Model fine-tuning ছাড়াই domain knowledge access।

RAG hallucination পুরোপুরি দূর করে না; retrieval ভুল হলে answer-ও ভুল হতে পারে।

## ৩. Ingestion Pipeline
```text
Documents
 -> Parse / Extract
 -> Clean
 -> Split into Chunks
 -> Attach Metadata
 -> Generate Embeddings
 -> Store in Vector / Search Index
```

## ৪. Query Pipeline
```text
User Question
 -> Query Processing
 -> Retrieve Candidates
 -> Filter / Rerank
 -> Build Context
 -> LLM Generate
 -> Validate / Cite Sources
 -> Answer
```

## ৫. Document Parsing
PDF, DOCX, HTML, Markdown থেকে text extract করতে হয়। Scanned PDF হলে OCR দরকার হতে পারে। Tables, headings, page number ও layout হারিয়ে গেলে retrieval quality কমে।

## ৬. Chunking
Document ছোট passage-এ ভাগ করা হয়, যাতে retrieval relevant অংশ ফেরত দিতে পারে।
- Fixed-size chunk: সহজ, কিন্তু sentence/section ভাঙতে পারে।
- Recursive chunking: separator hierarchy ব্যবহার করে।
- Semantic chunking: content meaning/structure অনুযায়ী।
- Parent-child: ছোট chunk retrieve, বড় parent context ব্যবহার।

Chunk size ও overlap dataset/task অনুযায়ী evaluate করতে হবে। বেশি overlap duplicate context বাড়াতে পারে।

## ৭. Metadata
প্রতিটি chunk-এ source ID, document name, page, section, tenant/ACL, timestamp, version রাখা যেতে পারে। Metadata filter access control-এর অংশ হতে পারে, কিন্তু authorization server-side enforce করতে হবে।

## ৮. Embedding
Embedding model text-কে vector-এ রূপান্তর করে। Query ও document embedding একই compatible embedding space/model ব্যবহার করা দরকার। Model পরিবর্তন করলে index re-embed লাগতে পারে।

## ৯. Vector Store
Vector database বা search engine embedding store ও nearest-neighbor retrieval করতে পারে। PostgreSQL pgvector, FAISS, Milvus, Qdrant ইত্যাদি option আছে। Selection scale, filtering, operations ও ecosystem-এর উপর নির্ভর করে।

## ১০. Context Construction
Retrieved chunks prompt-এ যোগ করা হয়। Context budget, source ordering, duplicate removal এবং untrusted text handling গুরুত্বপূর্ণ।

## ১১. Citation
Chunk-এর source metadata সংরক্ষণ করলে answer-এর supporting source/page reference দেওয়া যায়। Citation সত্যিই claim support করে কি না evaluate করতে হবে।

## Practice
একটি PDF থেকে ১০-২০টি chunk তৈরি করো। প্রতিটি chunk-এ source, page ও chunk index metadata রাখো। Query দিয়ে top-k retrieval করে result inspect করো।
