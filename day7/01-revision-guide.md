# Revision Guide

## Python
- Mutable/immutable, identity/equality।
- List, tuple, set, dict।
- Iterable, iterator, generator।
- Decorator, context manager।
- Exceptions, type hints, Pydantic।
- Asyncio, event loop, concurrency।

নিজেকে জিজ্ঞেস করো: ১০ GB file কীভাবে stream করে process করবে? Async network calls কীভাবে bounded করবে?

## NumPy ও Pandas
- ndarray shape, dtype, vectorization, broadcasting, axis।
- DataFrame inspection, missing value, duplicate, groupby, merge।
- Large dataset memory strategy।

নিজেকে জিজ্ঞেস করো: row-wise apply-এর বদলে vectorized operation কেন?

## ML
- Supervised/unsupervised, classification/regression।
- Feature/label, split, leakage।
- Logistic Regression, Decision Tree, Random Forest, KNN, K-Means, boosting।
- Overfit/underfit, bias/variance, regularization।
- Precision, recall, F1, ROC-AUC, PR-AUC, CV।

নিজেকে জিজ্ঞেস করো: ১% fraud data-তে accuracy কেন যথেষ্ট নয়?

## SQL
- JOIN, GROUP BY, HAVING, CTE, subquery।
- Window function, NULL, index, EXPLAIN।
- Parameterized query ও SQL injection।

নিজেকে জিজ্ঞেস করো: প্রতিটি customer-এর latest transaction কীভাবে বের করবে?

## LLM
- Token, embedding, Transformer, attention, context window।
- Temperature/top-p, hallucination।
- Prompt engineering, structured output, function calling।
- Prompt injection ও tool security।

নিজেকে জিজ্ঞেস করো: valid JSON কিন্তু incorrect answer কীভাবে detect করবে?

## RAG
- Ingestion, parsing, chunking, embedding, vector search।
- Hybrid search, reranking, metadata/ACL।
- Retrieval vs generation evaluation।
- Citation, abstention, freshness, prompt injection।

নিজেকে জিজ্ঞেস করো: relevant chunk retrieve না হলে কোন stage debug করবে?

## Agents & Production
- Tool calling, workflow vs agent, state, memory।
- Step budget, timeout, authorization, human approval।
- Retry, rate limit, caching, observability, cost, Docker।

নিজেকে জিজ্ঞেস করো: agent-কে unrestricted SQL access দেওয়া কেন বিপজ্জনক?

## Revision technique
প্রতিটি topic-এ ৩ ধাপ:
1. ১ মিনিটে concept explain।
2. ৫ মিনিটে ছোট code/query লিখে দেখাও।
3. একটি limitation বা production failure scenario বলো।
