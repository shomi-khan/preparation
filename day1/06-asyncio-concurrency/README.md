# Asyncio and Concurrency

একটি backend service একসঙ্গে অনেক request পায়। প্রতিটি request-এর database বা external API response-এর জন্য অপেক্ষা করতে হতে পারে। এই অপেক্ষার সময় পুরো program-কে idle না রেখে অন্য কাজ চালানোই concurrency-এর বড় সুবিধা। Python-এ `asyncio`, threading এবং multiprocessing ভিন্ন ধরনের workload-এর জন্য ভিন্ন tool।

## ১. Concurrency এবং Parallelism কী?

- **Concurrency:** একাধিক কাজের progress এমনভাবে manage করা, যাতে তাদের execution সময়ের মধ্যে overlap করতে পারে।
- **Parallelism:** একাধিক কাজ সত্যিকার অর্থে একই সময়ে একাধিক execution unit-এ চলে।

Concurrency মানেই parallel execution নয়। একটি event loop এক thread-এ অনেক I/O task-এর progress manage করতে পারে।

~~~mermaid
flowchart TD
    A["Multiple tasks"] --> B["Concurrency"]
    A --> C["Parallelism"]
    B --> D["Tasks take turns / overlap in time"]
    C --> E["Tasks execute simultaneously"]
~~~

## ২. I/O-bound এবং CPU-bound কাজের পার্থক্য কী?

**I/O-bound:** Network request, database query, file বা external service-এর response-এর জন্য অপেক্ষা বেশি।

**CPU-bound:** বড় numerical computation, image processing, compression বা heavy calculation-এ CPU time বেশি লাগে।

Asyncio সাধারণত I/O-bound কাজের concurrency-তে উপকারী। CPU-bound Python code-কে শুধু `async` করলেই parallel বা দ্রুত হয় না।

| Workload | সাধারণত বিবেচ্য approach |
|---|---|
| অনেক network/database wait | asyncio |
| Blocking I/O বা thread-based library | threading / asyncio.to_thread |
| CPU-heavy Python computation | multiprocessing / process pool |
| Native library যা GIL release করে | threading-ও parallelism দিতে পারে |

## ৩. Coroutine কী?

`async def` দিয়ে coroutine function তৈরি হয়। Call করলে coroutine object পাওয়া যায়; function body সঙ্গে সঙ্গে run হয় না। Coroutine execute করতে `await` বা task scheduling প্রয়োজন।

~~~python
import asyncio

async def greet():
    await asyncio.sleep(1)
    return "Hello"

async def main():
    message = await greet()
    print(message)

asyncio.run(main())
~~~

~~~mermaid
flowchart TD
    A["async def greet()"] --> B["greet() called"]
    B --> C["Coroutine object created"]
    C --> D["await or schedule as Task"]
    D --> E["Coroutine executes"]
    E --> F["Returns result"]
~~~

## ৪. await কী করে?

`await` current coroutine-কে এমন awaitable-এর result-এর জন্য suspend করে, যা complete না-ও হতে পারে। Control event loop-এ ফিরে যায়, ফলে অন্য ready task চলতে পারে।

~~~python
async def main():
    result = await greet()
    print(result)
~~~

**গুরুত্বপূর্ণ:** `await` সবসময় অন্য task-কে চালানোর সুযোগ দেবে এমন নয়। Awaitable যদি সঙ্গে সঙ্গে complete হয়, suspension নাও ঘটতে পারে। Scheduling event loop ও awaitable-এর behavior-এর উপর নির্ভর করে।

## ৫. Event Loop কী?

Event loop coroutine task schedule করে, I/O readiness ও timer-এর মতো event monitor করে এবং ready task resume করে। একটি coroutine যখন non-blocking I/O-এর জন্য await করে, তখন loop অন্য ready task চালাতে পারে।

