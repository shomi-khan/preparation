# Day 6 Practical Exercises

## Exercise 1: Tool Calling
Calculator এবং read-only customer lookup tool বানাও। Tool arguments Pydantic দিয়ে validate করো।

## Exercise 2: Bounded Agent Loop
Agent-কে সর্বোচ্চ ৪টি tool call করতে দাও। Step limit, timeout ও explicit stop condition যোগ করো।

## Exercise 3: SQL Safety
Natural language question থেকে SELECT query তৈরি করো। Enforce করো:
- Read-only DB user।
- Single statement।
- Allowlisted tables/columns।
- Statement timeout।
- Maximum row count।
- No DDL/DML।

## Exercise 4: Human Approval
একটি mock refund tool যোগ করো। Refund execution-এর আগে user confirmation ছাড়া tool execute হবে না।

## Exercise 5: Failure Handling
Tool timeout, invalid arguments, permission denied ও provider rate limit simulate করো। Retryable/non-retryable error আলাদা করো।

## Exercise 6: Observability
প্রতিটি request-এর correlation ID, model, prompt version, tool call count, latency ও token usage record করো। Sensitive prompt content log করবে না।

## Exercise 7: Docker
একটি FastAPI application containerize করো। Non-root user, health endpoint ও environment-based configuration যোগ করো।

## Deliverables
- Runnable tool-calling agent।
- Tool schema ও permission tests।
- Architecture diagram।
- Failure/observability test।
- Dockerized application।
- Security limitation note।

## Self-review
- [ ] Model arbitrary code execute করতে পারে না।
- [ ] Tool arguments validate করি।
- [ ] Server-side authorization আছে।
- [ ] Step/time/budget limit আছে।
- [ ] Irreversible action-এ confirmation আছে।
- [ ] Retry bounded।
- [ ] Logs-এ sensitive data নেই।
- [ ] Cost ও latency measure করি।
