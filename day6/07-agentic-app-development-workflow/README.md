# Building an Application with Agentic AI

এই guide-এর লক্ষ্য হলো interview-তে পরিষ্কারভাবে বোঝাতে পারা: একটি application তৈরি করতে AI agent কীভাবে ব্যবহার করব, কোন কাজ AI করবে, কোন কাজ deterministic হবে, এবং কীভাবে quality ও token cost নিয়ন্ত্রণ করব।

## 1. প্রথমে একটি গুরুত্বপূর্ণ ধারণা

Agentic development মানে অনেকগুলো Agent বানিয়ে তাদের দিয়ে সব কাজ করানো নয়। বরং software development process-কে এমনভাবে design করা, যাতে:

- AI reasoning দরকার এমন জায়গায় Agent কাজ করে।
- নির্দিষ্ট ও repeatable কাজ script, CLI বা normal code করে।
- মানুষ product decision, ambiguity এবং risky action-এ control রাখে।
- প্রতিটি ধাপের output পরের ধাপে structuredভাবে যায়।
- Code change বাস্তব test ও independent review দিয়ে verify হয়।

**সবচেয়ে কার্যকর নীতি: use the smallest amount of AI autonomy that safely solves the task.**

## 2. End-to-end workflow at a glance

~~~mermaid
flowchart TD
    A["1. Understand the goal"] --> B["2. Inspect repository and context"]
    B --> C["3. Confirm requirements and acceptance criteria"]
    C --> D["4. Design architecture and challenge the plan"]
    D --> E["5. Split into dependency-aware tasks"]
    E --> F["6. Choose agents, tools and skills only where needed"]
    F --> G["7. Orchestrate the workflow"]
    G --> H["8. Implement in small slices"]
    H --> I["9. Test and independently review"]
    I --> J{"Acceptance criteria met?"}
    J -->|No| K["Route findings to the owner"]
    K --> H
    J -->|Yes| L["Report and prepare PR"]
~~~

## 3. Step-by-step workflow

### Step 1: Understand the problem and decide whether an Agent is useful

প্রথমে feature request বা bug report-টি বোঝো।

প্রশ্ন করো:
- User-এর আসল সমস্যা কী?
- Expected behavior কী?
- কোন behavior পরিবর্তন হবে?
- কী কী scope-এর বাইরে?
- কাজটি কি ambiguous, নাকি পুরোপুরি deterministic?
- ভুল হলে impact কতটা?

যে কাজ fixed rule দিয়ে করা যায়, সেখানে script বা normal code যথেষ্ট। যেমন formatter চালানো, generated file তৈরি করা বা নির্দিষ্ট test suite চালানো। Repository বুঝে design decision নেওয়া বা unfamiliar bug investigate করার মতো কাজে Agent বেশি উপযোগী।

**Output:** concise problem statement, scope, non-goals, known constraints এবং open questions।

### Step 2: Inspect the repository before preparing context

Context আগে থেকে পুরো repository dump করে বানানো উচিত নয়। প্রথমে repository inspect করে বুঝতে হবে কোন অংশ task-এর সঙ্গে সম্পর্কিত।

পড়বে:
- Root/project instructions, যেমন `AGENTS.md`।
- Relevant architecture ও domain documentation।
- সংশ্লিষ্ট source code, tests, configuration।
- Related API contracts এবং recent changes।
- Existing implementation patterns।

এখানে একটি গুরুত্বপূর্ণ পার্থক্য আছে:

- **Canonical documentation** বলে intended/approved design কী।
- **Source code ও executable tests** বলে বর্তমানে কী implement এবং verify করা আছে।

দুটো এক না হলে conflict চিহ্নিত করবে। Target design-কে implemented behavior ধরে code লিখবে না।

**Output:** relevant context references এবং verified repository facts। অপ্রাসঙ্গিক file context-এ ঢোকাবে না।

### Step 3: Confirm requirements and define acceptance criteria

কাজের scope user বা responsible stakeholder-এর সঙ্গে confirm করো। Facts, assumptions, decisions এবং unresolved questions আলাদা রাখো।

প্রতিটি requirement-এর জন্য acceptance criteria লিখো। Criteria এমন হওয়া উচিত, যাতে পরে test বা observable evidence দিয়ে যাচাই করা যায়।

উদাহরণ:
- Todo তৈরি হলে list-এ দেখা যাবে।
- Page refresh করলেও Todo থাকবে।
- Invalid title দিলে validation error আসবে।
- Delete cancel করলে data অপরিবর্তিত থাকবে।

