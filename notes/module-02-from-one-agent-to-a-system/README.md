# Module 2: From One Agent to a System

এই module-এ একটি agent কীভাবে বাইরের capability ব্যবহার করে এবং একাধিক component মিলে production-grade agent system তৈরি করে, তা শিখবে। Tool, Skill, MCP, Orchestration ও AI Gateway আলাদা problem solve করে। এগুলোর boundary পরিষ্কার না হলে system-এ permission, ownership, reliability ও debugging সমস্যা তৈরি হয়।

## Notes

1. [Tool Calling](01-tool-calling.md)
2. [MCP: Model Context Protocol](02-mcp.md)
3. [Agent Skills](03-agent-skills.md)
4. [Multi-Agent ও Orchestration](04-multi-agent-orchestration.md)
5. [AI Gateway](05-ai-gateway.md)
6. [System Design: সব component একসঙ্গে](06-system-design-and-security.md)
7. [Recap ও Interview Drill](07-recap-interview.md)

## কীভাবে পড়বে

প্রতিটি chapter-এ component-এর definition-এর পাশাপাশি execution flow ও ownership boundary বোঝো। Tool calling chapter-এ proposal বনাম execution, Skill chapter-এ guidance বনাম enforcement, MCP chapter-এ protocol বনাম orchestration এবং Gateway chapter-এ centralized policy বনাম domain authorization আলাদা করে চিনে নাও। শেষের integrated system design-এ সব concept একটি refund workflow-তে যুক্ত করো।

## শেখার ফল

- Tool call proposal ও actual execution-এর পার্থক্য ব্যাখ্যা।
- Tool schema, validation, authorization, execution ও result flow design।
- MCP host/client/server এবং tools/resources/prompts-এর boundary বোঝা।
- Skill package, progressive disclosure, versioning ও evaluation design।
- Multi-agent decomposition, delegation contract ও orchestration pattern বেছে নেওয়া।
- Gateway-এর access, policy, routing, credential, quota ও observability ভূমিকা বোঝা।
- Production failure, trust boundary ও security control ব্যাখ্যা করা।

## Source fidelity note

Module 2-এর topic outline—Tool Calling, MCP, Agent Skills, Multi-Agent System, AI Gateway ও integrated system design—ধরে notes সাজানো। সম্পূর্ণ video transcript/audio ও প্রতিটি on-screen demo independently verify করা হয়নি। তাই এগুলো topic-aligned detailed learning notes; ভিডিওর প্রতিটি বক্তব্যের exhaustive capture নয়।