# Case Study: Agentic Development Workflow for Arch Puzzle

এই case study-তে Arch Puzzle-এর বর্তমান Phase 2 development context ব্যবহার করে দেখানো হয়েছে, কীভাবে একটি বাস্তব software project-এ Agentic AI workflow প্রয়োগ করা যায়।

**Source of truth:** Arch Puzzle-এর active root `docs/` এবং implementation repositories-এর actual source code ও executable tests। `old_docs/` archive; এটি current context, requirement বা implementation decision-এর evidence হিসেবে ব্যবহার করা যাবে না।

এই document-এ current approved rules এবং open PR-এর proposed UI/UX changes আলাদা করে দেখানো হয়েছে। PR merge না হওয়া পর্যন্ত তার content-কে canonical main-branch rule ধরে নেওয়া যাবে না।

## 1. Project context বুঝে নেওয়া

Arch Puzzle একটি system-design learning/simulation application। User Problem অনুযায়ী Application design করে, Cloud Infrastructure configure করে এবং Functional/Performance result দেখে।

বর্তমান Phase 2-তে Application Definition, Entry Point, Action composition, VM hosting, infrastructure এবং simulation execution boundary গুরুত্বপূর্ণ।

নতুন agent workflow চালুর আগে Agent-কে এই project context বুঝতে হবে:
- Product progression: `docs/product/DEVELOPMENT-CHAPTERS.md`
- System architecture: `docs/architecture/application-simulation.md`
- Application/API contract: `docs/api/APPLICATION-CONTRACT.md`
- Current simulation behavior: `docs/simulation/`
- Backend/frontend ownership: `docs/backend/`, `docs/frontend/`
- Test strategy: `docs/testing/`
- Workflow rules: `docs/AGENT-WORKFLOW.md`, `docs/AGENT-INVOCATION.md`, `docs/WORKFLOW-ORCHESTRATION.md`

শুধু task-এর সঙ্গে relevant অংশ load করতে হবে। পুরো documentation tree প্রতিটি Agent prompt-এ পাঠানো efficient নয়।

### Target design বনাম implemented behavior

Arch Puzzle-এর কিছু document approved target architecture বর্ণনা করে, কিন্তু তা current implementation-এ সম্পূর্ণ implement হয়েছে এমন নয়। তাই:
- Product/architecture/API docs থেকে intended contract বোঝো।
- Backend/frontend source code ও executable tests থেকে implemented behavior যাচাই করো।
- Target docs-কে implementation evidence হিসেবে ব্যবহার করো না।
- Cloud component context লেখার সময় সংশ্লিষ্ট BE source ও executable tests-ই implementation evidence।
- কোনো behavior implement না থাকলে সেটিকে implemented বলে দাবি করো না।

## 2. Arch Puzzle-এর জন্য সবচেয়ে গুরুত্বপূর্ণ workflow principle

Arch Puzzle-এ component ownership পরিষ্কার রাখতে হবে:

**যে component-এর behavior, state ও calculation তার নিজের component-ই own করবে। Simulation মূলত execution order, timing এবং component-গুলোর মধ্যে orchestration করবে।**

Agent-দের instructions ও architecture review-তে এই principle enforce করতে হবে। উদাহরণস্বরূপ, Simulation যেন Cache-এর internal hit/miss calculation নিজে না করে; Cache component-এর behavior তার নিজস্ব implementation-এ থাকবে।

এটি শুধু Cache-এর জন্য নয়, সব component boundary-এর জন্য প্রযোজ্য।

## 3. Arch Puzzle-এ Agentic workflow-এর architecture

এখানে সব Agent স্বয়ংক্রিয়ভাবে একে অপরকে invoke করবে, এমন architecture দরকার নেই। বর্তমান project workflow অনুযায়ী Cursor-এ role manually invoke করা হয়; Go workflow script state, prompts, handoff ও validation পরিচালনা করে।

