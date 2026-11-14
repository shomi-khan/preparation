# RAG Evaluation and Production

## ১. RAG-এর কোন অংশ evaluate করবে?
দুটি স্তর:
- Retrieval: relevant evidence retrieve হয়েছে কি?
- Generation: retrieved evidence থেকে answer সঠিক ও grounded কি?

End-to-end answer score দিয়ে retrieval ও generation failure আলাদা বোঝা কঠিন।

## ২. Retrieval Metrics
- Recall@k: top-k result-এ relevant item-এর কত অংশ পাওয়া গেছে।
- Precision@k: top-k result-এর কত অংশ relevant।
- MRR: প্রথম relevant result কত উপরে এসেছে।
- nDCG: relevance grade ও rank position বিবেচনা করে।

## ৩. Generation Metrics
- Faithfulness/Groundedness: answer context-supported কি?
- Answer relevance: প্রশ্নের উত্তর দিয়েছে কি?
- Correctness: trusted reference answer-এর সঙ্গে মিল।
- Citation correctness: cited source claim support করে কি?

LLM-as-judge সহায়ক হতে পারে, কিন্তু judge bias ও inconsistency থাকে। Human-reviewed test set রাখবে।

## ৪. Evaluation Dataset
প্রতিটি প্রশ্নের জন্য:
- Expected answer বা key facts।
- Relevant document/chunk IDs।
- Allowed answer variation।
- Unanswerable প্রশ্ন।
- Adversarial/ambiguous প্রশ্ন।

## ৫. No-answer Behavior
Evidence যথেষ্ট না হলে model-কে guess না করে uncertainty জানাতে শেখাও। Application policy-তে abstention condition নির্ধারণ করো।

## ৬. Access Control
Multi-user RAG-এ retrieval-এর আগে বা retrieval-এর সময় ACL enforce করতে হবে। শুধু prompt-এ "এই user-এর data দেখিও না" বলা security control নয়। Tenant isolation ও authorization test আবশ্যক।

## ৭. Prompt Injection in Documents
Retrieved document-এ hostile instruction থাকতে পারে। Document content-কে untrusted evidence হিসেবে treat করবে; tool/action permission retrieval text থেকে আসবে না।

## ৮. Latency ও Cost
Parsing/indexing offline করা যায়। Query path-এ embedding, retrieval, rerank ও generation latency measure করো। Cache query/result কেবল permission-aware key দিয়ে করবে।

## ৯. Freshness ও Re-indexing
Document update/delete হলে পুরোনো chunk/vector remove বা version invalidate করতে হবে। Stale data retrieval security ও correctness issue হতে পারে।

## ১০. Observability
Log/trace:
- Query/request ID
- Retrieved document IDs ও scores
- Filter/tenant context (sensitive value ছাড়া)
- Prompt/model/index version
- Latency per stage
- Token usage
- User feedback ও evaluation result

## ১১. RAG Production Checklist
- Document parser errors handle।
- Chunk/source metadata persist।
- ACL filter enforce।
- Retrieval quality benchmark।
- No-answer behavior।
- Citation validation।
- Prompt injection defense-in-depth।
- Re-index/update/delete lifecycle।
- Cost/latency monitoring।

## Practice
কমপক্ষে ২০টি question-এর evaluation set বানাও। Recall@5, answer faithfulness ও citation correctness measure করো। অন্তত ৫টি unanswerable question যোগ করো।
