# Module 1: From Answering to Doing

এই module-এ আমরা LLM-এর text generation capability থেকে শুরু করে goal-directed Agent system-এর architecture বুঝব। লক্ষ্য শুধু Agent-এর সংজ্ঞা জানা নয়; একটি agent কীভাবে observe, decide, act ও feedback ব্যবহার করে এবং production-এ তার state, memory, permission ও context কীভাবে manage করতে হয়, তা বোঝা।

## Lessons

1. [What Makes Something an Agent?](01-what-makes-an-agent.md)
2. [Agent-এর Core Subsystems](02-agent-core-subsystems.md)
3. [Single Agent বনাম Multi-Agent](03-single-vs-multi-agent.md)
4. [Agent Memory: কেন Agent ভুলে যায়?](04-agent-memory.md)
5. [Context Window as a Budget ও Lost in the Middle](05-context-window-and-lost-in-the-middle.md)
6. [Module Recap ও Interview Drill](06-recap-interview.md)

## কীভাবে পড়বে

প্রতিটি chapter-এ প্রথমে বাস্তব সমস্যা ও conceptual explanation পড়ো। এরপর diagram/code/example দেখে mechanism বোঝো। Failure ও trade-off section-এ থেমে নিজে ভাবো design ভাঙলে কী হবে। Interview section মুখস্থ করার বদলে নিজের ভাষায় reasoning দিয়ে উত্তর দাও। শেষের practice exercise নিজে design করে দেখো।

## শেখার ফলাফল

এই module শেষে তুমি:
- LLM, chatbot, tool-using assistant এবং agentic system-এর পার্থক্য ব্যাখ্যা করতে পারবে।
- Agent loop ও perception, reasoning, action, learning, communication subsystem-এর responsibility বুঝবে।
- Single agent নাকি multi-agent দরকার, measurable trade-off দিয়ে সিদ্ধান্ত নিতে পারবে।
- Working, long-term, episodic ও semantic memory-এর ব্যবহার বুঝবে।
- Context window, token budget, context overflow ও Lost in the Middle আলাদা করতে পারবে।
- একটি bounded, secure এবং observable refund agent-এর design ব্যাখ্যা করতে পারবে।

## Source note

ভিডিওর chapter metadata অনুযায়ী Module 1-এর বিষয়গুলো সাজানো। পূর্ণ transcript/audio ও প্রতিটি on-screen demo independently verify করা হয়নি। তাই এগুলো topic-aligned বিস্তারিত learning notes; ভিডিওর প্রতিটি বক্তব্যের exhaustive বা verbatim record নয়।