# Agentic AI for Software Development

একটি AI model code লিখে দিতে পারে। কিন্তু code লেখা আর software তৈরি করে পরীক্ষা করে ঠিক করা এক জিনিস নয়। Coding agent repository বুঝতে পারে, file পরিবর্তন করে, test চালায় এবং test-এর ফল দেখে পরবর্তী কাজ ঠিক করতে পারে। এই ক্ষমতার জন্য model-এর পাশাপাশি instructions, context, tools ও control ব্যবস্থা লাগে।

## 1. LLM, Assistant, Agent

- **LLM:** তথ্য দেখে উত্তর বা code তৈরি করে এমন model। নিজে থেকে computer-এর file বা terminal ব্যবহার করতে পারে না।
- **Assistant:** LLM-কে ঘিরে তৈরি user-facing application। এতে chat interface, tools বা memory থাকতে পারে।
- **Agent:** Goal অর্জনের জন্য তথ্য দেখে সিদ্ধান্ত নেয়, action চালায়, ফল পর্যবেক্ষণ করে এবং প্রয়োজনে পরবর্তী action বদলায়।

~~~mermaid
flowchart TD
    A["Goal"] --> B["Observe context"]
    B --> C["Decide"]
    C --> D["Act using tool"]
    D --> E["Observe result"]
    E --> F{"Complete or stop?"}
    F -->|No| B
    F -->|Yes| G["Final result"]
~~~

Agent বিশেষভাবে কাজে লাগে এমন task-এ, যেখানে একটি action-এর ফল দেখে পরবর্তী কাজ বদলাতে হয়। তবে কাজ যদি সবসময় একই ধাপে চলে, তাহলে সাধারণ script বা fixed workflow সহজ, predictable এবং test করা সুবিধাজনক।

## 2. Agent Anatomy

| Component | কাজ | না থাকলে |
|---|---|---|
| Model | Reasoning/decision প্রস্তাব | Decision engine নেই |
| Instructions | Rules, role, scope, stop behavior | Inconsistent behavior |
| Context | Current task information | ভুল/incomplete decision |
| Tools | File/API/terminal/browser actions | শুধু text suggestion |
| Skills | Reusable task-specific instructions | একই rules বারবার লিখতে হয় |
| Workflow | Step, branch, state, completion | Steps বাদ/অগোছালো execution |
| Memory | Persisted information | Previous decisions ভুলে যেতে পারে |
| Orchestrator | Model, tools, state coordinate | Control flow ছড়িয়ে যায় |
| Guardrails | Permission, limits, validation | Uncontrolled actions risk |

## 3. Instructions

Instructions হলো Agent-এর কাজের নিয়ম: তার দায়িত্ব, scope, project conventions, কোন কাজ নিষিদ্ধ, কীভাবে verify করবে এবং কখন থামবে।

Example coding instructions:
- Repository inspect করার আগে edit নয়।
- Existing architecture/conventions follow।
- Unnecessary dependency নয়।
- Scope-এর বাইরে feature নয়।
- Tests add/run।
- Test pass claim করবে শুধু run করলে।
- Ambiguity বা risky action-এ clarification চাইবে।

Instructions না থাকলে Agent generic assumptions নিতে পারে, scope creep করতে পারে, test বাদ দিতে পারে বা project convention miss করতে পারে।

**Instructions বনাম Prompt:** Instructions Agent-এর স্থায়ী বা project-level নিয়ম বলে। Prompt বলে এই মুহূর্তে কী task করতে হবে। যেমন instruction: 'test চালাও'; prompt: 'Todo delete bug ঠিক করো'।

## 4. Tools

Tool হলো Agent-এর ব্যবহারযোগ্য capability। যেমন file পড়া, code edit করা, terminal command চালানো, test run করা বা API call করা।

Model কোনো tool ব্যবহার করার request তৈরি করে। Agent runtime request ও permission যাচাই করে tool চালায়, তারপর ফল model-এর কাছে ফেরত পাঠায়। Model সেই ফল দেখে পরবর্তী action ঠিক করে।

