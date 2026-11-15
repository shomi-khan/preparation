# LLM and Transformer Fundamentals

Large Language Model (LLM) নিয়ে কাজ করতে গেলে শুধু prompt লেখা জানলেই হয় না। Model কীভাবে text represent করে, context থেকে next token predict করে, training-এর মাধ্যমে behavior শেখে এবং inference-এ token generate করে, তার mental model থাকা জরুরি।

## ১. LLM কী?

Large Language Model হলো বড় text corpus-এ train করা neural network, যা context দেখে পরবর্তী token-এর probability distribution estimate করে। এই capability দিয়ে text generation, summarization, classification, extraction, translation ও conversation করা যায়।

~~~mermaid
flowchart TD
    A["Context tokens"] --> B["LLM"]
    B --> C["Next-token probability distribution"]
    C --> D["Select next token"]
    D --> E["Append to context"]
    E --> B
~~~

LLM database নয়। এটি learned parameters থেকে language pattern generate করে; তাই fluent output মানেই factual correctness নয়।

## ২. Token কী? Word-এর সঙ্গে token-এর পার্থক্য কী?

Model text-কে token-এ process করে। Token word, word-এর অংশ, punctuation বা অন্য unit হতে পারে। Token count character/word count-এর সমান নয়।

~~~mermaid
flowchart LR
    A["Input text"] --> B["Tokenizer"]
    B --> C["Token IDs"]
    C --> D["Model"]
~~~

API cost ও context window সাধারণত token-ভিত্তিক। Different model-এর tokenizer আলাদা হতে পারে।

## ৩. Tokenization কীভাবে কাজ করে?

Tokenizer text-কে token ID-তে map করে; output token ID থেকে text decode করা যায়। Subword tokenization rare word-কে ছোট অংশে ভাগ করতে পারে। তাই একটি long word একাধিক token হতে পারে, আবার common phrase তুলনামূলক কম token-এ represent হতে পারে।

~~~python
# Provider/model-specific tokenizer API ব্যবহার করে
# token_ids = tokenizer.encode("Hello, world!")
# text = tokenizer.decode(token_ids)
~~~

Prompt size ও cost estimate করতে target model-এর tokenizer ব্যবহার করো; word count দিয়ে অনুমান সবসময় নির্ভুল নয়।

## ৪. Embedding কী?

Embedding হলো token, sentence বা document-এর dense vector representation। Semantic similarity search, clustering এবং retrieval-এ ব্যবহৃত হয়।

~~~mermaid
flowchart TD
    A["Text"] --> B["Embedding model"]
    B --> C["Dense vector"]
    C --> D["Vector similarity / retrieval"]
~~~

Similar meaning-এর text vector space-এ কাছাকাছি থাকতে পারে, তবে embedding model ও task-এর উপর নির্ভর করে। Similarity মানেই factual equivalence নয়।

## ৫. Transformer কী?

Transformer architecture attention mechanism ব্যবহার করে sequence-এর বিভিন্ন token-এর সম্পর্ক model করে। Common architecture:
- Encoder-only: representation/understanding-oriented task-এ ব্যবহৃত।
- Decoder-only: autoregressive generation-এ ব্যবহৃত।
- Encoder-decoder: input sequence encode করে output sequence generate করে।

~~~mermaid
flowchart TD
    A["Transformer families"] --> B["Encoder-only"]
    A --> C["Decoder-only"]
    A --> D["Encoder-decoder"]
    B --> E["Representations / classification"]
    C --> F["Autoregressive generation"]
    D --> G["Sequence-to-sequence tasks"]
~~~

অনেক modern generative LLM decoder-only Transformer ভিত্তিক, তবে সব LLM architecture এক নয়।

## ৬. Self-Attention কী?

Self-attention-এ প্রতিটি token sequence-এর অন্য token-গুলোর information weight করে combine করতে পারে। Query, Key, Value projection দিয়ে attention score ও weighted value তৈরি হয়।

~~~text
Attention(Q, K, V) = softmax(QK^T / sqrt(d_k)) V
~~~

- Query: current token কী ধরনের information খুঁজছে।
- Key: প্রতিটি token কী information represent করছে।
- Value: সেই token থেকে কোন content combine হবে।

