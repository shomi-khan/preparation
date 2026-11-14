# AI/ML Engineer Preparation

এই repository-তে AI/ML Engineer role-এর জন্য structured, hands-on preparation material রাখা হয়েছে।

## Study Roadmap

| Day | মূল বিষয় | বিস্তারিত |
|---|---|---|
| [Day 1](day1/README.md) | Python for AI Engineering | Python, CPython, object model, data structures, asyncio, Pydantic, NumPy, Pandas |
| [Day 2](day2/README.md) | Machine Learning Fundamentals | ML concepts, algorithms, Scikit-learn workflow |
| [Day 3](day3/README.md) | ML Evaluation, Data Analysis & SQL | Metrics, validation, data analysis, SQL |
| [Day 4](day4/README.md) | LLM Fundamentals & Prompt Engineering | Transformer, tokenization, prompts, structured output, LLM apps |
| [Day 5](day5/README.md) | Retrieval-Augmented Generation | RAG architecture, embeddings, retrieval, evaluation |
| [Day 6](day6/README.md) | AI Agents & Production AI | Tool calling, workflows, state, safety, production concerns |
| [Day 7](day7/README.md) | Revision, System Design & Mock Interview | Revision, system design, mock interview, project presentation |

## কীভাবে পড়বে

প্রতিটি chapter আলাদা folder-এ সাজানো:
- `README.md`: বাংলা explanation, technical terms ইংরেজিতে, diagrams ও code examples।
- `exercises.ipynb`: runnable code, guided practice এবং self-review।

Notebook Google Colab বা local Jupyter-এ চালানো যাবে। API key দরকার হলে environment variable/Colab Secrets ব্যবহার করবে; কখনো notebook-এ secret hardcode করবে না।

## Learning Strategy

1. README পড়ে concept বুঝবে।
2. Notebook-এর code cell নিজে run করবে।
3. Run করার আগে output predict করবে।
4. Exercise নিজে implement করবে।
5. Interview প্রশ্নের উত্তর নিজের ভাষায় বলবে।

লক্ষ্য definition মুখস্থ করা নয়। কীভাবে কাজ করে, কেন এমন design, কখন ব্যবহার করতে হয়, কী trade-off এবং failure mode আছে, তা বোঝা।

## Directory Structure

```text
preparation/
├── README.md
├── day1/
│   ├── README.md
│   ├── 01-python-fundamentals/
│   │   ├── README.md
│   │   └── exercises.ipynb
│   └── ...
├── day2/
├── day3/
├── day4/
├── day5/
├── day6/
└── day7/
```
