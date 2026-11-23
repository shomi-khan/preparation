# 05. GraphRAG: Combining Graphs and Retrieval

RAG document-এর relevant chunk খুঁজে আনে। Knowledge Graph entity ও relationship ধরে রাখে। **GraphRAG** এই দুই ধরনের representation একসঙ্গে ব্যবহার করে।

GraphRAG-এর একটিমাত্র universal implementation নেই। বিভিন্ন system graph construction, community detection, retrieval ও summarization ভিন্নভাবে করতে পারে।

## সাধারণ RAG কোথায় আটকে যেতে পারে?

প্রশ্ন: “গত দুই quarter-এ Checkout Service-এর reliability কমার প্রধান কারণ কী এবং কোন team-গুলো জড়িত?”

উত্তর পেতে Incident report, Deployment history, Service ownership, Dependency graph, Team responsibility ও Quarter-wise trend লাগতে পারে। একটি semantic query কয়েকটি paragraph খুঁজে দিতে পারে, কিন্তু cross-document relationship জোড়া লাগানো কঠিন হতে পারে।

## GraphRAG mental model

~~~mermaid
flowchart TD
  D[Documents / Structured Sources] --> P[Parse and Chunk]
  P --> E[Entity and Relation Extraction]
  E --> G[(Knowledge Graph)]
  P --> V[(Vector / Text Index)]
  G --> Q[Graph-aware Retrieval]
  V --> Q
  U[User Query] --> Q
  Q --> X[Evidence + Paths + Summaries]
  X --> L[LLM]
  L --> A[Grounded Answer]
~~~

Graph এবং text index দুটোই evidence source। Query অনুযায়ী graph traversal, vector search, keyword search বা combination হতে পারে।

## GraphRAG indexing pipeline

1. Documents ingest ও normalize।
2. Text chunk তৈরি।
3. Entity extract।
4. Entity mention resolve।
5. Relationship extract।
6. Knowledge Graph build।
7. প্রয়োজন হলে graph community/cluster তৈরি।
8. Community বা graph-derived summary তৈরি।
9. Original chunk ও graph fact-এর provenance সংরক্ষণ।
10. Graph ও text index publish।

Graph construction ভুল হলে GraphRAG ভুল relationship-এর ওপর reasoning করবে। Extraction validation জরুরি।

## দুইটি broad retrieval pattern

**Local / Entity-centric retrieval:** প্রশ্নে নির্দিষ্ট entity আছে। Entity resolve, আশেপাশের relation traverse, supporting document chunk retrieve, তারপর evidence model-কে দেওয়া।

**Global / Corpus-level retrieval:** প্রশ্ন পুরো corpus-এর pattern বা summary চায়। যেমন “সব incident মিলিয়ে recurring reliability issue কী?” Community/cluster বা precomputed summary কাজে লাগতে পারে। গুরুত্বপূর্ণ claim original document evidence দিয়ে verify করতে হবে।

এই local/global terminology কিছু GraphRAG architecture-এ ব্যবহৃত হয়; সব implementation-এ একই algorithm থাকবে না।

## Local retrieval sequence

~~~mermaid
sequenceDiagram
  participant U as User
  participant R as Retriever
  participant G as Knowledge Graph
  participant V as Vector Index
  participant L as LLM
  U->>R: Checkout-এর dependency কী?
  R->>G: Entity lookup + traversal
  G-->>R: Orders DB relation + provenance
  R->>V: Supporting text search
  V-->>R: Architecture document chunks
  R->>L: Graph path + source evidence
  L-->>U: Answer with citations
~~~

Graph result relationship দেখায়; document result textual evidence দেয়। দুটো মিললে traceability বাড়তে পারে।

## GraphRAG বনাম Vector RAG

| বিষয় | Vector RAG | GraphRAG |
|---|---|---|
| Representation | Text chunk embeddings | Graph + text index |
| Retrieval | Semantic similarity | Traversal, semantic search, summary |
| Strength | Passage-level search | Relationship ও multi-hop context |
| Preparation | Parse/chunk/embed | সঙ্গে entity/relation extraction |
| Complexity | তুলনামূলক কম | বেশি |
| Failure | Wrong/missing chunk | ভুল entity/edge ও retrieval error |
| Cost | Embedding ও search | Graph build, extraction, refresh |