~~~mermaid
flowchart LR
    A["Token representations"] --> B["Query projection"]
    A --> C["Key projection"]
    A --> D["Value projection"]
    B --> E["Attention scores: Q × K"]
    C --> E
    E --> F["Softmax weights"]
    F --> G["Weighted combination of V"]
    D --> G
~~~

## ৭. Multi-Head Attention কেন?

একাধিক attention head parallelভাবে ভিন্ন relation বা representation pattern capture করতে পারে। Head output combine করে পরবর্তী layer-এ পাঠানো হয়। এর অর্থ প্রতিটি head সবসময় একটি নির্দিষ্ট human-readable relation শিখেছে, এমন নয়।

## ৮. Causal Masking কী?

Autoregressive decoder-এ token position t যেন future token t+1, t+2 দেখতে না পারে, তাই causal mask ব্যবহার হয়। এতে next-token prediction-এর training objective বজায় থাকে।

~~~mermaid
flowchart TD
    A["Token 1"] --> B["Can attend to token 1"]
    C["Token 2"] --> D["Can attend to tokens 1-2"]
    E["Token 3"] --> F["Can attend to tokens 1-3"]
    G["Future tokens"] --> H["Masked from earlier positions"]
~~~

## ৯. Positional Information কেন দরকার?

Self-attention নিজে token order জানে না। তাই positional encoding/embedding বা RoPE-এর মতো positional mechanism দিয়ে sequence order information যোগ করা হয়।

Token sequence "dog bites man" ও "man bites dog" একই শব্দ ব্যবহার করলেও order বদলালে meaning বদলায়। Positional information এই order model করতে সাহায্য করে।

## ১০. Next-token Prediction কী?

Autoregressive language model আগের token দেখে পরবর্তী token-এর probability estimate করে। Generation চলতে থাকে EOS বা stopping condition না আসা পর্যন্ত।

~~~mermaid
flowchart LR
    A["The cat"] --> B["Predict next token"]
    B --> C["sat"]
    C --> D["Append to context"]
    D --> E["Predict next token"]
    E --> F["on"]
~~~

## ১১. Pretraining, Supervised Fine-tuning এবং Preference Alignment কী?

একটি সাধারণ LLM training pipeline-এ বিভিন্ন stage থাকতে পারে:

- **Pretraining:** বড় corpus থেকে next-token prediction-এর মতো objective শেখা।
- **Supervised fine-tuning (SFT):** instruction/response example দিয়ে desired behavior শেখানো।
- **Preference alignment:** human বা synthetic preference signal ব্যবহার করে response behavior tune করা; RLHF বা DPO এর উদাহরণ।

~~~mermaid
flowchart TD
    A["Large text corpus"] --> B["Pretraining"]
    B --> C["Base model"]
    C --> D["Supervised fine-tuning"]
    D --> E["Instruction-following model"]
    E --> F["Preference alignment"]
    F --> G["Aligned assistant model"]
~~~

সব model একই pipeline ব্যবহার করে না। SFT/RLHF/DPO-এর objective ও optimization আলাদা।

## ১২. Context Window কী?

একটি request-এ model কত token context process করতে পারে তার সীমা। Input prompt, conversation history, system/developer instruction, retrieved documents এবং output budget সবই context ব্যবহার করে।

~~~mermaid
flowchart TD
    A["Context budget"] --> B["System / developer instructions"]
    A --> C["Conversation history"]
    A --> D["Retrieved context"]
    A --> E["Current user input"]
    A --> F["Reserved output tokens"]
~~~

Context window বড় হলেই model সব token সমানভাবে ভালোভাবে ব্যবহার করবে এমন নয়। Irrelevant context quality ও cost কমাতে পারে।

## ১৩. Temperature এবং Top-p কী?

- **Temperature:** logits/probability distribution-এর sharpness নিয়ন্ত্রণ করে। কম temperature সাধারণত বেশি predictable output দেয়; বেশি temperature diversity বাড়াতে পারে।
- **Top-p:** cumulative probability mass অনুযায়ী candidate token set সীমিত করে।

এগুলো output guarantee করে না; provider ও model অনুযায়ী behavior ভিন্ন হতে পারে। Sampling parameter task অনুযায়ী evaluate করো।

## ১৪. Hallucination কী?

