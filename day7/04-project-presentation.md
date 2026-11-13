# Project Presentation Guide

## Suggested project: Document Q&A with RAG

৫-১০ মিনিটে project explain করার কাঠামো:

### ১. Problem
কোন user problem solve করছ? কেন সাধারণ keyword search যথেষ্ট নয়?

### ২. Requirements
Document type, user roles, source citation, latency, privacy ও scale assumptions।

### ৩. Architecture
Ingestion ও query flow diagram দেখাও। API, object storage, database, embedding model, vector index ও LLM-এর দায়িত্ব explain করো।

### ৪. Key Design Decisions
- Chunking strategy কেন?
- Embedding model কেন?
- Vector/hybrid retrieval কেন?
- top-k ও reranker কীভাবে বেছে নিয়েছ?
- Source metadata কীভাবে রেখেছ?

### ৫. Reliability and Security
Timeout, retry, validation, tenant ACL, prompt injection, secret management ও error handling।

### ৬. Evaluation
নিজস্ব test questions, Recall@k, answer relevance, faithfulness, citation correctness ও latency। শুধু কয়েকটি ভালো demo দেখিয়ে quality claim করবে না।

### ৭. Limitations
যে document parsing ব্যর্থ হয়, unsupported query, multilingual limitation, latency/cost, access control gap ইত্যাদি সৎভাবে বলো।

### ৮. Future Improvements
Hybrid retrieval, reranker, better OCR, evaluation expansion, caching, observability বা human review।

## Sample Presentation Opening
"আমি একটি document question-answering system তৈরি করেছি, যার লক্ষ্য হলো user-কে বড় document collection থেকে relevant evidence খুঁজে source reference-সহ উত্তর দেওয়া। আমি ingestion এবং query path আলাদা রেখেছি, যাতে indexing asynchronous এবং question answering responsive থাকে।"

এটি নিজের implementation অনুযায়ী edit করবে; বাস্তবে implement না করা feature নিজের project-এ আছে বলে দাবি করবে না।

## Demo Checklist
- [ ] Sample documents ready।
- [ ] At least 3 answerable questions।
- [ ] At least 2 unanswerable questions।
- [ ] Citation/page reference visible।
- [ ] Permission test।
- [ ] Failure case demo।
- [ ] Architecture diagram।
- [ ] Evaluation result।
