# 01. Why an Agent Needs Grounded Knowledge

## LLM অনেক কিছু জানে, কিন্তু সবকিছু জানে না

একটি Large Language Model (LLM) training-এর সময় বিপুল text থেকে language pattern, concept ও relationship শেখে। এই শেখা জ্ঞানকে **parametric knowledge** বলা হয়, কারণ তা model-এর parameters-এর মধ্যে encode থাকে।

কিন্তু বাস্তব application-এ প্রশ্ন আসে:
- আমাদের কোম্পানির সর্বশেষ Refund Policy কী?
- গতকাল কোন customer-এর subscription renew হয়েছে?
- Internal Engineering Handbook-এ deployment freeze সম্পর্কে কী বলা আছে?
- এই quarter-এ কোন product-এর দাম পরিবর্তন হয়েছে?

এই তথ্য training data-তে নাও থাকতে পারে। থাকলেও পুরোনো, অসম্পূর্ণ বা নির্দিষ্ট user-এর জন্য প্রযোজ্য নাও হতে পারে। LLM-এর fluency-কে factual authority ভাবা বিপজ্জনক। সুন্দরভাবে বলা উত্তরও ভুল হতে পারে।

## Parametric বনাম external knowledge

| বিষয় | Parametric knowledge | External knowledge |
|---|---|---|
| কোথায় থাকে | Model parameters | Document, DB, API, index, graph |
| Update | Model update/fine-tuning | Source update ও index refresh |
| Private company data | সাধারণত উপযুক্ত নয় | ACL মেনে দেওয়া যায় |
| Provenance | নির্দিষ্ট source দেখানো কঠিন | Source reference রাখা যায় |
| ঝুঁকি | Stale/ভুল learned association | Retrieval miss, stale index, source error |

External knowledge শুধু PDF নয়। Policy, catalogue, database record, ticket, source code, wiki, API response বা Knowledge Graph-ও হতে পারে।

## Grounding কী?

**Grounding** হলো model-এর উত্তরকে নির্দিষ্ট, যাচাইযোগ্য evidence-এর সঙ্গে যুক্ত করা।

Ungrounded flow-তে model নিজের learned knowledge থেকে উত্তর বানায়। সেটি industry practice-এর মতো শোনালেও তোমার company policy-র সঙ্গে না-ও মিলতে পারে।

Grounded flow:
1. প্রশ্ন বোঝা।
2. Approved source থেকে প্রাসঙ্গিক অংশ খোঁজা।
3. Evidence model-এর context-এ দেওয়া।
4. Evidence-এর ভিত্তিতে উত্তর তৈরি।
5. সম্ভব হলে source reference দেখানো।

Model-এর কাজ evidence থেকে উত্তর তৈরি করা; policy উদ্ভাবন করা নয়।

## RAG কী?

RAG-এর পূর্ণরূপ **Retrieval-Augmented Generation**।
- Retrieval: external information খুঁজে আনা।
- Augmented: পাওয়া information model context-এ যোগ করা।
- Generation: model সেই context ব্যবহার করে উত্তর তৈরি করা।

~~~mermaid
flowchart LR
  Q[User Question] --> R[Retriever]
  D[(Knowledge Sources)] --> R
  R --> E[Relevant Evidence]
  E --> P[Prompt / Context]
  Q --> P
  P --> L[LLM]
  L --> A[Grounded Answer]
~~~

RAG model-এর weights পরিবর্তন করে না। প্রতিটি প্রশ্নের সময় relevant information retrieve করে context-এ যোগ করে।

## RAG বনাম Fine-tuning

Fine-tuning model-এর parameters/behavior পরিবর্তন করে; RAG inference-এর সময় external evidence এনে context দেয়।

| প্রয়োজন | সম্ভাব্য পছন্দ |
|---|---|
| পরিবর্তনশীল company policy | RAG |
| Private documents থেকে উত্তর | RAG |
| Source citation দরকার | RAG |
| নির্দিষ্ট response style | Prompt বা fine-tuning |
| Domain-specific task behavior | Fine-tuning বিবেচনা |
| নতুন factual data নিয়মিত যোগ | সাধারণত RAG |
| Tool permission enforce | Runtime/server-side policy |