GraphRAG সবসময় Vector RAG-এর চেয়ে ভালো নয়। Simple FAQ-তে extra complexity অপ্রয়োজনীয় হতে পারে।

## GraphRAG বনাম Knowledge Graph

Knowledge Graph হলো knowledge represent করার structure। GraphRAG হলো graph-সহ external knowledge retrieve করে LLM generation augment করার approach। Knowledge Graph থাকলেই GraphRAG system হয় না; query-time retrieval, context assembly ও LLM generation থাকতে হয়।

## কখন GraphRAG বিবেচনা করবে?

প্রথমে document RAG baseline তৈরি করো। বাস্তব প্রশ্ন সংগ্রহ করে কোন প্রশ্নে relationship/multi-hop দরকার তা label করো। যদি service ownership, dependency, incident chain বা organization relationship-এ RAG ব্যর্থ হয়, তখন Knowledge Graph যোগ করে Graph-aware retrieval পরীক্ষা করো।

## কঠিন সমস্যা

- Entity resolution: “Payments”, “Payment Platform” ও “payments-service” একই entity কি না।
- Relationship correctness: A depends on B আর B depends on A উল্টে যাওয়া।
- Temporal validity: পুরোনো owner আজও owner নাও হতে পারে।
- Graph freshness: Document update হলেও graph update না হলে inconsistent answer।
- Provenance: Summary থেকে original supporting document-এ ফিরতে হবে।
- Access control: Traversal ও summary তৈরিতেও access policy মানতে হবে।

## Query strategy: illustrative pseudocode

~~~python
def answer_relationship_question(question, identity):
    entities = resolve_entities(question)
    graph_facts = graph.search(
        entities=entities,
        max_hops=2,
        identity=identity
    )
    text_evidence = document_search.search(
        query=question,
        identity=identity,
        top_k=10
    )
    evidence = merge_and_deduplicate(graph_facts, text_evidence)
    evidence = rerank_and_check_provenance(question, evidence)
    return llm.answer(
        question=question,
        evidence=evidence,
        abstain_if_insufficient=True
    )
~~~

এটি illustrative pseudocode। max_hops, identity-aware traversal ও provenance validation graph store/security model অনুযায়ী implement করতে হবে।

## Evaluation

- Entity extraction precision/recall
- Relation extraction precision/recall
- Entity resolution accuracy
- Path correctness
- Graph freshness
- Provenance coverage
- Multi-hop answer correctness
- Graph build cost/refresh latency
- Query latency ও cost

ভুল edge ঢুকিয়ে system সেটি confidently repeat করে কি না adversarial test করো।

## Interview drill

**GraphRAG কী?**  
Graph structure ও document retrieval একত্রে ব্যবহার করে LLM-কে relationship, multi-hop evidence বা corpus-level pattern-সহ context দেওয়ার approach family।

**কেন Vector RAG-এর চেয়ে ভালো হতে পারে?**  
Relationship ও multi-hop প্রশ্নে। কিন্তু graph construction/maintenance cost আছে; সব corpus-এ লাভ হয় না।

**Knowledge Graph থাকলেই GraphRAG?**  
না। Graph হলো representation; GraphRAG-এ query-time graph-aware retrieval ও LLM generation থাকে।

**সবচেয়ে বড় ঝুঁকি কী?**  
ভুল entity/edge, stale graph, provenance loss, access leak এবং unsupported summary। Extraction validation ও traceability দরকার।

## অনুশীলন

একটি incident investigation assistant design করো: incident report, service ownership, deployment record, dependency graph ও team roster।
1. “INC-42 কোন service-কে affect করেছে?” (local)
2. “গত ছয় মাসে কোন dependency pattern সবচেয়ে বেশি incident-এ জড়িত?” (global)

প্রতিটির retrieval strategy, evidence source, provenance ও evaluation metric লিখো।