~~~mermaid
flowchart TD
    U["Developer / Human"] --> E["Manual Entry Agent"]
    E --> BA["Business Analyst"]
    E --> AR["Architect"]
    E --> CS["Cloud Semantics"]
    BA --> WF["Go Workflow Manager"]
    AR --> WF
    CS --> WF
    WF --> G{"Context approval needed?"}
    G -->|Yes| H["Human Gate"]
    H --> CA["Original Agent applies approved context"]
    G -->|No| P["Selected workflow stages"]
    CA --> P
    P --> TA["Test Authoring"]
    TA --> TR["Test Design Review"]
    TR --> CO["Coder"]
    CO --> IR["Implementation Review"]
    IR --> AT["Automated Tests"]
    AT --> TV["Test Review"]
    TV --> RP["Report"]
    RP --> RR["Report Review"]
    RR -->|Pass| DONE["Workflow complete"]
    RR -->|Findings| FIX["Route to responsible owner"]
    FIX --> P
~~~

### Current implementation-এর গুরুত্বপূর্ণ সীমা

- Entry role manually Cursor-এ invoke করা হয়।
- Go workflow script Cursor Agent-কে নিজে automatically invoke করে না।
- Script workflow state, stage transition, prompts, artifact handoff, gate ও event log পরিচালনা করে।
- Agent result structured JSON payload হিসেবে workflow-তে ফেরত দেয়।
- Orchestrator নিজে product/architecture reasoning করে না; responsible Agent সেই কাজ করে।

অতএব, Arch Puzzle-এর জন্য প্রথম লক্ষ্য হওয়া উচিত existing workflow manager-কে সঠিকভাবে ব্যবহার করা এবং তার reliability বাড়ানো। নতুন autonomous multi-agent framework বানানো নয়।

## 4. একটি feature implement করার ধাপ

ধরা যাক, Phase 2-এর Application Design-এ একটি confirmed behavior change implement করতে হবে। Workflow হবে:

### Step 1: Entry role দিয়ে scope clarify

Task অনুযায়ী একটি entry role নির্বাচন:

| Entry role | দায়িত্ব |
|---|---|
| Business Analyst | User-visible behavior, business rules, acceptance criteria ও user flow |
| Architect | Architecture, contracts, dependencies, trade-offs ও implementation plan |
| Cloud Semantics | Cloud component-এর verified semantics, simulation behavior ও scenario |

প্রথমে Cursor-এ Agent-এর সঙ্গে সরাসরি আলোচনা হবে। Facts, assumptions, confirmed decisions এবং unresolved questions আলাদা করতে হবে। User-এর explicit confirmation ছাড়া requirement final নয়।

কোন role entry হবে তা feature/bug/refactor label দিয়ে নয়, কাজের মূল ownership দেখে ঠিক করতে হবে।

### Step 2: প্রয়োজনীয় canonical context update

User-visible business behavior নতুন বা পরিবর্তিত হলে feature folder-এ:
- `requirements.md`
- `user-flows.md`

রাখতে হবে। Expected behavior অপরিবর্তিত থাকলে শুধু implementation change-এর জন্য user-flow document তৈরি বাধ্যতামূলক নয়।

Entry Agent canonical docs সরাসরি পরিবর্তন করবে না। পরিবর্তনের proposal লিখবে:

`.work/<workflow_id>/requirement.md`

এখানে proposed changes ও exact canonical target paths থাকবে।

যদি proposal থাকে:
1. Workflow `context-approval` human gate চালাবে।
2. User approve করলে একই original entry Agent আবার invoke হবে।
3. Agent exact approved proposal canonical docs-এ apply করবে।
4. Workflow approved target set ও repository change validate করবে।

Proposal না থাকলে context gate ও context-apply skip হবে।

### Step 3: Architecture plan তৈরি ও challenge

Architect একই stage-এর মধ্যে এই lifecycle অনুসরণ করবে:

~~~mermaid
flowchart LR
    A["UNDERSTAND"] --> B["INSPECT"]
    B --> C["DESIGN"]
    C --> D["PLAN"]
    D --> E["CHALLENGE"]
    E --> F{"Critical finding?"}
    F -->|Yes| G["REVISE"]
    G --> E
    F -->|No| H["FINALIZE"]
