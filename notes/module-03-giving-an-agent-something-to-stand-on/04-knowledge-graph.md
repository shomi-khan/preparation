# 04. Knowledge Graph: Knowledge as Relationships

Document retrieval অনেক প্রশ্নের উত্তর দিতে পারে। কিন্তু কিছু প্রশ্নে শুধু relevant paragraph খুঁজলেই হয় না; একাধিক entity এবং তাদের relationship বুঝতে হয়।

উদাহরণ: কোন team কোন service maintain করে? Service A কোন database-এর ওপর নির্ভরশীল? Incident কোন deployment-এর পরে ঘটেছিল? কোন policy কোন region-এ প্রযোজ্য?

## Graph-এর মৌলিক ধারণা

- **Node / Entity:** ব্যক্তি, প্রতিষ্ঠান, service, document, product, incident।
- **Edge / Relationship:** owns, depends_on, works_for, caused_by, applies_to।
- **Property:** node বা edge-এর attribute, যেমন name, date, status, confidence, source।

~~~mermaid
flowchart LR
  A[Team: Payments] -->|maintains| B[Service: Checkout]
  B -->|depends_on| C[Database: Orders DB]
  D[Incident: INC-42] -->|affected| B
  D -->|occurred_after| E[Deployment: DEP-91]
~~~

## Triple representation

Knowledge Graph-এর একটি সাধারণ fact হলো subject-predicate-object triple:

~~~text
(Payments Team, MAINTAINS, Checkout Service)
(Checkout Service, DEPENDS_ON, Orders DB)
(INC-42, AFFECTED, Checkout Service)
~~~

Relation-এর direction গুরুত্বপূর্ণ। Schema-তে consistent convention দরকার।

## Graph বনাম relational database

| বিষয় | Relational DB | Graph DB |
|---|---|---|
| Data model | Table, row, foreign key | Node, edge, property |
| Relationship query | JOIN | Traversal / pattern match |
| Strength | Transaction, structured records | Relationship-heavy traversal |
| Trade-off | Complex joins বড় হতে পারে | Schema/governance/query planning দরকার |

Graph database সবসময় SQL database-এর বিকল্প নয়। Transactional record relational DB-তে এবং relationship graph-এ রাখা যেতে পারে।

## Knowledge Graph তৈরির pipeline

~~~mermaid
flowchart TD
  S[Documents / APIs / DB] --> E[Entity Extraction]
  E --> R[Relationship Extraction]
  R --> N[Normalization]
  N --> L[Entity Resolution]
  L --> V[Validation and Provenance]
  V --> G[(Knowledge Graph)]
~~~

**Entity extraction:** Text থেকে “Payments team”, “Checkout service”, “Orders database”, “INC-42” বের করা। Rule, parser, NER model বা LLM ব্যবহার করা যায়।

**Relationship extraction:** Entity-গুলোর মধ্যে relation বের করা। LLM সাহায্য করতে পারে, কিন্তু ভুল relation-ও তৈরি করতে পারে। Schema ও validation দরকার।

**Normalization:** “Checkout”, “checkout-service” এবং “Checkout API” একই entity বোঝাতে পারে। Alias mapping ও canonical ID দরকার।

**Entity resolution:** দুইটি record একই real-world entity কি না নির্ধারণ। একই নামের দুই employee একই ব্যক্তি নয়। Stable ID, domain, source, temporal information, confidence এবং human review কাজে লাগে।

**Provenance:** প্রতিটি fact-এর সঙ্গে source document/record ID, page/section, extraction time, source version, confidence, valid time/observed time রাখা উচিত। Provenance ছাড়া ভুল fact কোথা থেকে এসেছে খুঁজে পাওয়া কঠিন।

## Schema / Ontology

Ontology বা schema বলে কী ধরনের node থাকবে, কী relation allowed, relation-এর domain/range কী, কোন property required এবং temporal rule কী।

~~~text
Node types:
  Team, Service, Database, Incident

Allowed relationships:
  Team -[MAINTAINS]-> Service
  Service -[DEPENDS_ON]-> Database
  Incident -[AFFECTED]-> Service
~~~

Schema খুব rigid হলে নতুন concept যোগ কঠিন; খুব loose হলে inconsistent data তৈরি হয়।

## Graph query ও traversal

প্রশ্ন: “Payments team-এর maintained service কোন database-এর ওপর নির্ভরশীল?”

1. Payments Team node খুঁজো।
2. MAINTAINS edge ধরে Service-এ যাও।
3. DEPENDS_ON edge ধরে Database-এ যাও।
4. Path ও source evidence ফেরত দাও।

~~~text
(Payments)-[:MAINTAINS]->(Service)-[:DEPENDS_ON]->(Database)
~~~

এটি conceptual pattern; নির্দিষ্ট query language নয়।

## Knowledge Graph এবং RAG

Document RAG relevant text chunk retrieve করে। Knowledge Graph structured entity ও relationship query করে।

Hybrid system-এ graph থেকে relationship/path, document index থেকে supporting paragraph retrieve করে দুটো evidence একত্রে LLM-কে দেওয়া যায়। Graph fact-ও source-backed, versioned ও access-controlled হওয়া উচিত।

## কখন Knowledge Graph ব্যবহার করবে?

উপযোগী: entity relationship গুরুত্বপূর্ণ, multi-hop question, impact analysis, বহু source-এ ছড়ানো data, entity identity/lineage track করা।

কম উপযোগী: সাধারণ FAQ-তে document retrieval যথেষ্ট; relationship কম এবং SQL-এ সহজ; graph maintenance cost বেশি; extraction quality খারাপ এবং review নেই।

## Failure modes

| Failure | প্রভাব | প্রতিকার |
|---|---|---|
| Entity duplication | একই entity আলাদা node | Entity resolution |
| False edge | ভুল relationship | Schema, evidence, review |
| Stale edge | পুরোনো ownership | Validity, sync, expiration |
| Missing provenance | Fact যাচাই অসম্ভব | Source reference mandatory |
| Broad traversal | Huge/irrelevant result | Depth/relationship constraints |
| Access leak | Restricted relation প্রকাশ | Node/edge authorization |
| Ontology drift | Inconsistent graph | Versioned schema |

## Interview drill

**Knowledge Graph কী?**  
Entity ও relationship-কে node-edge-property structure-এ প্রকাশ করা knowledge representation। Relationship-centric query ও multi-hop traversal-এ কাজে লাগে।

**Graph DB কেন relational DB-এর চেয়ে ভালো?**  
সব ক্ষেত্রে ভালো নয়। Deep/variable traversal-এ graph natural; transactional tabular workload-এ relational DB ভালো fit হতে পারে।

**Entity resolution কেন কঠিন?**  
একই নামের আলাদা entity থাকতে পারে; একই entity-র alias থাকতে পারে। Stable ID, domain constraint, matching confidence ও review দরকার।

**Graph fact বিশ্বাস করবে কীভাবে?**  
Provenance, source version, extraction confidence, valid time, schema validation ও source-of-truth policy দরকার। LLM extracted relation authoritative নয়।

## অনুশীলন

একটি software company architecture graph বানাও: Team, Service, Database, API, Incident, Deployment। কমপক্ষে ৫টি relationship define করো। তারপর service down হলে কোন team notify হবে, কোন DB affected, incident কোন deployment-এর পরে শুরু এবং ownership পরিবর্তন কীভাবে represent করবে তা ব্যাখ্যা করো।