~~~mermaid
sequenceDiagram
    participant U as User
    participant A as Agent Runtime
    participant M as Model
    participant T as Tool
    U->>A: Goal
    A->>M: Context + tool definitions
    M-->>A: Tool proposal
    A->>A: Validate and authorize
    A->>T: Execute
    T-->>A: Result
    A->>M: Result
    M-->>A: Next action / final
~~~

Tools না থাকলে Agent code suggest করতে পারলেও নিজে file edit বা test চালাতে পারবে না। আবার unrestricted access দিলে ভুল command বা destructive action-এর ঝুঁকি থাকে। তাই প্রয়োজনীয় permission-ই দিতে হবে, risky action-এ approval রাখতে হবে।

## 5. Skills

Skill হলো কোনো নির্দিষ্ট ধরনের কাজ ভালোভাবে করার reusable guidance। এতে নির্দেশনা, process, example এবং প্রয়োজনে reference file বা helper script থাকতে পারে।

Example backend skill:
- Existing router/model patterns inspect।
- Pydantic request/response schema।
- Validation ও consistent errors।
- Route-এ business logic জমতে না দেওয়া।
- Tests add/run।
- Diff review।

Skill না থাকলে প্রতিবার একই নিয়ম prompt-এ লিখতে হয়। কোনো task-এ নিয়ম বাদ পড়লে Agent-এর কাজ inconsistent হতে পারে।

**Skill বনাম Tool:** Skill বলে কাজটি কীভাবে করতে হবে; Tool সেই কাজ করার ক্ষমতা দেয়। যেমন Skill API-এর project convention শেখায়, আর file-edit tool code পরিবর্তন করে।

Skill structure/metadata coding agent অনুযায়ী ভিন্ন হতে পারে। Skill auto-discovery/loading সব tool-এ একই নয়।

## 6. Workflow

Workflow হলো কাজের ধাপ, কোন ধাপের ওপর কোনটি নির্ভর করে, কোথায় সিদ্ধান্ত নিতে হবে এবং কাজ শেষ হয়েছে বলে কখন ধরা হবে, তার পরিকল্পনা।

~~~mermaid
flowchart TD
    A["Task"] --> B["Inspect repository"]
    B --> C["Clarify and plan"]
    C --> D["Implement"]
    D --> E["Test"]
    E --> F{"Pass?"}
    F -->|No| G["Diagnose and fix"]
    G --> E
    F -->|Yes| H["Review diff"]
    H --> I["Report"]
~~~

Workflow না থাকলে Agent code লিখেই থেমে যেতে পারে। Test, review বা documentation-এর মতো জরুরি step বাদ পড়ার সম্ভাবনা থাকে।

**Workflow বনাম Skill:** Workflow ঠিক করে কোন কাজ কখন হবে। Skill বলে সেই কাজ কীভাবে ভালোভাবে করতে হবে। যেমন workflow-এর 'test' ধাপে Testing Skill ব্যবহার করা যেতে পারে।

Deterministic steps workflow-তে রাখো; ambiguous decision প্রয়োজন হলে agent-কে decision নিতে দাও।

## 7. Context

Context হলো model-এর এই মুহূর্তের সিদ্ধান্ত নেওয়ার জন্য সরবরাহ করা তথ্য। এতে থাকতে পারে:
- Instructions।
- User task।
- Conversation history।
- Repository rules।
- Relevant source files।
- Tool definitions/results।
- Skills content।
- Task state/retrieved information।

প্রয়োজনীয় context না থাকলে Agent project-এর architecture বা API contract না বুঝে generic solution দিতে পারে, যা existing code-এর সঙ্গে না-ও মিলতে পারে।

**Instructions বনাম Context:** Instructions বলে কোন নিয়ম মেনে কাজ করতে হবে; Context বলে সেই কাজ বুঝতে কী তথ্য জানা দরকার।

## 8. Context Window

Context window হলো একটি model call-এ input ও output-এর জন্য model-এর token সীমা। Prompt, instructions, conversation, files, tool results এবং model-এর response এই সীমার মধ্যে জায়গা নেয়।

~~~mermaid
flowchart TD
    A["Context window budget"] --> B["Instructions"]
    A --> C["User prompt / history"]
    A --> D["Files / retrieved context"]
    A --> E["Tool definitions and results"]
    A --> F["Output"]
