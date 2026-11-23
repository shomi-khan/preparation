# 07. Module 2 Recap ও Interview Drill

## 90-second recap

Tool calling external capability ব্যবহার করতে দেয়। Model structured call propose করে; host/runtime validate, authorize ও execute করে। MCP host/client/server-এর মধ্যে capability discover ও access করার common protocol; এটি tools/resources/prompts expose করতে পারে। Agent Skill tool access-এর বদলে reusable workflow ও instruction দেয়, progressive loading দিয়ে relevant context-এ আসে। Multi-agent system responsibility ভাগ করে, কিন্তু coordination, handoff, latency, cost ও failure propagation বাড়ায়। AI Gateway tool access-এর সামনে routing, auth, policy, quota, credential ও observability centralize করতে পারে। সব component থাকলেও security/correctness trusted server-side layer-এ enforce করতে হবে।

## Quick comparison

| Concept | মূল প্রশ্ন |
|---|---|
| Agent | Goal নিয়ে কে সিদ্ধান্ত/coordination করে? |
| Tool | কোন কাজ execute করতে পারে? |
| Skill | কাজটি কীভাবে ভালোভাবে করবে? |
| MCP | Capability standardized ভাবে কীভাবে discover/call হবে? |
| Multi-agent | Responsibility কীভাবে ভাগ/coordinate হবে? |
| Gateway | Access কীভাবে govern/route/monitor হবে? |

## Foundational interview

**1. Tool calling কীভাবে কাজ করে?**  
Schema model-কে দেওয়া হয়; model call propose করে; runtime parse/validate/authorize/execute করে; result context-এ ফেরত দিয়ে model continue বা final answer দেয়।

**2. MCP কেন?**  
AI host/client ও external capability server-এর মধ্যে reusable standard interface দিতে, bespoke integration কমাতে।

**3. MCP Host, Client, Server?**  
Host application/runtime; client protocol connection; server capability expose ও request handle করে।

**4. Tool vs Skill?**  
Tool executable capability; Skill reusable instruction/workflow।

**5. Multi-agent কখন?**  
Task decomposition/specialization/parallelism বাস্তব value দিলে; baseline-এর তুলনায় measurable improvement থাকলে।

**6. AI Gateway?**  
Tools/services-এর access intermediary, policy, routing, credential, quota ও observability manage করে।

## Deep interview questions

1. Model schema-valid arguments দিলেও business rule violate করলে কোথায় reject?
2. Tool timeout-এর পর duplicate side effect কীভাবে আটকাবে?
3. Tool description বনাম authorization policy?
4. Compromised MCP server-এর blast radius কীভাবে limit?
5. Skill outdated হলে version/review/evaluation?
6. 100 tools দিলে selection degradation কীভাবে measure?
7. Parallel agent partial failure কীভাবে handle?
8. Handoff context/contract কীভাবে নিশ্চিত?
9. Gateway ও downstream service দুটিতে authorization কেন?
10. Gateway outage-এর failure mode?
11. Tool-call audit-এ কী রাখবে, কী redact?
12. Tool output prompt injection defense?
13. User, skill, retrieved doc-এর trust boundary?
14. MCP কি orchestration, memory, reasoning replace করে?
15. Parent agent child-agent contract কীভাবে define করবে?

## System design exercise

Design a Refund Agent Platform. Diagram-এ:
- Host/runtime, LLM
- Skill catalog
- Tool catalog
- MCP client/server
- Gateway/policy
- Order/refund API
- Memory/state
- Human approval
- Audit/tracing

Component ownership ও trust boundary explain করো। Failure paths: timeout, policy deny, approval pending, duplicate refund, service outage।

## Self-review checklist

- [ ] Tool call proposal বনাম execution
- [ ] Schema ও business validation
- [ ] Authorization ও approval
- [ ] MCP Host/Client/Server
- [ ] Tools/Resources/Prompts
- [ ] Transport/deployment trade-offs
- [ ] Skill vs Tool
- [ ] Progressive disclosure
- [ ] Single vs Multi-agent
- [ ] Sequential/parallel/supervisor/handoff
- [ ] Gateway responsibilities/limitations
- [ ] Least privilege, idempotency, timeout, trace
- [ ] Prompt injection ও untrusted tool output

## Mock interview

Interviewer: “আমাদের agent-কে Slack, Jira, GitHub, database ও deployment system access দিতে হবে। সব tools এক agent-কে দেব, নাকি multiple agents? MCP ও Gateway কোথায় বসবে? Skills দরকার? Production security কীভাবে করব?”

উত্তর:
1. Requirements/risk clarify।
2. Single-agent baseline।
3. Tool boundary/schema।
4. Skill workflow।
5. MCP integration decision।
6. Multi-agent-এর measurable justification।
7. Gateway/security।
8. Observability/evaluation।
9. Failure handling/human approval।

Framework-এর নাম বলার চেয়ে component-এর responsibility, trade-off, failure mode ও enforcement point ব্যাখ্যা করাই বেশি গুরুত্বপূর্ণ।