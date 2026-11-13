# AI/ML Engineer Preparation

এই repository-তে AI/ML Engineer role-এর জন্য structured, hands-on preparation material রাখা হবে। প্রতিটি দিনের জন্য আলাদা directory থাকবে। সেখানে topic-wise notes, প্রশ্নোত্তর, code example, practical exercise এবং interview preparation যোগ করা হবে।

## Study Roadmap

| Day | মূল বিষয় | কী কী কভার করা হবে |
|---|---|---|
| [Day 1](day1/README.md) | Python for AI Engineering | Python fundamentals, data structures, iterators ও generators, decorators, context managers, exception handling, type hints, asyncio, Pydantic, NumPy, Pandas |
| Day 2 | Machine Learning Fundamentals | Supervised ও unsupervised learning, classification, regression, clustering, feature engineering, train/test split, Scikit-learn, Linear ও Logistic Regression, Decision Tree, Random Forest, KNN, K-Means, XGBoost-এর পরিচিতি |
| Day 3 | ML Evaluation, Data Analysis & SQL | Confusion matrix, accuracy, precision, recall, F1-score, ROC-AUC, cross-validation, overfitting ও underfitting, bias-variance, data leakage, imbalanced dataset, SQL JOIN, GROUP BY, CTE, subquery, window functions, query optimization |
| Day 4 | LLM Fundamentals & Prompt Engineering | Tokenization, tokens, embeddings, Transformer, attention ও self-attention, context window, next-token prediction, temperature, top-p, prompt design, zero-shot ও few-shot prompting, structured output, function calling, hallucination, prompt injection |
| Day 5 | Retrieval-Augmented Generation (RAG) | RAG architecture, document loading, text extraction, chunking, embeddings, vector database, semantic search, cosine similarity, hybrid search, reranking, context construction, source citation, retrieval ও answer evaluation |
| Day 6 | AI Agents & Production AI Engineering | LLM application বনাম agent, tool calling, agentic workflow, planning ও execution, state, memory, human-in-the-loop, LangChain ও LangGraph-এর ধারণা, database query agent, API integration, timeout, retry, rate limit, logging, observability, caching, latency ও token cost, Docker |
| Day 7 | Revision, System Design & Mock Interview | সব দিনের গুরুত্বপূর্ণ বিষয় revision, Enterprise Document Intelligence System design, RAG system design discussion, Python ও SQL coding practice, ML/LLM/Agent interview questions, project presentation এবং mock interview |

## প্রতিদিনের শেখার পদ্ধতি

প্রতিটি দিনের material-এ যতটা সম্ভব নিচের বিষয়গুলো রাখা হবে:

1. **Concepts:** বিষয়গুলোর সহজ বাংলা ব্যাখ্যা; technical terms ইংরেজিতে।
2. **Questions & Answers:** মৌলিক ধারণা থেকে interview-level প্রশ্নোত্তর।
3. **Code Examples:** প্রয়োজন অনুযায়ী runnable Python বা SQL example।
4. **Practical Exercises:** নিজে code লিখে অনুশীলনের task।
5. **Interview Preparation:** বাস্তব interview-এ আসতে পারে এমন conceptual ও scenario-based প্রশ্ন।
6. **Review Checklist:** দিনের শেষে কোন বিষয়গুলো বুঝেছি এবং কোনগুলো আবার দেখতে হবে তা যাচাই।

## Preparation Priority

সময় সীমিত হলে নিচের ক্রমে গুরুত্ব দেওয়া হবে:

- **Priority 1:** Python, ML fundamentals, LLM application development এবং RAG।
- **Priority 2:** SQL, AI Agents এবং production engineering।
- **Priority 3:** Deep Learning ও fine-tuning-এর foundational ধারণা।

লক্ষ্য শুধু definition মুখস্থ করা নয়। প্রতিটি গুরুত্বপূর্ণ concept-এর ক্ষেত্রে কীভাবে কাজ করে, কখন ব্যবহার করতে হয়, কী trade-off আছে এবং production system-এ কী সমস্যা হতে পারে, সেগুলো বোঝা।

## Directory Structure

```text
preparation/
├── README.md
├── day1/
│   ├── README.md
│   ├── 01-python-fundamentals.md
│   ├── 02-data-structures.md
│   ├── 03-iterators-generators.md
│   ├── 04-decorators-context-managers.md
│   ├── 05-exceptions-type-hints.md
│   ├── 06-asyncio-concurrency.md
│   ├── 07-pydantic.md
│   ├── 08-numpy.md
│   ├── 09-pandas.md
│   ├── 10-practical-exercises.md
│   └── 11-interview-questions.md
├── day2/
├── day3/
├── day4/
├── day5/
├── day6/
└── day7/
```

বর্তমানে Day 1-এর material তৈরি করা হয়েছে। পরবর্তী দিনগুলোর directory ও content roadmap অনুযায়ী ধাপে ধাপে যোগ করা হবে।
