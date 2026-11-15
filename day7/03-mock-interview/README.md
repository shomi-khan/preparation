# Mock Interview Questions

এই mock interview guide-টি শুধু প্রশ্নের তালিকা নয়। প্রতিটি প্রশ্নে direct answer, practical example, trade-off এবং failure mode explain করার অনুশীলন করবে। নিজের উত্তর record করে সময়, clarity ও technical depth review করো।

## Interview Simulation Format

| Round | Focus | Suggested time |
|---|---|---:|
| A | Python and Backend | 20 min |
| B | ML and Data | 20 min |
| C | LLM and RAG | 20 min |
| D | Agents and Production | 20 min |
| E | System Design | 25 min |
| F | Project Presentation | 10 min |

প্রতিটি round-এ interviewer follow-up করবে। Definition মুখস্থ করে থেমে যেও না।

## Round A: Python and Backend

1. Python iterable ও iterator-এর পার্থক্য?
2. Generator memory কীভাবে বাঁচায়? কখন generator ব্যবহার করবে না?
3. Mutable default argument-এর bug দেখাও।
4. Shallow copy ও deep copy-এর পার্থক্য?
5. Decorator কীভাবে implement করবে? wraps কেন?
6. Context manager exception হলেও cleanup কীভাবে নিশ্চিত করে?
7. Asyncio কখন threading-এর চেয়ে উপযোগী?
8. CPU-bound কাজ কীভাবে parallel করবে?
9. Pydantic runtime-এ কী validate করে? Type hints-এর সঙ্গে পার্থক্য?
10. ১০ GB CSV কীভাবে process করবে?
11. একটি FastAPI endpoint-এর timeout/retry strategy কী?
12. List, set, dict এবং deque-এর complexity compare করো।

### Follow-up probes
- Shared mutable state থাকলে কী হবে?
- Exception cleanup-এর সময় suppress করা উচিত কি?
- Async code-এ blocking library call করলে কী হবে?
- GIL-এর বক্তব্য কোন CPython build/version-এর জন্য বলছ?

## Round B: ML and Data

1. Classification ও regression-এর পার্থক্য?
2. Overfitting কীভাবে detect করবে?
3. Data leakage-এর উদাহরণ?
4. Random Forest কীভাবে কাজ করে?
5. KNN-এ scaling কেন জরুরি?
6. Imbalanced data-তে কোন metric দেখবে?
7. Precision ও recall trade-off কী?
8. Cross-validation কী?
9. Group/time-based split কখন দরকার?
10. SQL window function কী?
11. Customer-এর latest transaction query লেখো।
12. Model probability 0.8 দিলে কি 80% certainty?
13. MAE, RMSE ও R²-এর পার্থক্য?

### Follow-up probes
- Test set দিয়ে threshold tune করলে কী সমস্যা?
- Scaler কোথায় fit করবে?
- PR-AUC কখন ROC-AUC-এর চেয়ে informative?
- Production data drift দেখলে কী করবে?

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
11. Faithfulness ও correctness-এর পার্থক্য?
12. Context window বড় হলেই answer quality বাড়ে?

### Follow-up probes
- Relevant chunk top-k-তে না এলে কী করবে?
- Reranker first-stage retrieval-এর বিকল্প কি?
- Unanswerable query কীভাবে detect করবে?
- Document update/delete হলে stale vector কীভাবে invalidate করবে?

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
11. Agent checkpoint কেন দরকার?
12. Tool output-এ prompt injection থাকলে কী করবে?

### Follow-up probes
- Model tool call propose করলে কি সেটি execute হবে?
- Approval-এর পরে resource state বদলে গেলে?
- Same idempotency key আবার এলে?
- Agent final answer correct হলেও trajectory unsafe হতে পারে কি?

## Round E: System Design Prompts

### Prompt 1: Enterprise Document Q&A

Enterprise PDF Q&A system design করো। Requirements, scale estimate, ingestion, storage, retrieval, ACL, generation, evaluation, reliability ও monitoring explain করো।

### Prompt 2: Customer Support AI Assistant

একটি customer-support AI assistant design করো যা:
- Ticket classify করে।
- Knowledge base search করে।
- Customer/account তথ্য read-only lookup করে।
- প্রয়োজন হলে human agent-এ escalate করে।
- Refund বা account change-এর মতো action-এ approval নেয়।

Tool permission, idempotency, audit trail ও state lifecycle explain করো।

### Prompt 3: Async Batch Document Processing

বড় document batch ingest করার service design করো। Queue, worker, retries, idempotency, partial failure, DLQ, re-indexing ও progress reporting explain করো।

## Round F: Project Presentation

নিজের একটি project ৫-১০ মিনিটে explain করো:
- Problem ও user।
- Architecture।
- তোমার actual contribution।
- Key design decisions।
- Evaluation evidence।
- Limitation ও failure।
- Next improvement।

## Coding / Whiteboard Tasks

1. Generator দিয়ে large file line-by-line process করো।
2. Decorator দিয়ে execution time measure করো।
3. Async function-এ semaphore ও timeout যোগ করো।
4. Pandas দিয়ে group-wise aggregation করো।
5. SQL দিয়ে latest row per group বের করো।
6. Scikit-learn Pipeline বানাও।
7. Confusion matrix থেকে precision/recall হিসাব করো।
8. Vector retrieval-এর top-k result evaluate করো।
9. Pydantic schema দিয়ে tool argument validate করো।
10. Bounded agent loop-এর pseudocode লেখো।

## Answer Framework

প্রতিটি উত্তর:
1. **Direct answer:** ১-২ বাক্যে concept।
2. **Mechanism:** কীভাবে কাজ করে।
3. **Example:** বাস্তব use case বা ছোট code/query।
4. **Trade-off:** কখন ভালো, limitation কী।
5. **Failure/test:** কী ভুল হতে পারে, কীভাবে verify করবে।

কোনো উত্তর না জানলে bluff না করে assumption পরিষ্কার করে reasoning দেখাও।

## Self-scoring Rubric

প্রতিটি উত্তর ০-৪ scale-এ score করো:

| Score | Meaning |
|---:|---|
| 0 | উত্তর দিতে পারিনি / ভুল |
| 1 | Definition বলতে পারি, কিন্তু mechanism অস্পষ্ট |
| 2 | Mechanism ও example আছে, trade-off দুর্বল |
| 3 | Correct explanation, example ও limitation |
| 4 | Clear reasoning, trade-off, failure mode ও test strategy |

প্রতিটি round শেষে:
- Strongest 3 topics।
- Weakest 3 topics।
- ভুল উত্তরগুলোর correction।
- পরবর্তী practice task।

## Final Mock Interview Rule

একটি উত্তর ৬০-৯০ সেকেন্ডে explain করার চেষ্টা করো। System design-এ আগে clarification ও assumption, তারপর architecture আঁকো। নিজের project-এ implement না করা feature production experience হিসেবে দাবি কোরো না।