~~~

Context window-এর সীমা পূরণ হলে সব information একসঙ্গে দেওয়া সম্ভব হয় না। পুরোনো বা কম relevant information বাদ/summarize করতে হতে পারে, আর output-এর জন্যও জায়গা কমে যেতে পারে।

বড় context window থাকলেই ভালো result নিশ্চিত নয়। অপ্রাসঙ্গিক information দিলে noise, processing cost ও latency বাড়ে; গুরুত্বপূর্ণ detail-ও অনেক তথ্যের মধ্যে হারিয়ে যেতে পারে।

## 9. Context Engineering

Context Engineering হলো task অনুযায়ী প্রয়োজনীয় information খুঁজে বের করা, নির্বাচন করা, সাজানো, summarize করা এবং context window-এর budget ঠিকভাবে ব্যবহার করা।

- Relevant files search, পুরো repo blindly load নয়।
- Task অনুযায়ী project instructions ও skills load।
- Tool output প্রয়োজনীয় অংশে সীমিত।
- Long task-এ completed work summary/checkpoint।
- Stale বা irrelevant history বাদ।
- Context budget-এর মধ্যে output allowance।

Context Engineering না করলে অপ্রাসঙ্গিক information context ভরিয়ে দিতে পারে, গুরুত্বপূর্ণ detail বাদ পড়ে যেতে পারে, token ও cost নষ্ট হতে পারে।

Example: Login endpoint task-এ auth router, user schema, existing auth implementation ও relevant tests দরকার। Unrelated CSS বা analytics file সাধারণত দরকার নেই।

## 10. Memory

Memory হলো এমন information সংরক্ষণ করা, যা পরবর্তী step বা session-এ আবার কাজে লাগতে পারে।

- **Working/task state:** Current plan, completed steps, errors, pending actions।
- **Conversation memory:** Previous interaction-এর useful decisions/preferences।
- **Project memory:** Architecture, conventions, known constraints।

Memory না থাকলে দীর্ঘ task বা নতুন session-এ আগের decision ও progress হারিয়ে যেতে পারে, ফলে একই কাজ আবার করা বা পুরোনো ভুলের পুনরাবৃত্তি হতে পারে।

Memory আর Context এক নয়। Memory-তে information save থাকে; কিন্তু model তখনই তা ব্যবহার করতে পারে, যখন প্রয়োজনীয় memory retrieve করে বর্তমান context-এ দেওয়া হয়। পুরোনো বা ভুল memory বিভ্রান্ত করতে পারে, তাই memory update ও review করা দরকার।

## 11. Repository Instructions / AGENTS.md

Repository-specific instruction file-এ project overview, architecture, coding convention, test/build command, forbidden action ও documentation rules থাকতে পারে।

Instructions file না থাকলে agent convention discover করার চেষ্টা করে, কিন্তু project-specific rule miss করতে পারে।

কোন file auto-load হয়, nested instruction কীভাবে apply হয়, precedence কী, তা agent runtime-এর উপর নির্ভর করে।

## 12. Subagents

Subagent হলো Main Agent-এর দেওয়া নির্দিষ্ট, সীমাবদ্ধ task নিয়ে আলাদাভাবে কাজ করা agent।

Example: Main agent coordinates; test-analysis agent missing tests identify করে; review agent changes inspect করে।

~~~mermaid
flowchart TD
    A["Main Agent"] --> B["Implementation Agent"]
    A --> C["Test Analysis Agent"]
    A --> D["Review Agent"]
    B --> E["Code Changes"]
    C --> F["Test Findings"]
    D --> G["Review Findings"]
    E --> H["Main validates and integrates"]
    F --> H
    G --> H
~~~

Subagent দিয়ে independent কাজ parallel করানো যায়, ফলে সময় বাঁচতে পারে। তবে token cost, coordination, conflicting changes এবং result review-এর ঝামেলা বাড়ে। ছোট বা tightly coupled কাজ ভাগ করলে উল্টো complexity বাড়তে পারে।

## 13. MCP

