# Agentic AI for Software Development

একটি LLM text generate করে। Coding agent repository inspect, file edit, command/test run, result observe এবং task-এর next step বেছে নিতে পারে। এটি controlled loop; model নিজে filesystem access পায় না, runtime তাকে tools দেয়।

## 1. LLM, Assistant, Agent

- **LLM:** Input context থেকে output generate করা model।
- **Assistant:** LLM-কে ঘিরে user-facing application; tools থাকতে পারে।
- **Agent:** Goal নিয়ে context ও capabilities ব্যবহার করে decision/action/observation loop চালানো system।

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

Agent প্রয়োজন multi-step task-এ, যেখানে action-এর result দেখে পরবর্তী action বদলাতে হয়। Fixed sequence হলে deterministic workflow সহজ, predictable ও testable।

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

Instructions Agent-এর operating rules: role, scope, conventions, forbidden actions, verification ও stop conditions।

Example coding instructions:
- Repository inspect করার আগে edit নয়।
- Existing architecture/conventions follow।
- Unnecessary dependency নয়।
- Scope-এর বাইরে feature নয়।
- Tests add/run।
- Test pass claim করবে শুধু run করলে।
- Ambiguity বা risky action-এ clarification চাইবে।

Instructions না থাকলে Agent generic assumptions নিতে পারে, scope creep করতে পারে, test বাদ দিতে পারে বা project convention miss করতে পারে।

**Instructions বনাম Prompt:** Instructions define করে Agent কীভাবে কাজ করবে; prompt/current task define করে এখন কী করতে হবে।

## 4. Tools

Tool হলো বাস্তব capability: read file, search, edit, terminal, test, git diff, browser, API call।

Model tool call propose করে। Runtime arguments validate, permission check করে tool execute এবং result model-কে ফেরত দেয়।

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

Tools না থাকলে Agent action-এর code বা instruction suggest করতে পারে, কিন্তু নিজে file edit/test পারবে না। Tool access বেশি হলে destructive action risk বাড়ে। Scope, permission, command limits ও approval দরকার।

## 5. Skills

Skill হলো reusable instructions, knowledge, workflow এবং supporting resources-এর package, যা নির্দিষ্ট task-এ Agent-কে সাহায্য করে।

Example backend skill:
- Existing router/model patterns inspect।
- Pydantic request/response schema।
- Validation ও consistent errors।
- Route-এ business logic জমতে না দেওয়া।
- Tests add/run।
- Diff review।

Skill না থাকলে একই guidance প্রতিটি prompt-এ দিতে হবে; consistency কমতে পারে।

**Skill বনাম Tool:** Skill শেখায় কীভাবে কাজ করবে; Tool দেয় কাজ করার capability। Skill বলে API কীভাবে implement করবে; file-edit tool actual file পরিবর্তন করে।

Skill structure/metadata coding agent অনুযায়ী ভিন্ন হতে পারে। Skill auto-discovery/loading সব tool-এ একই নয়।

## 6. Workflow

Workflow হলো task-এর steps, dependencies, branching, state ও completion criteria।

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

Workflow না থাকলে Agent test, review, documentation বা validation বাদ দিতে পারে।

**Workflow বনাম Skill:** Workflow বলে কোন step কখন হবে; skill বলে সেই step কীভাবে ভালোভাবে করবে। Workflow-এর test step testing skill ব্যবহার করতে পারে।

Deterministic steps workflow-তে রাখো; ambiguous decision প্রয়োজন হলে agent-কে decision নিতে দাও।

## 7. Context

Context হলো current model call-এ supplied information:
- Instructions।
- User task।
- Conversation history।
- Repository rules।
- Relevant source files।
- Tool definitions/results।
- Skills content।
- Task state/retrieved information।

Context না থাকলে Agent architecture, API contract বা decisions না জেনে generic implementation করতে পারে।