~~~

Architect:
- Confirmed requirements ও constraints বুঝবে।
- Relevant source, tests, contracts ও docs inspect করবে।
- Component boundary, API/data contract এবং dependency design করবে।
- Ordered implementation task ও acceptance check লিখবে।
- Compatibility, edge case, failure path ও testability challenge করবে।
- Critical finding থাকলে plan revise করে আবার challenge করবে।
- Missing evidence বা user decision ছাড়া safe plan করা না গেলে stage `BLOCKED` করবে।

একটিই authoritative plan থাকবে:

`.work/<workflow_id>/agents/architecture-plan.md`

এটি downstream Agent-দের artifact handoff-এ দেওয়া হবে। আলাদা plan-approval human gate বা আলাদা plan-review stage নেই।

### Step 4: Test-first implementation pipeline

Finalized plan থেকে Coder-এর আগে Test Authoring stage test design তৈরি করবে।

| Stage | Owner | দায়িত্ব |
|---|---|---|
| `test-authoring` | Test Author | Requirements ও plan অনুযায়ী test design |
| `test-authoring-review` | Reviewer | Test coverage, edge case ও correctness review |
| `implementation` | Coder | Approved scope implement |
| `implementation-review` | Reviewer | Code ও architecture compliance review |
| `automated-testing` | Test runner/script | Independent test execution |
| `test-review` | Reviewer | Actual test evidence যাচাই |
| `report` | Reporting stage | Result, limitations ও evidence report |
| `report-review` | Reviewer | Requirements, plan ও evidence-এর সঙ্গে final report মিলিয়ে দেখা |

Test fail করলে "সব test pass" বলে report করা যাবে না। Failed, skipped এবং not-run checks আলাদা করতে হবে।

### Step 5: Findings সঠিক owner-এ ফেরত পাঠানো

Final report review-এ issue পাওয়া গেলে সংশ্লিষ্ট owner-এ ফেরত যাবে:

| Finding | Owner |
|---|---|
| Code defect | Coder |
| Architecture/design/contract issue | Architect |
| Requirement/user-flow gap | Business Analyst |
| Cloud semantics issue | Cloud Semantics |

যে stage-এ correction দরকার, সেই stage এবং downstream affected review/test/report stage আবার চলবে। Unaffected upstream approval অকারণে reset হবে না।

### Step 6: Completion ও PR

সব review pass হলে workflow complete হবে। Final report-এ থাকবে:
- Verified changes।
- Actual test commands ও outcomes।
- Evidence paths।
- Limitations ও remaining risks।

Commit কেবল user request করলে। PR তৈরির আগে branch diff, tests, docs এবং scope verify করতে হবে।

## 5. Orchestrator-এর দায়িত্ব: Arch Puzzle-এ কী deterministic হবে?

Arch Puzzle-এর Go workflow manager-এর কাজ:

| Deterministic workflow manager | Agent-এর reasoning |
|---|---|
| Stage order ও legal transition validate | Requirement ambiguity resolve |
| Workflow state ও append-only event log | Architecture trade-off |
| Next-agent prompt তৈরি | Implementation plan challenge |
| Artifact path validate ও handoff | Code/test design |
| Human gate state record | Test failure root cause |
| Retry ও owner routing | Fix strategy |
| Test command/script execute | Evidence interpret |
| Completion condition enforce | Final technical judgment |

Workflow manager-এর source of truth হলো append-only `events.jsonl`; workflow CLI দিয়ে state replay ও transition করতে হবে। Agent-কে event log বা orchestration report manually edit করতে দেওয়া যাবে না।

Handoff-এ workflow manager ঠিক দুইটি field পাঠায়:

~~~json
{
  "summary": "Most recent completed upstream agent summary",
  "artifacts": [
    ".work/<workflow_id>/agents/architecture-plan.md",
    ".work/<workflow_id>/agents/test-design.md"
  ]
}
~~~

Artifact content prompt-এ dump করা হয় না। Receiving Agent path পড়ে প্রয়োজনীয় অংশ নেয়। Missing বা unsafe artifact থাকলে incomplete handoff তৈরি না করে workflow block করতে হবে।