~~~mermaid
sequenceDiagram
    participant L as Event Loop
    participant A as Task A
    participant B as Task B
    L->>A: Run A
    A-->>L: await network I/O
    L->>B: Run B
    B-->>L: await timer
    L->>A: Resume when I/O ready
    A-->>L: Complete
    L->>B: Resume when timer ready
    B-->>L: Complete
~~~

## ৬. Sequential বনাম Concurrent I/O কীভাবে compare করব?

~~~python
import asyncio
import time

async def fetch(name: str) -> str:
    await asyncio.sleep(1)  # network I/O-এর placeholder
    return f"{name} done"

async def sequential():
    return [
        await fetch("service-a"),
        await fetch("service-b"),
        await fetch("service-c"),
    ]

async def concurrent():
    return await asyncio.gather(
        fetch("service-a"),
        fetch("service-b"),
        fetch("service-c"),
    )

async def main():
    for operation in (sequential, concurrent):
        start = time.perf_counter()
        result = await operation()
        print(result, time.perf_counter() - start)

asyncio.run(main())
~~~

Sequential version-এ একটির কাজ শেষ হওয়ার পর পরেরটি শুরু হয়। Concurrent version-এ awaitable-গুলো একসঙ্গে schedule হয়। এখানে sleep network wait-এর placeholder, real network benchmark নয়।

~~~mermaid
flowchart TD
    A["Three independent I/O operations"] --> B["Sequential"]
    A --> C["Concurrent"]
    B --> D["A completes → B → C"]
    C --> E["A, B, C start; event loop interleaves waits"]
~~~

## ৭. Coroutine, Task এবং Future-এর পার্থক্য কী?

- **Coroutine:** `async def` call থেকে তৈরি awaitable execution object।
- **Task:** Event loop-এ coroutine schedule করে তার execution manage করে।
- **Future:** এমন object যা পরে পাওয়া যাবে এমন result বা exception represent করে। Task Future-এর behavior-ও দেয়।

~~~python
import asyncio

async def work():
    await asyncio.sleep(0.1)
    return "done"

async def main():
    task = asyncio.create_task(work())
    result = await task
    print(result)

asyncio.run(main())
~~~

`create_task()` coroutine-কে schedule করে। Task-এর result await করো, অথবা অন্যভাবে lifecycle ও exception manage করো।

## ৮. asyncio.gather কী করে?

`asyncio.gather()` একাধিক awaitable concurrentভাবে চালায় এবং input order অনুযায়ী result list দেয়।

~~~python
results = await asyncio.gather(
    fetch("a"),
    fetch("b"),
    fetch("c"),
)
~~~

Default `return_exceptions=False` হলে একটি awaitable exception raise করলে gather সেই exception caller-এ propagate করে। অন্য submitted awaitable-গুলো স্বয়ংক্রিয়ভাবে cancel হয়েছে এমন ধরে নেওয়া যাবে না। `return_exceptions=True` দিলে exception result list-এ value হিসেবে আসে, তাই প্রতিটি result inspect করতে হয়।

## ৯. TaskGroup কী? gather-এর থেকে আলাদা কেন?

Python 3.11+ এ `asyncio.TaskGroup` structured concurrency দেয়। Group-এর ভিতরে তৈরি child task-গুলোর lifecycle একটি scope-এর মধ্যে থাকে। কোনো child task failure করলে group sibling task-গুলো cancel করে এবং scope exit-এর সময় failure report করে।

~~~python
import asyncio

async def worker(name):
    await asyncio.sleep(0.2)
    return f"{name} done"

async def main():
    async with asyncio.TaskGroup() as group:
        first = group.create_task(worker("A"))
        second = group.create_task(worker("B"))

    print(first.result(), second.result())

asyncio.run(main())
~~~

| gather | TaskGroup |
|---|---|
| Result list দেয় | Task object group-এর ভিতরে তৈরি |
| Default failure propagate করে; siblings cancel হবে এমন নয় | Child failure হলে siblings cancel করে |
| Independent awaitable collect করতে সুবিধাজনক | Related task-এর lifecycle এক scope-এ রাখতে সুবিধাজনক |
| Python-এর পুরোনো version-গুলোতেও আছে | Python 3.11+ |