দুটো একই system-এ একসঙ্গে থাকতে পারে।

## RAG-এর দুইটি lifecycle

~~~mermaid
flowchart TD
  subgraph ING["Indexing / Ingestion"]
    S[Documents] --> P[Parse and Clean]
    P --> C[Chunk]
    C --> M[Metadata]
    M --> E[Embeddings]
    E --> I[(Search Index)]
  end
  subgraph QUERY["Query time"]
    Q[Question] --> N[Query Processing]
    N --> R[Retrieve]
    I --> R
    R --> F[Filter / Rerank]
    F --> X[Build Context]
    X --> G[Generate Answer]
  end
~~~

Indexing pipeline data-কে search-ready করে। Query pipeline প্রশ্নের জন্য evidence খুঁজে আনে। Indexing খারাপ হলে retrieval দুর্বল; index ভালো হলেও query ভুল হলে evidence miss হতে পারে।

## কখন RAG প্রয়োজন নেই?

- সাধারণ ভাষাগত প্রশ্নে model-এর built-in knowledge যথেষ্ট।
- Exact current state-এর জন্য authoritative API/database query ভালো।
- হিসাবের জন্য calculator বা deterministic code ভালো।
- Action সম্পন্ন করতে tool calling দরকার।
- ছোট ও স্থির context সরাসরি prompt-এ দেওয়া যায়।

RAG knowledge retrieval-এর সমস্যা সমাধান করে; transaction, authorization বা deterministic calculation-এর বিকল্প নয়।

## Failure modes

| Failure | কেন হয় | কী করা যায় |
|---|---|---|
| Hallucination | Evidence নেই/অপর্যাপ্ত | Evidence check, abstain |
| Stale answer | Index পুরোনো | Refresh, freshness metadata |
| Wrong document | Retrieval ভুল | Chunking, hybrid search, rerank |
| Access leak | অন্য user-এর document retrieve | Pre-retrieval ACL filter |
| Unsupported citation | Source claim support করে না | Claim-source validation |
| Conflicting policy | একাধিক version | Authority ও effective date |

## Interview drill

**RAG কী এবং কেন দরকার?**  
RAG inference-time-এ external source থেকে evidence retrieve করে model context-এ যোগ করে। Private, changing বা domain-specific knowledge এবং source-backed answer-এর জন্য উপযোগী।

**RAG কি hallucination পুরোপুরি বন্ধ করে?**  
না। Retriever ভুল evidence আনতে পারে, source ভুল হতে পারে, evidence অসম্পূর্ণ হতে পারে, model ভুল বুঝতে পারে। Retrieval evaluation, grounded prompt, citation validation ও abstention দরকার।

**RAG আর fine-tuning-এর পার্থক্য কী?**  
RAG runtime-এ external evidence যোগ করে; fine-tuning model behavior/parameters update করে। Changing factual knowledge ও citation-এর জন্য RAG; consistent task behavior/style-এর জন্য fine-tuning কাজে আসতে পারে।

**RAG কি database query-এর বিকল্প?**  
সবসময় নয়। Exact current state বা transactional data-এর জন্য structured API/SQL ভালো। RAG unstructured content-এর semantic retrieval-এ কার্যকর।

## নিজে যাচাই করো

একটি HR assistant-এর কাছে employee handbook, leave policy ও employee-specific leave balance আছে।
1. কোন তথ্য RAG দিয়ে retrieve করবে?
2. কোন তথ্য authoritative database/API থেকে আনবে?
3. অন্য employee-এর balance জানতে চাইলে কোথায় reject করবে?
4. পুরোনো handbook version index-এ থাকলে কীভাবে detect করবে?

মূল শিক্ষা: **Model-এর উত্তর যত গুরুত্বপূর্ণ, উত্তর তৈরির আগে তাকে কোন evidence দেওয়া হলো সেটিও তত গুরুত্বপূর্ণ।**