User-visible behavior বদলালে normal flow-এর পাশাপাশি invalid input, failure এবং edge case-ও define করো।

**কেন দরকার?** Acceptance criteria না থাকলে Agent নিজের মতো করে "কাজ শেষ" নির্ধারণ করতে পারে।

**Output:** confirmed requirements, user flows where needed, acceptance criteria, non-goals।

### Step 4: Design the solution and challenge the plan

এখন Architect role বা architecture-capable Agent:
1. Existing design ও code inspect করবে।
2. System boundary ও component ownership বুঝবে।
3. Interface/API/data contract নির্ধারণ করবে।
4. Dependencies ও implementation order ঠিক করবে।
5. Failure, compatibility, migration এবং rollback impact বিবেচনা করবে।
6. Draft plan-কে adversarialভাবে challenge করবে।
7. Critical finding থাকলে plan revise করে আবার challenge করবে।

একটি ভালো plan-এর প্রতিটি task-এ থাকবে:
- কী deliver করবে।
- কোন file/module প্রভাবিত হতে পারে।
- কোন task-এর ওপর নির্ভরশীল।
- কোন contract বদলাবে।
- কীভাবে verify হবে।

কোনো critical decision-এর জন্য user-এর তথ্য দরকার হলে অনুমান করে implementation শুরু করবে না। Block করে নির্দিষ্ট প্রশ্ন করবে।

**Output:** one authoritative, dependency-aware implementation plan।

### Step 5: Break the plan into small, verifiable tasks

Task ছোট হবে, কিন্তু অতিরিক্ত ছোট নয়। প্রতিটি task-এর scope, input, output এবং completion condition পরিষ্কার থাকতে হবে।

ভালো task:
- "Todo create endpoint-এর validation এবং persistence implement করো; success ও invalid-input tests যোগ করো।"

খারাপ task:
- "Backend পুরোটা বানাও।"

আবার প্রতিটি line of code-এর জন্য আলাদা task বানানোও অকার্যকর। Task এমনভাবে ভাগ করো যাতে একটি meaningful behavior বা cohesive layer complete ও verify করা যায়।

Dependencies স্পষ্ট করো। Independent task parallel হতে পারে; tightly coupled task sequential হওয়া উচিত।

**Output:** ordered task list বা dependency graph।

### Step 6: Decide the minimum agent setup

এখন নির্ধারণ করো কোন role বা Agent প্রয়োজন। শুরুতেই প্রতিটি task-এর জন্য আলাদা Agent বানাবে না।

সাধারণত এই ownership যথেষ্ট:
- **Business/Requirement Agent:** user intent ও acceptance criteria পরিষ্কার করে।
- **Architect Agent:** design, contract, dependency ও implementation plan তৈরি করে।
- **Coder Agent:** approved plan অনুযায়ী code ও tests তৈরি/পরিবর্তন করে।
- **Reviewer Agent:** আলাদাভাবে design, tests, implementation ও final evidence review করে।

ছোট task-এ একই general-purpose coding agent একাধিক দায়িত্ব নিতে পারে। Independent review দরকার হলে reviewer আলাদা রাখা বেশি কার্যকর।

একটি Agent role মানেই আলাদা model বা আলাদা long-running process নয়। এটি আলাদা instructions, responsibility এবং handoff contract-সহ invocation-ও হতে পারে।

**Output:** minimal role map, responsibility boundaries ও handoff rules।

### Step 7: Prepare instructions, skills and context

এই তিনটি এক জিনিস নয়:

| বিষয় | প্রশ্নের উত্তর |
|---|---|
| Instructions | Agent কী নিয়ম মেনে কাজ করবে? |
| Skill | নির্দিষ্ট ধরনের কাজ কীভাবে করবে? |
| Context | এই task বুঝতে কী তথ্য দরকার? |

**Instructions**-এ project convention, scope, forbidden actions, reporting format ও stop conditions রাখো।

**Skills** reusable হলে তৈরি করো। যেমন backend API skill, database migration skill, frontend accessibility skill, test-design skill। Skill না থাকলে guidance instructions বা task prompt-এ দেওয়া যায়। Skill format ও loading behavior tool-specific।

**Context** task অনুযায়ী just-in-time load করো। পুরো repo, সব chat history বা সব docs একসঙ্গে পাঠিও না। Relevant paths, contract, tests ও upstream artifacts দাও।

**Output:** role instructions, optional reusable skills, task-specific context references।

### Step 8: Integrate tools with least privilege

Agent-এর প্রয়োজনীয় tools বেছে নাও:
- Read/search files।
- Edit files।
- Run tests/build/lint।
- Inspect diff।
- Git operations।
- প্রয়োজন হলে browser বা external API।