Model Context Protocol (MCP) হলো AI application ও external capability-এর মধ্যে যোগাযোগের একটি standard protocol। MCP server tools, resources বা prompts expose করতে পারে, আর client সেগুলো discover ও ব্যবহার করতে পারে।

MCP server-এর মাধ্যমে filesystem, database, GitHub বা অন্য service-এর capability পাওয়া যেতে পারে। MCP integration সহজ করে, কিন্তু নিরাপত্তা নিশ্চিত করে না। Server-এর permission আলাদাভাবে সীমিত করতে হবে।

- **Agent:** Goal-directed decision/action system।
- **Tool calling:** Model/application interaction pattern।
- **MCP:** Capability integration protocol।

MCP ছাড়াও tool calling করা যায়; MCP ব্যবহার করলেই agent তৈরি হয় না।

## 14. Guardrails

Agent কী করতে পারবে, তা সীমিত রাখো:
- Tool allowlist।
- Scoped read/write।
- Command allowlist বা sandbox।
- Network restriction।
- Step/time/token/cost budget।
- Approval gates।
- Output validation।
- Diff review ও tests।

Prompt-এ dangerous command চালাবে না লেখা runtime permission-এর বিকল্প নয়।

# Practical Example: Todo App-এর Coding Agent Setup

একটি Todo app develop করব:
- Todo create/list।
- Complete/incomplete toggle।
- Delete।
- SQLite persistence।

Stack: FastAPI, SQLAlchemy, Pydantic, React, TypeScript, SQLite।

Authentication, tags, priorities, notifications scope-এর বাইরে। Agent আগে repo inspect করবে, existing files blindly overwrite করবে না।

## Project instructions: AGENTS.md

~~~markdown
# Todo App Rules
- Inspect existing files before editing.
- Follow architecture and conventions.
- Backend: FastAPI, SQLAlchemy, Pydantic.
- Frontend: React, TypeScript.
- Validate input and handle errors.
- Avoid unnecessary dependencies and scope creep.
- Add tests and run them.
- Do not delete unrelated files.
- Ask before major architecture changes.
- Report actual test results and limitations.
~~~

## Skills

Backend skill reusable process:
- Inspect existing endpoint patterns।
- Schema validation।
- Error/authorization conventions।
- Business logic separation।
- Tests add/run।
- Diff review।

Frontend skill:
- Small components।
- Typed props/API data।
- Loading, error, empty, success states।
- Responsive UI।
- API logic separate from presentation।
- Key interactions test।

Testing skill:
- Happy path, invalid input, failure cases, state transitions।
- Run relevant tests।
- Report actual result।

Coding agent-এর skill support ও skill load করার নিয়ম tool অনুযায়ী আলাদা। সব tool একই structure ব্যবহার করে না।

## Workflow

1. Inspect repository, summarize conventions।
2. Requirements clarify ও API contract propose।
3. Backend implement, tests run।
4. Frontend implement, checks run।
5. Integration verify।
6. Diff review।
7. Final report।

~~~mermaid
flowchart TD
    A["Todo requirements"] --> B["Repository inspection"]
    B --> C["Plan and API contract"]
    C --> D["Backend + tests"]
    D --> E{"Backend verified?"}
    E -->|No| D
    E -->|Yes| F["Frontend + checks"]
    F --> G{"Frontend verified?"}
    G -->|No| F
    G -->|Yes| H["Integration tests"]
    H --> I["Diff review and report"]
~~~

## Main agent prompt

~~~text
Build a small Todo application in this repository.

Features:
- Create todos with title.
- List todos.
- Mark complete/incomplete.
- Delete todos.
- Persist using SQLite.

Backend: FastAPI, SQLAlchemy, Pydantic.
Frontend: React, TypeScript.

Do not add authentication or features outside this scope.
Avoid unnecessary dependencies.

Process:
1. Read AGENTS.md and inspect repository.
2. Summarize structure and conventions.
3. Propose implementation plan and API contract.
4. Ask before major architecture changes.
5. Implement backend and run backend tests.
6. Implement frontend and run relevant checks.
7. Verify integration and core flows.
8. Review diff for unrelated changes.
9. Report files changed, tests actually run, and limitations.

