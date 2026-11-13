# Embeddings, Vector Search and Retrieval

## ১. Similarity Search
Query embedding ও document embedding compare করে nearest candidate retrieve করা হয়। Similarity metric embedding model-এর training objective-এর সঙ্গে সামঞ্জস্যপূর্ণ হওয়া দরকার।

## ২. Cosine Similarity
`cosine(a,b) = (a·b) / (||a|| ||b||)`। Vector-এর direction similarity মাপে; magnitude normalize করে। Normalized vector-এ dot product cosine similarity-এর সমান হতে পারে।

## ৩. Dot Product ও Euclidean Distance
Dot product vector alignment ও magnitude দুটোতেই প্রভাবিত। Euclidean distance absolute geometric distance মাপে। কোন metric ভালো তা embedding model ও index configuration অনুযায়ী নির্ভর করে।

## ৪. Top-k Retrieval
সবচেয়ে relevant kটি candidate ফেরত দেয়। k বেশি হলে recall বাড়তে পারে, কিন্তু noise ও context cost-ও বাড়তে পারে।

## ৫. Approximate Nearest Neighbor (ANN)
বড় vector collection-এ exact search ব্যয়বহুল হতে পারে। ANN index দ্রুত candidate দেয়, তবে কিছু nearest result miss করতে পারে। Recall-latency trade-off tune করতে হয়।

## ৬. Metadata Filtering
Tenant, document type, ACL, date ইত্যাদি filter দিয়ে candidate সীমিত করা যায়। Multi-tenant system-এ user authorization filter query-র আগেই enforce এবং retrieval path-এ বাধ্যতামূলক করতে হবে।

## ৭. Hybrid Search
Semantic vector search ও lexical keyword search (যেমন BM25) combine করে। Exact identifier, code, product name ও semantic paraphrase একসঙ্গে ধরতে সহায়ক।

## ৮. Reranking
প্রথম retrieval stage বেশি candidate আনে; reranker query-document pair-এর relevance score দিয়ে candidate re-order করে। এটি latency/cost বাড়ায়, তাই প্রয়োজন অনুযায়ী ব্যবহার।

## ৯. Query Transformation
Query rewrite, multi-query বা HyDE retrieval improve করতে পারে, কিন্তু generated query ভুল direction-এও যেতে পারে। Evaluation ছাড়া যোগ করবে না।

## ১০. Contextual Retrieval
Chunk-এর আশেপাশের heading বা document context যোগ করলে isolated chunk বোঝা সহজ হয়। তবে index size ও token cost বাড়তে পারে।

## ১১. Retrieval Failure
সম্ভাব্য কারণ:
- Poor parsing বা OCR।
- Chunk boundary খারাপ।
- Wrong embedding model।
- Query অস্পষ্ট।
- Metadata filter ভুল।
- Relevant document index-এ নেই।
- top-k অপর্যাপ্ত।
- Permission filter অতিরিক্ত restrictive বা অনুপস্থিত।

## ১২. Retrieval Debugging
প্রথমে LLM বাদ দিয়ে query-এর top-k chunk inspect করো। Relevant evidence retrieve না হলে prompt পরিবর্তন করে লাভ হবে না।

## Practice
একই প্রশ্নে keyword search, vector search ও hybrid search compare করো। Top-k ফলাফলে relevant chunk আছে কি না manually label করো।