প্রতিটি tool-এর permission সীমিত করো। Read-only investigation-এর জন্য write permission দিও না। Destructive operation, deployment, production data বা external side effect-এর আগে approval রাখো।

MCP external tools/resources integrate করার একটি standard protocol; MCP বাধ্যতামূলক নয়। Direct tool integration থাকলে MCP যোগ করার প্রয়োজন নেই।

**Output:** minimum tool set, permission policy, approval boundaries।

### Step 9: Build the orchestrator around control flow, not intelligence

Orchestrator-এর কাজ Agent-এর মতো সব reasoning করা নয়। এর কাজ workflow-কে reliableভাবে পরিচালনা করা।

Orchestrator সাধারণত:
- Workflow state সংরক্ষণ করে।
- কোন stage ready তা নির্ধারণ করে।
- Dependency ও legal transition validate করে।
- নির্দিষ্ট role-এর জন্য prompt/handoff প্রস্তুত করে।
- Artifact path ও output schema validate করে।
- Human approval gate পরিচালনা করে।
- Retry, failure, timeout ও budget enforce করে।
- Event log এবং execution evidence রাখে।
- Completion report তৈরি করতে সাহায্য করে।

যেখানে সিদ্ধান্তের rule স্পষ্ট, orchestrator-এ deterministic logic রাখো। যেমন "test review pass না হলে report stage শুরু হবে না"। কিন্তু "এই error-এর root cause কী?" সেটি Agent-এর reasoning-এর কাজ।

**গুরুত্বপূর্ণ:** Orchestrator-কে সবসময় automatically Agent invoke করতেই হবে এমন নয়। Cursor-এর মতো tool-এ human manually Agent invoke করতে পারে; orchestrator শুধু state, prompt ও handoff manage করতে পারে। Automation পরে প্রয়োজন অনুযায়ী যোগ করা যায়।

**Output:** explicit state machine/workflow, handoff contract, gates, retry policy ও logs।

### Step 10: Implement in small slices with test-first discipline

প্রতিটি implementation slice-এর জন্য:
1. Coder approved plan ও relevant artifacts পড়বে।
2. Acceptance criteria থেকে tests লিখবে বা update করবে।
3. Test design independentভাবে review হবে।
4. Coder implementation করবে।
5. Relevant tests চালাবে।
6. Implementation diff review হবে।
7. Failure হলে সংশ্লিষ্ট owner-এর কাছে finding পাঠানো হবে।

একটি task complete হওয়ার আগে পরবর্তী dependent task শুরু করবে না। Independent task parallel করা যায়, তবে shared files বা shared state-এ conflict হতে পারে কি না আগে দেখতে হবে।

### Step 11: Independently verify the whole change

Coder-এর "Done" message proof নয়। Independent verification দরকার।

- Tests সত্যিই run হয়েছে?
- কোন test pass, fail, skipped বা not run?
- Acceptance criteria প্রতিটি cover হয়েছে?
- API/schema compatibility ঠিক আছে?
- Error ও edge case handle হয়েছে?
- Unrelated file পরিবর্তন হয়েছে?
- Security, performance বা migration impact আছে?
- Documentation update প্রয়োজন ছিল কি?

Reviewer evidence দেখে verdict দেবে। Failed check আর not-run check আলাদা করে report করবে।

### Step 12: Report, PR and learn

Final report-এ থাকবে:
- কী পরিবর্তন হয়েছে।
- কোন requirements পূরণ হয়েছে।
- কোন files/artifacts তৈরি বা update হয়েছে।
- কোন commands/tests বাস্তবে run হয়েছে।
- Test result ও evidence।
- Known limitations, risks ও skipped checks।

তারপর human review-এর জন্য PR তৈরি হবে। Merge/deploy-এর মতো high-impact action explicit approval ছাড়া হবে না।

---

## 4. Token ও cost efficient করার নিয়ম

| Technique | কীভাবে সাহায্য করে |
|---|---|
| Just-in-time context | শুধু current task-এর relevant file load হয় |
| Artifact handoff | পুরো conversation পাঠানোর বদলে concise summary ও file references যায় |
| One authoritative plan | একই plan বারবার regenerate বা duplicate হয় না |
| Reusable instructions/skills | একই guidance প্রতিটি prompt-এ লিখতে হয় না |
| Deterministic orchestration | fixed decision-এর জন্য বারবার LLM call লাগে না |
| Small cohesive tasks | Context কম লাগে, failure isolate করা সহজ |
| Targeted tests | প্রতিটি ছোট change-এ অপ্রয়োজনীয় full test suite চালাতে হয় না; final stage-এ full verification হয় |
| Bounded retries | একই ভুলে endless model calls বন্ধ হয় |
| Parallelize only independent work | coordination ও duplicate context কমে |
| Summarize at boundaries | দীর্ঘ history বহন না করে verified state রাখা যায় |

