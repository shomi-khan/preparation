# Mock Interview Questions

## Round A: Python and Backend

1. Python iterable ও iterator-এর পার্থক্য?
2. Generator memory কীভাবে বাঁচায়?
3. Mutable default argument-এর bug দেখাও।
4. Decorator কীভাবে implement করবে?
5. Context manager exception হলেও cleanup কীভাবে নিশ্চিত করে?
6. Asyncio কখন threading-এর চেয়ে উপযোগী?
7. CPU-bound কাজ কীভাবে parallel করবে?
8. Pydantic runtime-এ কী validate করে?
9. ১০ GB CSV কীভাবে process করবে?
10. একটি FastAPI endpoint-এর timeout/retry strategy কী?

## Round B: ML and Data

1. Classification ও regression-এর পার্থক্য?
2. Overfitting কীভাবে detect করবে?
3. Data leakage-এর উদাহরণ?
4. Random Forest কীভাবে কাজ করে?
5. KNN-এ scaling কেন জরুরি?
6. Imbalanced data-তে কোন metric দেখবে?
7. Precision ও recall trade-off কী?
8. Cross-validation কী?
9. SQL window function কী?
10. Customer-এর latest transaction query লেখো।

## Round C: LLM and RAG

1. Token ও embedding কী?
2. Transformer self-attention কীভাবে কাজ করে?
3. Hallucination কেন হয়?
4. RAG ও fine-tuning কখন বেছে নেবে?
5. Chunk size কীভাবে tune করবে?
6. Hybrid search কেন?
7. Retrieval ও generation আলাদা evaluate করবে কীভাবে?
8. Citation correctness কীভাবে যাচাই করবে?
9. Prompt injection কীভাবে mitigate করবে?
10. User A-এর document User B-কে retrieve না হতে কী করবে?

## Round D: Agents and Production

1. Workflow ও agent-এর পার্থক্য?
2. Tool calling cycle explain করো।
3. SQL agent secure করবে কীভাবে?
4. Agent infinite loop আটকাবে কীভাবে?
5. Human approval কখন লাগবে?
6. Retry ও idempotency কেন গুরুত্বপূর্ণ?
7. LLM cost কীভাবে monitor করবে?
8. Prompt/model versioning কেন?
9. Provider outage হলে কী করবে?
10. Production quality কীভাবে evaluate করবে?

## System Design Prompts

### Prompt 1
Enterprise PDF Q&A system design করো। Requirements, ingestion, storage, retrieval, ACL, generation, evaluation, scaling ও monitoring explain করো।

### Prompt 2
একটি customer-support AI assistant design করো যা ticket classify, knowledge base search এবং প্রয়োজন হলে human agent-এ escalate করে। Tool permission ও audit trail explain করো।

## Answer Framework
প্রতিটি উত্তর:
1. Definition / direct answer।
2. Practical example।
3. Trade-off / limitation।
4. Production concern।
5. কীভাবে test বা measure করবে।

কোনো উত্তর না জানলে bluff না করে assumption পরিষ্কার করে reasoning দেখাও।