~~~mermaid
flowchart TD
    A["TaskGroup scope"] --> B["Create Task A"]
    A --> C["Create Task B"]
    A --> D["Create Task C"]
    B --> E{"Child failure?"}
    E -->|হ্যাঁ| F["Cancel unfinished siblings"]
    F --> G["Wait for child cleanup"]
    G --> H["Raise grouped failure"]
    E -->|না| I["Wait until all children complete"]
~~~

## ১০. Semaphore কেন প্রয়োজন?

একসঙ্গে কতগুলো operation চলবে তা সীমিত করতে semaphore ব্যবহার করা যায়। External API rate limit, database connection capacity বা memory constraint মানতে এটি সাহায্য করে।

~~~python
import asyncio

semaphore = asyncio.Semaphore(3)

async def limited_request(item: int):
    async with semaphore:
        await asyncio.sleep(0.2)
        return item * 2

async def main():
    results = await asyncio.gather(
        *(limited_request(i) for i in range(10))
    )
    print(results)

asyncio.run(main())
~~~

এখানে সর্বোচ্চ তিনটি coroutine semaphore-এর protected section-এ একসঙ্গে ঢুকতে পারবে। Semaphore API-এর actual rate-per-second policy নিজে থেকে implement করে না; rate limiting আলাদা concern।

## ১১. Timeout কীভাবে handle করব?

Network বা external operation অনির্দিষ্টকাল wait করতে পারে না। Timeout দিয়ে সর্বোচ্চ অপেক্ষার সময় নির্ধারণ করা হয়।

~~~python
import asyncio

async def slow_operation():
    await asyncio.sleep(5)

async def main():
    try:
        async with asyncio.timeout(1):
            await slow_operation()
    except TimeoutError:
        print("Operation timed out")

asyncio.run(main())
~~~

`asyncio.timeout()` Python 3.11+ এ আছে। পুরোনো code-এ `asyncio.wait_for()` দেখা যায়। Timeout হলে cancellation mechanism ব্যবহার হয়; তাই coroutine-এর cleanup logic ঠিক থাকা জরুরি।

## ১২. Cancellation কীভাবে কাজ করে?

Task cancel করলে তার coroutine-এ `CancelledError` inject করা হয়। Coroutine cleanup করে cancellation propagate করতে পারে।

~~~python
import asyncio

async def worker():
    try:
        while True:
            print("Working")
            await asyncio.sleep(1)
    finally:
        print("Cleanup")

async def main():
    task = asyncio.create_task(worker())
    await asyncio.sleep(0.1)
    task.cancel()

    try:
        await task
    except asyncio.CancelledError:
        print("Task cancelled")

asyncio.run(main())
~~~

Cancellation request মানেই task সঙ্গে সঙ্গে থেমে যায় না; coroutine cancellation point-এ exception পায়। `CancelledError` অযথা swallow কোরো না। Cleanup-এর জন্য `finally` ব্যবহার করো।

## ১৩. Blocking call event loop-কে কীভাবে আটকে দেয়?

Async function-এর ভিতরে `time.sleep()` বা blocking HTTP/database library call করলে event loop thread আটকে যায়। অন্য coroutine-ও তখন progress করতে পারে না।

খারাপ:

~~~python
import time

async def bad():
    time.sleep(2)  # event loop block করে
~~~

ভালো:

~~~python
import asyncio

async def good():
    await asyncio.sleep(2)
~~~

Blocking I/O library ব্যবহার করতেই হলে `asyncio.to_thread()` দিয়ে worker thread-এ চালানো যায়:

~~~python
import asyncio
import time

def blocking_io():
    time.sleep(1)
    return "finished"

async def main():
    result = await asyncio.to_thread(blocking_io)
    print(result)