## 6. Context ও token efficiency: Arch Puzzle-এর জন্য practical rules

1. **একটি task, একটি context pack:** শুধু সংশ্লিষ্ট domain docs, source files, tests ও contract references দাও।
2. **Architecture plan একবার তৈরি:** একই plan বিভিন্ন Agent prompt-এ copy না করে artifact path handoff করো।
3. **Full conversation নয়, summary + artifacts:** পরের Agent-কে শুধু latest upstream summary ও recorded artifacts দাও।
4. **No context duplication:** canonical fact এক জায়গায় রাখো; অন্য docs থেকে link করো।
5. **Target vs implemented আলাদা:** approved target contract ও actual code evidence গুলিয়ে ফেলো না।
6. **Relevant test আগে:** প্রতিটি slice-এ সংশ্লিষ্ট tests চালাও; final automated-testing stage-এ required broader suite চালাও।
7. **Retry bounded ও evidence-based:** failure report ও specific feedback handoff-এ দাও; একই prompt blindভাবে repeat করো না।
8. **Unrelated agents invoke নয়:** task-এর ownership অনুযায়ী entry role ও stage নির্বাচন করো।
9. **Manual invocation আপাতত গ্রহণযোগ্য:** Cursor Agent auto-invocation না থাকলেও workflow manager handoff ও state reliable রাখতে পারে।
10. **Secrets ও production access নয়:** Agent-কে minimum required permissions দাও।

## 7. Phase 2-তে Arch Puzzle-এর জন্য বাস্তবায়ন অগ্রাধিকার

Arch Puzzle-এর অনেক অংশ Phase 2-তে পরিবর্তনাধীন এবং কিছু behavior broken থাকতে পারে। তাই agentic workflow-কে "পুরো application একবারে fix করো" বলা যাবে না।

এই ক্রমে এগোনো বেশি নিরাপদ:

### Priority 1: Workflow infrastructure reliable করা

- Workflow CLI-এর state transition, event replay ও crash recovery test।
- Artifact handoff-এর path validation ও missing-artifact blocking।
- Retry/rejection-এ stale downstream artifacts reset হচ্ছে কি না।
- Human gate approval/rejection ও context apply lifecycle।
- Stage result schema validation।
- Report review owner routing এবং affected-stage reset।

কারণ workflow manager নিজেই unreliable হলে Agent-দের output ও state trace করা কঠিন হবে।

### Priority 2: Canonical context ও implementation truth align করা

- Active `docs/` tree-এর architecture/API/product contracts inspect।
- Backend ও frontend-এর actual source/tests inspect।
- Target contract বনাম implemented behavior gap তালিকা।
- Broken behavior-কে ছোট, independently verifiable task-এ ভাগ।
- Cloud component semantics কেবল BE source ও executable test evidence থেকে document করা।

### Priority 3: User-visible flow vertical slice

Problem Details-এর four-step progression ধরে একটি slice end-to-end implement ও verify করা:
- Step 1: Problem Details।
- Step 2: Application Design।
- Step 3: Cloud Infrastructure Design।
- Step 4: Results।

একটি step-এর UI, API, persistence, validation এবং test একসঙ্গে verify না করে অনেক layer-এ অসম্পূর্ণ কাজ ছড়িয়ে দেবে না।

### Priority 4: Simulation semantics ও component behavior

প্রতিটি component-এর behavior তার own implementation-এ থাকবে। Simulation execution order, timing এবং orchestration করবে। Functional Test, Application Execution Context, endpoint invocation checkpoint এবং MQ-related execution-এর contract অনুযায়ী tests তৈরি করতে হবে।

### Priority 5: Performance ও regression

Functional correctness stable হওয়ার পর performance test, load/scaling behavior, regression এবং cost-related simulation verify করবে।

এই priorities একটি proposed sequencing strategy, নতুন product scope বা approved architecture decision নয়। প্রতিটি বাস্তব কাজের আগে current source/tests ও confirmed Phase 2 scope inspect করতে হবে।

