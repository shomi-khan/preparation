# Day 4 Practical Exercises

## Exercise 1: Tokenization
একটি বাংলা ও ইংরেজি paragraph tokenizer দিয়ে encode করো। Token count, character count ও word count compare করো।

## Exercise 2: Prompt Design
একটি support ticket classify করার zero-shot prompt তৈরি করো। এরপর ৩টি few-shot example যোগ করে output consistency compare করো।

## Exercise 3: Structured Output
Ticket classification-এর Pydantic schema বানাও। Invalid category, missing field ও malformed response test করো।

## Exercise 4: Prompt Injection
Customer message-এ "ignore previous instructions" এবং secret বের করার request যোগ করো। দেখো application policy ও tool boundary কীভাবে তা আটকায়। Prompt-only defense যথেষ্ট কি না ব্যাখ্যা করো।

## Exercise 5: LLM API Wrapper
একটি wrapper implement করো:
- Input validation
- Timeout
- Bounded retry
- Error classification
- Response validation
- Request ID
- Token/latency logging

## Exercise 6: Evaluation Set
কমপক্ষে ২০টি representative ticket লিখে expected category/priority annotate করো। Prompt change-এর আগে/পরে accuracy ও format validity compare করো।

## Deliverables
- Runnable Python project।
- Prompt template ও schema।
- Test cases।
- Evaluation report।
- ৩ মিনিটের architecture walkthrough।

## Self-review
- [ ] Output schema validate করেছি।
- [ ] Hallucination-এর সীমাবদ্ধতা বুঝেছি।
- [ ] Prompt injection-কে security problem হিসেবে নিয়েছি।
- [ ] Tool permission সীমিত করেছি।
- [ ] Timeout/retry implement করেছি।
- [ ] Sensitive data log করিনি।
- [ ] Prompt evaluation করেছি।