Ask when requirements are ambiguous.
Never claim a test passed unless it was run.
~~~

## Context selection

Backend task-এ load: backend instructions/skill, routes, models/schemas, DB setup, relevant tests।

Frontend task-এ load: frontend instructions/skill, API contract, components, TypeScript config।

Failure debugging-এ load: error, failed test, relevant source/test, recent diff।

পুরো repo একবারে context-এ ঢোকানোর বদলে relevant context select করলে noise ও token cost কমে।

## Bounded failure loop

Agent max tool calls, time budget ও fix attempts enforce করবে। Test failure হলে error inspect, focused fix, relevant test rerun। Repeated failure বা scope drift হলে stop করে report করবে।

## Verification

- Backend tests run।
- Frontend checks/build run।
- Create/list/complete/delete flow verify।
- Persistence verify।
- API contract frontend-এর সঙ্গে match।
- Error states handle।
- Diff reviewed, unrelated changes নেই।

Agent 'Done' বললেই কাজ ঠিক হয়েছে প্রমাণ হয় না। Test সত্যিই run হয়েছে কি না এবং ফল কী, তা যাচাই করতে হবে।

## Components একসঙ্গে

~~~mermaid
flowchart TD
    U["User Goal"] --> A["Agent Runtime"]
    I["Instructions"] --> A
    C["Relevant Context"] --> A
    M["Memory"] --> C
    S["Skills"] --> A
    W["Workflow / State"] --> A
    A --> D["Model Decision"]
    D --> T["Tools"]
    T --> R["Execution Result"]
    R --> C
    R --> W
    A --> G["Guardrails / Budget"]
    G --> X["Continue or Stop"]
    X --> A
    X --> F["Final Report"]
~~~

| Concept | কেন দরকার | না থাকলে |
|---|---|---|
| Agent | Multi-step goal পরিচালনা | Manual step-by-step |
| Instructions | Behavior consistent | Generic assumptions |
| Context | Task বুঝতে তথ্য | ভুল/incomplete decisions |
| Context window | Processing capacity limit | Limit ছাড়ালে context/output সমস্যা |
| Context engineering | Relevant info manage | Noise ও token waste |
| Tool | বাস্তব action | শুধু suggestion |
| Skill | Reusable task knowledge | Instructions repeat |
| Workflow | Step/dependency/completion | Step skip/disorder |
| Memory | Persistent decisions/state | Progress ভুলে যাওয়া |
| Subagent | Delegated independent tasks | Sequential work |
| MCP | External capability integration | Custom integration overhead |
| Guardrails | Safe boundary | Uncontrolled action risk |

## Interview Questions

1. LLM, assistant ও agent-এর পার্থক্য?
2. Agent loop explain করো।
3. Agent কখন ব্যবহার করবে না?
4. Context কী? Context window কী?
5. Context window full হলে কী হয়?
6. Context window বড় হলেই quality বাড়ে?
7. Context engineering কী?
8. Context ও memory-এর পার্থক্য?
9. Skill কী? Skill ও tool-এর পার্থক্য?
10. Skill ও workflow-এর পার্থক্য?
11. Workflow না থাকলে কী সমস্যা?
12. MCP কী? MCP ও tool calling একই?
13. Tool authorization কোথায় enforce করবে?
14. Agent wrong file edit করলে কী করবে?
15. Context overflow হলে recover কীভাবে?
16. Agent test না চালিয়ে completion claim করলে?
17. Repeated failure loop আটকাবে কীভাবে?
18. Subagents conflicting changes করলে resolve কীভাবে?

## Mental Model

Model decision প্রস্তাব করে। Instructions behavior guide করে। Context current task-এর information দেয়। Context window capacity সীমিত করে। Context engineering relevant information বেছে নেয়। Tools action করতে দেয়। Skills task-specific process/knowledge দেয়। Workflow execution structure/state manage করে। Memory persistence দেয়। Orchestrator coordination করে। Guardrails permission ও limits enforce করে।

Agent engineering মানে শুধু model-কে prompt করা নয়, model, context, capability, state, execution ও control একসঙ্গে design করা।