## 8. Open PR #84: কীভাবে workflow-তে বিবেচনা করবে?

**PR #84: `docs: specify Application Design UI/UX`** বর্তমানে open। এটি Application Design UI/UX specification-কে modular Problem Details structure-এ সাজায়।

PR-এ:
- Step 2 Application Design-এর detailed UI/UX ও user flow যোগ হয়েছে।
- Step 1, Step 3 ও Step 4-এর detailed UI/UX এখনো pending placeholder।
- Application/Action semantics, API contract ও four-step progression canonical docs-এ রাখার নিয়ম বজায় রাখা হয়েছে।
- MQ-কে existing Entry Point type হিসেবেই ধরা হয়েছে; নতুন capability হিসেবে যোগ করা হয়নি।
- এটি documentation-only; runtime code পরিবর্তন করে না।

**Workflow-এর জন্য implication:**
- PR merge হওয়ার আগে Step 2-এর নতুন UX details-কে proposed/unmerged context হিসেবে ধরবে।
- Step 2 implementation task এলে PR-এর branch ও diff inspect করে intended UI behavior বুঝবে, কিন্তু main-এর canonical docs-এর সঙ্গে conflict থাকলে merge/decision না হওয়া পর্যন্ত ambiguity resolve করবে।
- PR merge হলে updated canonical docs থেকে context তৈরি করবে।
- Step 1/3/4-এর placeholder থেকে কোনো unspecified UI behavior অনুমান করবে না।

PR link: https://github.com/shomi-khan/arch-puzzle/pull/84

## 9. Arch Puzzle নিয়ে interview-তে বলার example

"আমার personal project Arch Puzzle-এ আমি Agentic AI-কে সরাসরি code generation-এর জন্য নয়, বরং controlled software delivery workflow-এর অংশ হিসেবে ব্যবহার করছি।

প্রথমে Business Analyst, Architect বা Cloud Semantics role-এর সঙ্গে scope clarify করি। Confirmed requirement অনুযায়ী প্রয়োজন হলে canonical documentation-এর proposal তৈরি হয় এবং context পরিবর্তনের আগে human approval gate থাকে। Architect existing code, docs, API contract ও tests inspect করে dependency-aware implementation plan তৈরি করে এবং critical risk challenge করে।

এরপর Go-based deterministic workflow manager stage transition, state, artifact handoff, approval, retry এবং event log পরিচালনা করে। এটি Cursor Agent-কে নিজে automatically invoke করে না; role-based prompt তৈরি করে, আমি Cursor-এ সংশ্লিষ্ট Agent manually invoke করি। Agent-রা structured result ও artifact path ফেরত দেয়।

Implementation-এর আগে test design তৈরি ও independently review হয়। তারপর Coder ছোট scope implement করে। Independent test execution, test review এবং final report review হয়। Finding থাকলে responsible owner-এ ফেরত যায়।

এতে model-কে সবসময় পুরো repository বা conversation দিতে হয় না। Workflow manager summary ও artifact references handoff করে; Agent প্রয়োজনীয় file নিজে পড়ে। Fixed transition ও validation deterministic code-এ থাকে, আর requirement analysis, architecture reasoning ও failure diagnosis Agent করে। এই separation token cost কমায়, traceability বাড়ায় এবং human control বজায় রাখে।"

## 10. Key takeaways

- Arch Puzzle-এ Agent orchestration আর workflow orchestration এক জিনিস নয়।
- Go workflow manager state ও process control করে; Cursor Agent reasoning ও task execution করে।
- Agent invocation এখন manual; workflow manager prompt ও handoff প্রস্তুত করে।
- Requirements, architecture plan, test design, code ও reports হলো explicit artifacts।
- Context হলো প্রয়োজনীয় artifact references; পুরো repository prompt-এ ঢোকানো নয়।
- Target architecture ও actual implementation evidence আলাদা রাখতে হবে।
- `old_docs/` ব্যবহার করা যাবে না।
- Phase 2-তে একসঙ্গে সব broken area fix নয়; ছোট verified vertical slice-এ এগোতে হবে।
