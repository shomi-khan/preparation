# Day 5 Practical Exercises

## Exercise 1: Document Ingestion
একটি PDF, একটি Markdown file ও একটি HTML page থেকে text extract করো। Source, page/section metadata রাখো।

## Exercise 2: Chunking Experiment
Chunk size ও overlap-এর ৩টি configuration পরীক্ষা করো। একই প্রশ্নে কোন configuration relevant evidence ফেরত দেয় তা compare করো।

## Exercise 3: Embedding Search
Embedding তৈরি করে vector store-এ রাখো। Query embedding দিয়ে top-5 chunk retrieve করো। Similarity score inspect করো।

## Exercise 4: Hybrid Retrieval
Keyword search ও vector search-এর result combine করো। Identifier-heavy query এবং natural language query-তে পার্থক্য দেখো।

## Exercise 5: RAG Answering
Retrieved chunk prompt-এ দিয়ে answer generate করো। প্রতিটি claim-এর source citation ফেরত দাও। Evidence না থাকলে abstain করো।

## Exercise 6: Retrieval Evaluation
২০টি প্রশ্নের জন্য relevant chunk label করো। Recall@k ও MRR calculate করো।

## Exercise 7: Access Control
দুইটি user/tenant-এর document index করো। নিশ্চিত করো এক tenant অন্য tenant-এর chunk retrieve করতে পারে না। শুধু prompt instruction দিয়ে নয়, server-side filter দিয়ে enforce করো।

## Exercise 8: Failure Analysis
কমপক্ষে ৫টি ভুল answer inspect করে failure classify করো: parsing, chunking, embedding, retrieval, context assembly, generation, citation বা permission।

## Deliverables
- Runnable PDF Q&A prototype।
- Ingestion ও query architecture diagram।
- Evaluation dataset ও metric report।
- Access control test।
- Known limitation document।

## Self-review
- [ ] Chunk metadata রাখি।
- [ ] Retrieval result inspect করি।
- [ ] Retrieval ও generation আলাদা evaluate করি।
- [ ] No-answer case test করি।
- [ ] Citation verify করি।
- [ ] ACL enforce করি।
- [ ] Prompt injection threat বিবেচনা করি।