**Instructions বনাম Context:** Instructions বলে কীভাবে কাজ করবে; context বলে কাজ করার জন্য কী জানা দরকার।

## 8. Context Window

Context window হলো একটি model call-এর input ও output token capacity। Instructions, prompt, history, files, tool outputs ও generated response একই budget consume করে।

~~~mermaid
flowchart TD
    A["Context window budget"] --> B["Instructions"]
    A --> C["User prompt / history"]
    A --> D["Files / retrieved context"]
    A --> E["Tool definitions and results"]
    A --> F["Output"]
~~~

Context window ছোট/full হলে relevant information বাদ পড়তে পারে, history truncate হতে পারে বা output budget কমে যেতে পারে।

বড় context window-ও সব সমস্যা solve করে না: irrelevant context reasoning noise, cost ও latency বাড়ায়; important details buried হতে পারে।

## 9. Context Engineering

Context engineering হলো current task-এর জন্য information selection, retrieval, organization, summarization, update ও budget management।

- Relevant files search, পুরো repo blindly load নয়।
- Task অনুযায়ী project instructions ও skills load।
- Tool output প্রয়োজনীয় অংশে সীমিত।
- Long task-এ completed work summary/checkpoint।
- Stale বা irrelevant history বাদ।
- Context budget-এর মধ্যে output allowance।

Context engineering না থাকলে token waste, noise, missed detail, overflow ও cost বাড়ে।

Example: Login endpoint task-এ auth router, user schema, existing auth implementation ও relevant tests দরকার। Unrelated CSS বা analytics file সাধারণত দরকার নেই।

## 10. Memory

Memory হলো information persist করা, যাতে future step/session-এ reuse করা যায়।

- **Working/task state:** Current plan, completed steps, errors, pending actions।
- **Conversation memory:** Previous interaction-এর useful decisions/preferences।
- **Project memory:** Architecture, conventions, known constraints।

Memory না থাকলে long task-এ decisions ও progress ভুলে একই কাজ repeat হতে পারে।

Context ও memory এক নয়: memory stored information; current context হলো model call-এ actually supplied information। Relevant memory retrieve করে context-এ দিতে হয়। Memory stale, incorrect বা sensitive হতে পারে, তাই source, freshness, permission ও retention বিবেচনা করো।

## 11. Repository Instructions / AGENTS.md

Repository-specific instruction file-এ project overview, architecture, coding convention, test/build command, forbidden action ও documentation rules থাকতে পারে।

Instructions file না থাকলে agent convention discover করার চেষ্টা করে, কিন্তু project-specific rule miss করতে পারে।

কোন file auto-load হয়, nested instruction কীভাবে apply হয়, precedence কী, তা agent runtime-এর উপর নির্ভর করে।

## 12. Subagents

Subagent হলো parent agent-এর delegated bounded task নিয়ে independently কাজ করা agent।

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

Subagent independent work parallel করতে পারে, কিন্তু cost, latency, handoff, conflicting edits ও coordination complexity বাড়ায়। Clear independent deliverable না থাকলে ব্যবহার করো না।

## 13. MCP

Model Context Protocol (MCP) হলো AI application-এর tools/resources/prompts external system থেকে expose/use করার protocol।

MCP server filesystem, database, GitHub বা অন্য service capability expose করতে পারে। MCP integration সহজ করতে পারে, কিন্তু permission/safety guarantee করে না।

- **Agent:** Goal-directed decision/action system।
- **Tool calling:** Model/application interaction pattern।
- **MCP:** Capability integration protocol।

MCP ছাড়াও tool calling করা যায়; MCP ব্যবহার করলেই agent তৈরি হয় না।

## 14. Guardrails

Agent-এর capability সীমিত করো:
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

Skills agent tool অনুযায়ী discover/load হবে; সব coding environment একই structure use করে না।

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

Agent completion message নিজে verification নয়। Actual tool/test results review করতে হবে।

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
