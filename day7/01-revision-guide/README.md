# Final Revision Guide

এই guide শেষ মুহূর্তে সব chapter আবার পড়ার জন্য নয়। বরং প্রতিটি domain-এর mental model, practical skill এবং interview-এ explain করার মতো trade-off মনে আছে কি না যাচাই করার জন্য।

প্রতিটি topic-এ নিজেকে তিনটি প্রশ্ন করো:
1. এটি কী এবং কীভাবে কাজ করে?
2. কখন ব্যবহার করব, কখন করব না?
3. Failure mode, trade-off এবং test কী?

## ১. Python Fundamentals

### Must know
- Dynamic typing, object reference, identity বনাম equality।
- Mutable/immutable এবং shallow/deep copy।
- List, tuple, set, dict, deque, heap।
- Hashability ও dictionary collision।
- Iterable, iterator, generator, yield, yield from।
- Decorator, wraps, context manager, ExitStack।
- Exception hierarchy, propagation, chaining।
- Type hints, Any বনাম object, Protocol, Generic।
- Asyncio, event loop, Task, TaskGroup, cancellation।
- Threading, multiprocessing, GIL caveats।

### Practical prompts
- ১০ GB file কীভাবে stream করে process করবে?
- Mutable default argument bug দেখাও এবং fix করো।
- Queue ও heap কখন বেছে নেবে?
- Context manager exception হলেও cleanup কীভাবে করে?
- Async network calls কীভাবে bounded করবে?
- Blocking I/O async service-এ কীভাবে handle করবে?

~~~mermaid
flowchart TD
    A["Python runtime"] --> B["Object model and mutability"]
    B --> C["Collections and complexity"]
    C --> D["Iteration and generators"]
    D --> E["Decorators / resource lifecycle"]
    E --> F["Exceptions and typing"]
    F --> G["Concurrency and async"]
~~~

## ২. NumPy এবং Pandas

### NumPy
- ndarray shape, ndim, size, dtype।
- Vectorization ও broadcasting।
- axis=0/1 reasoning।
- Basic slicing view বনাম advanced indexing copy।
- reshape, ravel, flatten, transpose।
- Element-wise multiplication বনাম matrix multiplication।
- NaN, dtype precision, random Generator।

### Pandas
- Series/DataFrame।
- loc বনাম iloc।
- Missing value, duplicate ও type conversion।
- GroupBy split-apply-combine।
- merge বনাম concat; join cardinality।
- pivot বনাম pivot_table।
- Chained assignment এবং explicit loc।
- Chunked processing ও memory strategy।

### Practical prompts
- Row-wise apply-এর বদলে vectorized operation কেন?
- Merge-এর পরে row count unexpectedly বেড়ে গেলে কী inspect করবে?
- ১০ GB CSV কীভাবে process করবে?
- Train/test leakage এড়াতে imputer/scaler কোথায় fit করবে?

## ৩. Machine Learning

### Core concepts
- Supervised, unsupervised, semi/self-supervised, reinforcement learning।
- Classification বনাম regression।
- Feature/label, prediction time, generalization।
- Train/validation/test ও cross-validation।
- Random, stratified, group ও time-based split।
- Overfitting, underfitting, bias/variance।
- Feature engineering ও data leakage।
- Baseline, loss বনাম metric, threshold ও calibration।

### Algorithms
- Linear/Logistic Regression।
- Decision Tree, Random Forest।
- KNN, SVM, Naive Bayes।
- K-Means।
- Gradient Boosting, bagging বনাম boosting।
- Scaling ও regularization।

### Evaluation
- Confusion matrix, accuracy, precision, recall, specificity, F1।
- ROC-AUC, PR-AUC/average precision।
- MAE, MSE, RMSE, R²।
- Calibration, metric uncertainty।

~~~mermaid
flowchart TD
    A["Problem and prediction time"] --> B["Data audit / leakage check"]
    B --> C["Appropriate split"]
    C --> D["Baseline"]
    D --> E["Pipeline and model"]
    E --> F["Cross-validation / tuning"]
    F --> G["Validation threshold / calibration"]
    G --> H["Final test"]
~~~