asyncio.run(main())
~~~

`to_thread()` মূলত blocking I/O-bound function-এর জন্য। Pure Python CPU-heavy code-এর জন্য process pool বিবেচনা করো।

## ১৪. Threading, Multiprocessing এবং Asyncio-এর পার্থক্য কী?

| Approach | Execution model | ভালো use case |
|---|---|---|
| asyncio | Event loop, cooperative task switching | অনেক concurrent I/O |
| threading | এক process-এ multiple OS thread | Blocking I/O, thread-based integration |
| multiprocessing | Multiple process | CPU-bound parallel work |

~~~mermaid
flowchart TD
    A["Choose concurrency model"] --> B{"Workload"}
    B -->|Many non-blocking I/O waits| C["asyncio"]
    B -->|Blocking I/O / thread API| D["threading"]
    B -->|CPU-heavy Python work| E["multiprocessing"]
~~~

## ১৫. GIL কী এবং এটি কি threading-কে অকার্যকর করে?

GIL (Global Interpreter Lock) CPython-এর traditional build-এ একই interpreter-এর মধ্যে এক সময়ে একটি thread-কে Python bytecode execute করতে দেয়। ফলে pure Python CPU-bound thread সাধারণত একাধিক core-এ parallel speedup পায় না।

তবে threading অকার্যকর নয়:
- I/O wait-এর সময় thread অন্য কাজ করতে পারে।
- কিছু native extension কাজের সময় GIL release করে।
- Python-এর free-threaded build-এ GIL disabled করে চালানোর option আছে, তবে ecosystem compatibility ও performance trade-off বিবেচনা করতে হয়।

তাই “Python-এ threading কখনো parallel নয়” বলা ভুল। Interpreter build ও workload গুরুত্বপূর্ণ।

## ১৬. Asyncio-তে race condition হতে পারে?

হ্যাঁ। Asyncio single-threaded হলেও একাধিক task shared mutable state access করতে পারে। কোনো task await-এ suspend হলে অন্য task state পরিবর্তন করতে পারে।

~~~python
import asyncio

counter = 0
lock = asyncio.Lock()

async def increment():
    global counter
    async with lock:
        current = counter
        await asyncio.sleep(0)
        counter = current + 1

async def main():
    await asyncio.gather(*(increment() for _ in range(100)))
    print(counter)

asyncio.run(main())
~~~

Lock critical section protect করে। তবে lock-এর ভিতরে অপ্রয়োজনীয় দীর্ঘ I/O করলে contention বাড়ে। Shared state design কমিয়ে আনাও ভালো সমাধান হতে পারে।

## ১৭. Async Queue কী কাজে লাগে?

`asyncio.Queue` producer ও consumer task-এর মধ্যে work item pass করতে সাহায্য করে। Bounded queue দিলে queue পূর্ণ হলে producer wait করে, যা backpressure তৈরি করে।

~~~python
import asyncio

async def producer(queue):
    for item in range(5):
        await queue.put(item)
    await queue.put(None)  # sentinel

async def consumer(queue):
    while True:
        item = await queue.get()
        try:
            if item is None:
                return
            print("Processing", item)
        finally:
            queue.task_done()

async def main():
    queue = asyncio.Queue(maxsize=2)
    async with asyncio.TaskGroup() as group:
        group.create_task(producer(queue))
        group.create_task(consumer(queue))

asyncio.run(main())
~~~

বাস্তব ব্যবহারে একাধিক consumer হলে shutdown sentinel-এর সংখ্যা এবং task lifecycle ঠিকভাবে manage করতে হবে।

## ১৮. Async generator কী?

Async generator function-এ `async def` ও `yield` একসঙ্গে থাকে। এটি asynchronous stream-এর item একে একে দেয় এবং `async for` দিয়ে consume করা যায়।

~~~python
import asyncio

async def events():
    for number in range(3):
        await asyncio.sleep(0.1)
        yield number

async def main():
    async for event in events():
        print(event)

