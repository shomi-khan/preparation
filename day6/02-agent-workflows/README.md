# Agent Workflows, State and Safety

## ১. Workflow বনাম Agent
Workflow-এ steps ও transitions আগে থেকে নির্ধারিত। Agent dynamically next action নির্বাচন করতে পারে। Production-এ deterministic workflow ও সীমিত agent decision combine করা প্রায়ই ভালো।

## ২. Planning ও Execution
Agent task ভেঙে subtask plan করতে পারে। Plan ভুল বা stale হতে পারে; প্রতিটি action-এর আগে policy ও current state যাচাই করতে হবে।

## ৩. State
State-এ user request, intermediate result, tool output, retry count ও current step থাকতে পারে। State schema versioned ও explicit রাখো।

## ৪. Memory
- Short-term memory: current task/session context।
- Long-term memory: sessions-এর বাইরে persist করা user/project knowledge।
Memory write-এর permission, retention, privacy ও deletion policy দরকার।

## ৫. Multi-agent
একাধিক specialized agent collaborate করতে পারে, কিন্তু coordination, duplicated work, latency, cost ও security complexity বাড়ে। Single agent/workflow যথেষ্ট হলে multi-agent ব্যবহার কোরো না।

## ৬. LangChain ও LangGraph
LangChain model/tool integration ও common building block দেয়। LangGraph stateful graph/loop-based agent workflow orchestrate করতে পারে। Framework concept বোঝার বিকল্প নয়; raw tool-call loop দিয়েও basic agent তৈরি করা যায়।

## ৭. Termination Conditions
- Task complete।
- Max step reached।
- No valid tool/action।
- Budget/token limit।
- Timeout/cancellation।
- Human approval required।
Infinite loop আটকাতে hard limit জরুরি।

## ৮. Error Handling
Tool error structuredভাবে return করা যায়, যাতে agent retry বা graceful failure সিদ্ধান্ত নিতে পারে। একই failing tool বারবার call করতে দেওয়া যাবে না।

## ৯. Prompt Injection ও Tool Abuse
Retrieved content বা user input malicious instruction দিতে পারে। Tool allowlist, permission checks, argument validation, output sanitization ও approval boundary রাখতে হবে।

## ১০. Auditability
কোন model কোন tool call propose করেছে, application কী validate করেছে, কী execute হয়েছে এবং ফল কী ছিল তা trace করতে হবে। Sensitive data redaction জরুরি।

## ১১. Agent Evaluation
Task completion, tool selection accuracy, argument validity, unnecessary action, policy violation, latency ও cost মাপো। Success শুধু final answer দেখে বিচার করবে না।

## Practice
একটি bounded workflow বানাও: user question -> customer lookup -> result summarize। Max ৩টি tool call, read-only access এবং structured error handling রাখো।
