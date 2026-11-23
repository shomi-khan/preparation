# Module 3: Giving an Agent Something to Stand On

একটি LLM অনেক কিছু জানে, কিন্তু কোনো প্রতিষ্ঠানের private document, সদ্য পরিবর্তিত policy বা customer-specific record তার training-এর মধ্যে থাকার নিশ্চয়তা নেই। Agent-কে বাস্তব কাজে উপযোগী করতে হলে সঠিক সময়ে সঠিক উৎস থেকে প্রাসঙ্গিক evidence এনে তার ভিত্তিতে answer তৈরি করতে হয়।

এই module-এ আমরা Grounding থেকে শুরু করে RAG-এর সম্পূর্ণ lifecycle, retrieval quality, Knowledge Graph, GraphRAG এবং production operations পর্যন্ত যাব।

## Notes

1. [Why an Agent Needs Grounded Knowledge](01-why-agents-need-grounded-knowledge.md)
2. [RAG: From Documents to Answers](02-rag-from-documents-to-answers.md)
3. [Retrieval Quality: Finding the Right Evidence](03-retrieval-quality.md)
4. [Knowledge Graph: Knowledge as Relationships](04-knowledge-graph.md)
5. [GraphRAG: Combining Graphs and Retrieval](05-graphrag.md)
6. [Production RAG: Evaluation, Security and Operations](06-production-rag.md)
7. [Recap ও Interview Drill](07-recap-interview.md)

## কীভাবে পড়বে

প্রথমে parametric ও external knowledge এবং grounding-এর প্রয়োজন বোঝো। এরপর RAG-এর ingestion ও query-time lifecycle আলাদা করে trace করো। Retrieval chapter-এ precision/recall, dense/keyword/hybrid search ও reranking-এর trade-off নিজে উদাহরণ দিয়ে যাচাই করো। Knowledge Graph ও GraphRAG পড়ার সময় entity, relation, provenance ও temporal validity খেয়াল করো। শেষ chapter-এ security, freshness, evaluation ও failure recovery-কে production release criteria হিসেবে দেখো।

## শেখার ফল

- Parametric ও external knowledge-এর পার্থক্য এবং grounding-এর প্রয়োজন ব্যাখ্যা।
- RAG-এর ingestion ও query-time lifecycle design।
- Parsing, chunking, embedding, indexing ও retrieval-এর failure শনাক্ত।
- Dense, keyword, hybrid retrieval এবং reranking evaluate।
- Knowledge Graph-এর entity, relationship, schema, provenance ও traversal বোঝা।
- GraphRAG-এর local/global retrieval pattern ও trade-off ব্যাখ্যা।
- Production RAG-এর ACL, freshness, deletion, caching, observability, evaluation, latency ও cost design।

## Source fidelity

এই module-এর topic map “Giving an Agent Something to Stand On”: RAG, Knowledge Graph, GraphRAG এবং company knowledge। এটি মূল course-এর 00:49:07–01:17:07 অংশের জন্য সাজানো। পূর্ণ video transcript/audio ও প্রতিটি on-screen demo line-by-line verify করা হয়নি। তাই এগুলো topic-aligned বিস্তারিত বাংলা learning notes; transcript-verified exhaustive transcription নয়।