asyncio.run(main())
~~~

~~~mermaid
flowchart TD
    A["async generator"] --> B["async for requests next item"]
    B --> C["Generator awaits I/O"]
    C --> D["Yield one item"]
    D --> E["Consumer processes item"]
    E --> B
~~~

## Common Mistakes

1. Async function call করে await না করা।
2. Coroutine object তৈরি করে কখনো schedule/await না করা।
3. Async code-এর ভেতরে দীর্ঘ blocking call চালানো।
4. সীমাহীন task তৈরি করে external service overload করা।
5. Timeout না রাখা।
6. Cancellation ও exception handling উপেক্ষা করা।
7. Task-এর exception/result retrieve না করা।
8. Shared mutable state lock ছাড়া update করা।
9. CPU-bound কাজ event loop-এ চালিয়ে loop block করা।

## Quick Interview Revision

| প্রশ্ন | সংক্ষিপ্ত উত্তর |
|---|---|
| Concurrency বনাম parallelism? | Overlapping progress বনাম simultaneous execution |
| Coroutine call করলে? | Coroutine object তৈরি হয় |
| Task কী? | Event loop-এ scheduled coroutine |
| Event loop কী? | Async task ও I/O readiness schedule করে |
| await কী করে? | Current coroutine suspend করে, awaitable complete হওয়া পর্যন্ত |
| gather কী দেয়? | Input order অনুযায়ী result list |
| TaskGroup কেন? | Structured child-task lifecycle |
| Semaphore কেন? | Concurrent operation-এর সংখ্যা limit |
| Timeout কেন? | Unbounded waiting এড়াতে |
| Cancellation কীভাবে? | Task-এ CancelledError inject হয় |
| Blocking call-এর সমস্যা? | Event loop আটকে দেয় |
| to_thread কী? | Blocking function worker thread-এ চালায় |
| CPU-bound Python work? | সাধারণত process pool |
| Asyncio parallel? | সাধারণত এক event loop thread-এ নয় |
| Asyncio-তে race condition? | হ্যাঁ, await boundary-তে interleaving হতে পারে |
| Async Queue? | Producer-consumer coordination ও backpressure |
| Async generator? | Asynchronously item yield করে |

## Practice Exercises

1. তিনটি simulated API call sequential এবং concurrentভাবে চালিয়ে elapsed time তুলনা করো।
2. ২০টি simulated API call চালাও; semaphore দিয়ে active call ৩-এ সীমিত করো এবং maximum concurrency measure করো।
3. একটি operation-এ timeout যোগ করো এবং timeout-এর পর cleanup হয়েছে কি না test করো।
4. দুইটি TaskGroup child task তৈরি করো। একটি fail করাও; sibling cancellation ও exception behavior observe করো।
5. একটি task cancel করো এবং `finally` cleanup execute হয় কি না verify করো।
6. Blocking function `asyncio.to_thread()` দিয়ে চালাও; একই সময়ে অন্য coroutine progress করছে কি না observe করো।
7. `asyncio.Queue(maxsize=2)` দিয়ে producer-consumer pipeline তৈরি করো।
8. Shared counter race condition reproduce করো এবং `asyncio.Lock` দিয়ে fix করো।
9. Async generator দিয়ে paginated result simulate করো।
10. একটি coroutine-এর exception retrieve না করলে কী warning/behavior দেখা যায় তা experiment করো।

## Practice করার পদ্ধতি

1. প্রতিটি task কখন start, suspend, resume এবং complete হয় তা timeline-এ লেখো।
2. Normal completion, exception, timeout ও cancellation আলাদাভাবে test করো।
3. Concurrency limit এবং backpressure বাস্তব workload-এ কেন দরকার ব্যাখ্যা করো।
4. CPU-bound ও I/O-bound workload-এর জন্য আলাদা approach justify করো।
5. Python version ও interpreter build উল্লেখ করে GIL-related claim যাচাই করো।