LLM fluent কিন্তু ভুল, fabricated বা unsupported তথ্য তৈরি করলে তাকে hallucination বলা হয়। Model language pattern generate করে; এটি guaranteed fact lookup system নয়।

~~~mermaid
flowchart TD
    A["User asks factual question"] --> B["LLM generates likely response"]
    B --> C{"Evidence grounded?"}
    C -->|না| D["Unsupported / hallucinated claim risk"]
    C -->|হ্যাঁ| E["Verify against trusted source"]
    D --> F["Retrieval, verification, abstention"]
~~~

Mitigation:
- Trusted retrieval ও citations।
- Constrained output ও schema validation।
- Verification against source data।
- Uncertainty/abstention policy।
- Representative evaluation।

RAG hallucination কমাতে পারে, কিন্তু ভুল retrieval বা unsupported synthesis পুরোপুরি বন্ধ করে না।

## ১৫. Prompting, RAG এবং Fine-tuning-এর পার্থক্য কী?

| Technique | কী বদলায় | ভালো use case |
|---|---|---|
| Prompting | Instruction ও context | Task framing, format, behavior |
| RAG | Inference-এ external context যোগ | Fresh/private knowledge |
| Fine-tuning | Model weights | Style/task behavior adaptation |

~~~mermaid
flowchart TD
    A["Need to improve LLM application"] --> B["Instruction unclear?"]
    B -->|হ্যাঁ| C["Improve prompt"]
    B -->|না| D["Knowledge missing / changing?"]
    D -->|হ্যাঁ| E["Consider RAG"]
    D -->|না| F["Task behavior consistently wrong?"]
    F -->|হ্যাঁ| G["Evaluate fine-tuning"]
~~~

Fresh private knowledge-এর জন্য RAG প্রায়ই বেশি practical; style/task consistency-এর জন্য fine-tuning বিবেচ্য। Fine-tuning-কে current facts update করার primary mechanism ধরে নিও না।

## ১৬. KV Cache কী?

Autoregressive generation-এ আগের token-এর Key ও Value projection cache করে রাখলে প্রতিটি নতুন token-এর জন্য পুরো previous sequence-এর K/V আবার compute করতে হয় না। এটি inference latency কমাতে সাহায্য করে, তবে memory ব্যবহার করে।

## ১৭. Quantization কী?

Model weight কম bit precision-এ represent করে memory footprint ও inference cost কমানোর পদ্ধতি। Quantization speed/memory উন্নত করতে পারে, কিন্তু quality/accuracy trade-off থাকতে পারে। Model, hardware এবং quantization method অনুযায়ী ফল বদলায়।

## ১৮. LLM Evaluation-এ কী মাপবে?

একটি single score যথেষ্ট নয়। Task অনুযায়ী:
- Correctness / factuality।
- Instruction following।
- Structured output validity।
- Groundedness ও citation correctness।
- Safety/refusal behavior।
- Latency ও token cost।
- Robustness to adversarial input।

## Quick Interview Revision

| প্রশ্ন | সংক্ষিপ্ত উত্তর |
|---|---|
| LLM কী predict করে? | Context থেকে next-token distribution |
| Token কী? | Model-এর text unit |
| Embedding? | Dense vector representation |
| Transformer? | Attention-based sequence architecture |
| Q/K/V? | Query, Key, Value projections |
| Causal mask? | Future token access block |
| Positional info? | Token order represent করে |
| Context window? | Request-এর token capacity |
| Temperature? | Sampling distribution sharpness |
| Hallucination? | Fluent but unsupported/false output |
| RAG? | Retrieved external context দিয়ে generation ground করা |
| Fine-tuning? | Model weights update |
| KV cache? | Previous K/V reuse করে decoding speed up |
| Quantization? | Lower precision representation |

## Practice

1. একটি paragraph tokenizer দিয়ে token count করো।
2. Token count ও word count compare করো।
3. Context window budget ভাগ করার example তৈরি করো।
4. একই prompt-এ temperature পরিবর্তন করে output variation observe করো।
5. Hallucination-এর তিনটি কারণ ও mitigation লিখো।
6. Prompting, RAG এবং fine-tuning-এর জন্য তিনটি আলাদা use case দাও।
7. Causal masking কেন autoregressive training-এ দরকার explain করো।
8. KV cache ও quantization latency/memory trade-off ব্যাখ্যা করো।