### Practical prompts
- ১% fraud data-তে accuracy কেন যথেষ্ট নয়?
- একই customer-এর record train ও test-এ গেলে কী সমস্যা?
- Model probability 0.8 দিলে কি 80% certainty?
- Production score কমে গেলে কী investigate করবে?

## ৪. SQL

- SELECT, WHERE, GROUP BY, HAVING, ORDER BY।
- INNER/LEFT/RIGHT/FULL JOIN।
- NULL semantics, COUNT(*) বনাম COUNT(column)।
- CTE, subquery, EXISTS।
- Window functions: ROW_NUMBER, RANK, DENSE_RANK, LAG।
- Index ও composite index।
- EXPLAIN / EXPLAIN ANALYZE।
- Transactions, ACID ও isolation।
- Parameterized query ও SQL injection।
- Offset বনাম keyset pagination।

### Practical prompts
- প্রতি customer-এর latest transaction।
- প্রতি month-এর top 3 customer।
- যেসব customer-এর transaction নেই।
- Join-এর পরে duplicate row explosion diagnose।
- Query plan দেখে slow query optimize।

## ৫. LLM Fundamentals এবং Prompt Engineering

- Tokenization, token budget, embeddings।
- Transformer, self-attention, Q/K/V, causal mask, positional information।
- Encoder/decoder architecture।
- Pretraining, SFT, preference alignment।
- Context window, temperature/top-p।
- Hallucination, grounding, abstention।
- KV cache, quantization।
- Zero-shot/few-shot, structured output।
- Prompt injection, tool calling ও evaluation।

### Practical prompts
- Valid JSON কিন্তু semantically wrong output কীভাবে reject করবে?
- Prompt injection-এ retrieved document malicious instruction দিলে কী হবে?
- Prompting, RAG ও fine-tuning কখন বেছে নেবে?
- Context budget কীভাবে ভাগ করবে?

## ৬. RAG

- Ingestion বনাম query pipeline।
- Parsing/OCR, chunking, metadata ও versioning।
- Embedding compatibility।
- Vector search, cosine/dot/Euclidean।
- ANN, hybrid search/BM25, RRF।
- Reranking, MMR, parent-child retrieval।
- ACL, tenant isolation, citation।
- Recall@k, Precision@k, MRR, nDCG।
- Faithfulness বনাম correctness।
- No-answer behavior, freshness ও re-indexing।

### Practical prompts
- Relevant chunk retrieve না হলে কোন stage debug করবে?
- Correct answer কিন্তু wrong citation হলে কী fix করবে?
- Tenant A-এর document Tenant B-কে কীভাবে block করবে?
- Document delete হলে vector/cache কীভাবে invalidate করবে?

## ৭. Agents এবং Production AI

- Workflow বনাম agent।
- Observe → decide → act → observe loop।
- Tool schema, registry, authorization।
- Least privilege, idempotency, human approval।
- State machine, checkpoint, memory।
- Max steps/time/token budget।
- Retry/backoff, circuit breaker, rate limit/backpressure।
- Logs, metrics, traces।
- Cost/latency monitoring।
- Docker, secret management, canary/rollback।

### Practical prompts
- Agent-কে unrestricted SQL access দেওয়া কেন বিপজ্জনক?
- Approval stale হলে কীভাবে handle করবে?
- Agent একই tool বারবার call করলে কী করবে?
- Provider outage-এ graceful degradation কীভাবে করবে?

## ৮. Interview Answer Framework

প্রতিটি technical question-এর উত্তর এই কাঠামোয় practice করো:

1. **Direct answer:** ১-২ বাক্যে concept।
2. **Mechanism:** কীভাবে কাজ করে।
3. **Example:** বাস্তব use case বা ছোট code/query।
4. **Trade-off:** কখন ভালো, limitation কী।
5. **Failure/test:** কী ভুল হতে পারে, কীভাবে verify করবে।

Definition মুখস্থ করলেই হবে না। নিজের reasoning ও trade-off explain করতে হবে।

## ৯. Final Self-Test

প্রতিটি domain থেকে:
- ৫টি question না দেখে explain করো।
- ২টি ছোট code/query নিজে লেখো।
- ১টি failure scenario debug করো।
- ১টি design decision justify করো।

যে topic explain করতে পারছ না, শুধু সেটিই targeted revision করো। সবকিছু আবার শুরু থেকে পড়ার দরকার নেই।
