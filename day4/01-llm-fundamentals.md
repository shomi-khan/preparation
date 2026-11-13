# LLM and Transformer Fundamentals

## ১. LLM কী?
Large Language Model (LLM) হলো বড় text corpus-এ train করা neural network, যা context দেখে পরবর্তী token-এর probability estimate করে। এই capability দিয়ে text generation, summarization, classification, extraction ও conversation করা যায়।

## ২. Token কী?
Model text-কে token-এ process করে। Token word, word-এর অংশ, punctuation বা অন্য unit হতে পারে। Token count character/word count-এর সমান নয়। API cost ও context window সাধারণত token-ভিত্তিক।

## ৩. Tokenization
Tokenizer text-কে token ID-তে map করে; output token ID থেকে text decode করা যায়। Different model-এর tokenizer আলাদা হতে পারে।

## ৪. Embedding কী?
Embedding হলো token, sentence বা document-এর dense vector representation। Semantic similarity search ও retrieval-এ ব্যবহৃত হয়। Similar meaning-এর text vector space-এ কাছাকাছি থাকতে পারে, তবে model ও task-এর উপর নির্ভর করে।

## ৫. Transformer কী?
Transformer architecture attention mechanism ব্যবহার করে sequence-এর বিভিন্ন token-এর সম্পর্ক model করে। Encoder, decoder বা encoder-decoder variant থাকতে পারে। অনেক generative LLM decoder-only Transformer ভিত্তিক।

## ৬. Self-Attention
একটি token representation তৈরি করার সময় sequence-এর অন্য token-গুলোর information weight করে combine করা হয়। Query, Key, Value projection দিয়ে attention score ও weighted value তৈরি হয়।

Conceptually:
```text
Attention(Q, K, V) = softmax(QK^T / sqrt(d_k)) V
```

## ৭. Positional Information
Self-attention নিজে token order জানে না। তাই positional encoding/embedding বা positional mechanism দিয়ে sequence order information যোগ করা হয়।

## ৮. Next-token Prediction
Autoregressive language model আগের token দেখে পরবর্তী token predict করে। Generation চলতে থাকে EOS বা stopping condition না আসা পর্যন্ত।

## ৯. Context Window
একটি request-এ model কত token context process করতে পারে তার সীমা। Input prompt, conversation history, retrieved documents এবং output budget সবই context ব্যবহার করে।

## ১০. Temperature ও Top-p
- Temperature distribution-এর randomness/peakedness নিয়ন্ত্রণ করে।
- Top-p nucleus sampling probability mass-এর cumulative threshold অনুযায়ী candidate সীমিত করে।
এগুলো output guarantee করে না; model/provider অনুযায়ী behavior ভিন্ন হতে পারে।

## ১১. Hallucination
LLM fluent কিন্তু ভুল বা unsupported তথ্য তৈরি করতে পারে। কারণ এটি language pattern থেকে generation করে, database-এর মতো guaranteed fact lookup করে না।

Mitigation: trusted retrieval, constrained output, citations, verification, abstention policy ও evaluation।

## ১২. Fine-tuning বনাম Prompting বনাম RAG
- Prompting: request instruction/context বদলায়।
- RAG: external knowledge retrieve করে context-এ যোগ করে।
- Fine-tuning: model weights update করে behavior/style/task adaptation করে।
Fresh private knowledge-এর জন্য RAG প্রায়ই বেশি practical; style/task consistency-এর জন্য fine-tuning বিবেচ্য।

## Practice
1. একটি paragraph tokenizer দিয়ে token count করো।
2. Token count ও word count compare করো।
3. Context window budget কীভাবে ভাগ করবে তার example লেখো।
4. Hallucination-এর তিনটি কারণ ও mitigation লিখো।