একটি গুরুত্বপূর্ণ trade-off: সবসময় সবচেয়ে ছোট context বা কম test চালানোই efficient নয়। Context বা verification এত কমালে চলবে না যে Agent ভুল assumption নেয় বা defect ধরা না পড়ে। লক্ষ্য হলো **minimum sufficient context and verification**।

## 5. কোন কাজ Agent করবে, কোন কাজ deterministic হবে?

| কাজ | উপযুক্ত পদ্ধতি | কারণ |
|---|---|---|
| Requirement ambiguity বুঝে প্রশ্ন করা | Agent + Human | Judgment ও product decision দরকার |
| Relevant code খুঁজে বের করা | Agent/tool | Repository navigation ও reasoning |
| Architecture proposal | Architect Agent | Trade-off analysis দরকার |
| Fixed stage transition | Deterministic orchestrator | Rule নির্দিষ্ট |
| JSON/schema validation | Code/CLI | Exact validation দরকার |
| Code implementation | Coder Agent | Context অনুযায়ী code changes |
| Formatting/lint/test command | Script/CI | Repeatable execution |
| Test failure root cause | Agent | Evidence interpret করতে হয় |
| Permission/approval | Runtime + Human | Safety ও authority |
| Final acceptance/merge | Reviewer + Human | Accountability দরকার |

## 6. Common mistakes

1. **শুরুতেই অনেক Agent বানানো:** role overlap, coordination ও token cost বাড়ে।
2. **সব context একবারে দেওয়া:** irrelevant information ও context overflow বাড়ে।
3. **একটি বিশাল task দেওয়া:** ভুল isolate ও review করা কঠিন।
4. **প্রতিটি ছোট কাজ Agent দিয়ে করানো:** deterministic কাজেও token খরচ হয়।
5. **Agent-কে unrestricted tools দেওয়া:** destructive action-এর ঝুঁকি।
6. **Coder-কেই নিজের কাজের একমাত্র reviewer করা:** একই ভুল review-তে থেকে যেতে পারে।
7. **Test run না করে completion বিশ্বাস করা:** code লেখা মানেই behavior verified নয়।
8. **Orchestrator-এ business reasoning ঢোকানো:** workflow brittle ও hard to maintain হয়।
9. **একাধিক competing plan রাখা:** downstream Agent ভুল বা stale plan পড়তে পারে।
10. **Unbounded retry/parallelism:** cost, latency ও inconsistent changes বাড়ে।

## 7. Interview-তে বলার সংক্ষিপ্ত উত্তর

"আমি প্রথমে feature-এর goal, scope এবং acceptance criteria clarify করি। এরপর repository inspect করে relevant architecture, instructions, source code ও tests থেকে task-specific context তৈরি করি। Requirements confirmed হলে Architect দিয়ে solution design ও dependency-aware plan তৈরি করাই, এবং plan-টি critical risks ও edge cases দিয়ে challenge করি।

তারপর plan-টিকে ছোট, independently verifiable task-এ ভাগ করি। প্রয়োজন অনুযায়ী Business Analyst, Architect, Coder এবং Reviewer role ব্যবহার করি; শুরুতেই অপ্রয়োজনীয় অনেক Agent তৈরি করি না। Reusable guidance-এর জন্য instructions ও skills রাখি, আর প্রতিটি task-এ শুধু relevant context দিই।

একটি deterministic orchestrator workflow state, task dependencies, handoff, approvals, artifact validation, retry limits এবং execution logs পরিচালনা করে। Fixed rules orchestrator বা scripts দিয়ে চালাই; reasoning দরকার হলে Agent invoke করি। Coder test-first পদ্ধতিতে ছোট slice implement করে। Independent Reviewer test design, code diff ও actual test evidence যাচাই করে। সব acceptance criteria pass হলে final report ও PR তৈরি করি।

Efficiency-এর জন্য just-in-time context, structured artifact handoff, reusable skills, targeted tests, deterministic steps এবং bounded retries ব্যবহার করি। এতে token cost কমে, কিন্তু quality ও human control বজায় থাকে।"

## 8. One-line mental model

**Human defines the outcome → Agents reason and create artifacts → Deterministic orchestration controls the process → Tools perform actions → Tests and reviewers verify the result → Human approves high-impact decisions.**
