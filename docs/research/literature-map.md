# Literature Map: LLM Memory, RAG, Agents, and Context Compilers

This document maps research ideas that should influence `llm-memory-mcp`. It is not an exhaustive literature review; it is a design-oriented guide for building a practical memory MCP.

## Reading lens

For this project, the key question is:

> What patterns help an LLM system preserve, retrieve, update, and summarize durable knowledge across many sessions and clients?

The most relevant threads are:

1. Retrieval-augmented generation.
2. Long-term memory for agents.
3. Hierarchical/global summarization.
4. Reflection and self-critique.
5. Graph-structured memory.
6. External context/memory operating systems.
7. Evaluation of faithfulness and retrieval quality.

---

## 1. Foundation: Retrieval-Augmented Generation

### Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks

- Authors: Patrick Lewis, Ethan Perez, Aleksandra Piktus, Fabio Petroni, et al.
- Year: 2020
- arXiv: https://arxiv.org/abs/2005.11401

Core idea:

- Combine parametric model memory with non-parametric retrieved documents.
- Improves factuality and knowledge-intensive QA.
- Provides a path to update knowledge without retraining model weights.

Implication for `llm-memory-mcp`:

- External memory is necessary because model weights are not enough.
- But plain RAG is only one layer; our project should not stop at chunk retrieval.
- Provenance and updateability are central requirements.

---

## 2. Reasoning + acting with tools

### ReAct: Synergizing Reasoning and Acting in Language Models

- Authors: Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, et al.
- Year: 2022
- arXiv: https://arxiv.org/abs/2210.03629

Core idea:

- Interleave reasoning traces with actions against tools/environments.
- Tool use reduces hallucination and improves interpretability.

Implication:

- MCP tools should be designed as composable actions:
  - search;
  - read;
  - ingest;
  - generate report;
  - refresh summary.
- Tools should return interpretable evidence, not just final answers.

---

## 3. Reflective/episodic agent memory

### Reflexion: Language Agents with Verbal Reinforcement Learning

- Authors: Noah Shinn, Federico Cassano, Edward Berman, Ashwin Gopinath, et al.
- Year: 2023
- arXiv: https://arxiv.org/abs/2303.11366

Core idea:

- Agents improve by storing verbal reflections from feedback.
- No weight updates required; improvement comes from durable text memory.

Implication:

- Valuable query results and lessons should become durable Markdown.
- Reflection should be explicit and reviewable, not hidden in a model's latent state.
- `reports/topics/` and `wiki/` can serve as reflection storage.

### Generative Agents: Interactive Simulacra of Human Behavior

- Authors: Joon Sung Park, Joseph C. O'Brien, Carrie J. Cai, Meredith Ringel Morris, et al.
- Year: 2023
- arXiv: https://arxiv.org/abs/2304.03442

Core idea:

- Agents store observations, retrieve relevant memories using recency/importance/relevance, and periodically reflect into higher-level summaries.

Implication:

- Ranking memory only by vector similarity is insufficient.
- Retrieval should combine:
  - relevance;
  - recency;
  - importance;
  - source type;
  - links/backlinks.
- Periodic reflection maps naturally to `summary-context.md` refresh and topic reports.

### MemoryBank: Enhancing Large Language Models with Long-Term Memory

- Authors: Wanjun Zhong, Lianghong Guo, Qiqi Gao, He Ye, et al.
- Year: 2023
- arXiv: https://arxiv.org/abs/2305.10250

Core idea:

- Long-term memory for sustained interactions.
- Includes memory updating, user personality modeling, and forgetting/reinforcement inspired by Ebbinghaus.

Implication:

- The system should track importance/recency for retrieval ranking.
- Forgetting should be retrieval decay, not destructive deletion.
- Durable evidence should remain inspectable.

---

## 4. Memory hierarchy and virtual context management

### MemGPT: Towards LLMs as Operating Systems

- Authors: Charles Packer, Sarah Wooders, Kevin Lin, Vivian Fang, et al.
- Year: 2023
- arXiv: https://arxiv.org/abs/2310.08560

Core idea:

- Treat limited context window like limited RAM.
- Manage memory tiers using OS-inspired virtual context management.
- Move information between short context and long-term storage.

Implication:

- `llm-memory-mcp` should have explicit memory tiers:
  - current context pack;
  - root summary;
  - topic reports;
  - extracted source derivatives;
  - raw source files;
  - indexes.
- `build_context_pack` is equivalent to loading the right pages into working memory.
- Do not expect one context window to hold everything.

### LongMem: Augmenting Language Models with Long-Term Memory

- Authors: Weizhi Wang, Li Dong, Hao Cheng, Xiaodong Liu, et al.
- Year: 2023
- arXiv: https://arxiv.org/abs/2306.07174

Core idea:

- Decoupled memory design with cached long-form memory and retriever/reader.

Implication:

- Even if we are not training a new model, the architecture argues for decoupling durable memory storage from the model call.
- Indexes should be replaceable and rebuildable.

---

## 5. Self-reflective and adaptive retrieval

### Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection

- Authors: Akari Asai, Zeqiu Wu, Yizhong Wang, Avirup Sil, et al.
- Year: 2023
- arXiv: https://arxiv.org/abs/2310.11511

Core idea:

- Retrieve only when needed.
- Critique retrieved passages and generated responses.
- Improves factuality and citation accuracy.

Implication:

- MCP should support adaptive retrieval rather than fixed-k chunk dumping.
- Tools should support critique/evaluation metadata:
  - relevance;
  - support/contradiction;
  - cited sources;
  - confidence.
- `generate_topic_report` should include uncertainty and evidence quality.

---

## 6. Hierarchical summarization and global questions

### RAPTOR: Recursive Abstractive Processing for Tree-Organized Retrieval

- Authors: commonly cited as Sarthi et al.
- Year: 2024
- arXiv: https://arxiv.org/abs/2401.18059

Core idea:

- Build a tree of summaries over text chunks.
- Retrieval can happen at different abstraction levels.

Implication:

- `summary-context.md` should be the top layer of a hierarchy, not a one-shot summary.
- Extracted source summaries and topic reports are useful intermediate nodes.

### From Local to Global: A Graph RAG Approach to Query-Focused Summarization

- Authors: Darren Edge, Ha Trinh, Newman Cheng, Joshua Bradley, et al.
- Year: 2024
- arXiv: https://arxiv.org/abs/2404.16130

Core idea:

- Standard RAG struggles with global questions over a corpus.
- GraphRAG builds an entity graph and community summaries, then synthesizes global answers.

Implication:

- `llm-memory-mcp` should support both local and global queries:
  - local: "what did this audio say?"
  - global: "what do we know overall about this medical issue?"
- Topic reports and `summary-context.md` are the durable equivalent of community/global summaries.
- Full knowledge graph can wait, but wikilinks and source/topic relationships should be captured early.

### LightRAG: Simple and Fast Retrieval-Augmented Generation

- Year: 2024
- arXiv commonly referenced: https://arxiv.org/abs/2410.05779

Core idea:

- Lightweight graph-enhanced retrieval with efficient indexing.

Implication:

- A small graph layer over Markdown links and source-topic edges may give much of the benefit without a heavy graph database.
- Do not add Neo4j in MVP.

---

## 7. Long-term memory as a missing piece for agents

### Position: Episodic Memory is the Missing Piece for Long-Term LLM Agents

- Year: 2025
- arXiv search result: https://arxiv.org/abs/2502.06975

Core idea:

- Long-lived agents need episodic memory: timestamped, experience-like records.

Implication:

- `sources/` + `extracted/` + `log.md` are not optional implementation details; they are the episodic substrate.
- `summary-context.md` should not erase event order.
- Timeline sections are important.

### Externalization in LLM Agents: A Unified Review of Memory, Skills, Protocols and Harness Engineering

- Year: 2026
- arXiv search result: https://arxiv.org/abs/2604.08224

Core idea:

- Modern agents externalize memory, skills, protocols, tools, and harnesses.

Implication:

- MCP is a good architectural boundary: memory should be external, tool-accessible, and reusable across clients.
- The repo should include not just code but protocols/templates/prompts that define how clients update memory.

---

## 8. Practical systems and prior art

### Basic Memory

- GitHub: https://github.com/basicmachines-co/basic-memory
- License: AGPL-3.0-or-later

Useful ideas:

- Markdown local-first.
- MCP-native.
- Search/read/write/build-context tools.
- Project routing.
- Wikilinks and graph-style navigation.
- Optional semantic search.

Caution:

- Avoid copying code due AGPL.

### Karpathy LLM Wiki pattern

- Gist: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f

Useful ideas:

- Build a compounding Markdown wiki instead of repeatedly doing stateless retrieval.
- Convert repeated queries into durable pages.
- Maintain links and summaries so future queries become cheaper.

### LLM wiki/compiler projects

Relevant examples:

- `lucasastorian/llmwiki`
- `atomicstrata/llm-wiki-compiler`
- Obsidian MCPs and vault tools

Implication:

- There is prior validation for Markdown + MCP + compiled wiki.
- Our differentiation is the explicit context-compiler structure with root `summary-context.md`, extracted source derivatives, and topic reports.

---

## 9. Synthesis: what this means for `llm-memory-mcp`

### Do

- Make context folders portable.
- Store all hard-source derivatives in `extracted/`.
- Keep one canonical `summary-context.md` per context.
- Treat generated topic reports as durable memory.
- Use search as evidence selection, not as the whole product.
- Add staleness and provenance from day one.
- Implement `build_context_pack` as the key retrieval abstraction.
- Use hierarchical summarization.
- Use wikilinks/source links before heavy graph infrastructure.
- Evaluate answer faithfulness and provenance.

### Avoid

- Dumping a fixed top-k retrieval result into every answer.
- Making vectors the source of truth.
- Hiding generated memory only in a DB.
- Creating multiple competing summaries.
- Letting a cheap model silently rewrite canonical context.
- Copying AGPL Basic Memory code.
- Adding a heavy graph/vector stack before the Markdown/FTS workflow is proven.

---

## 10. Reference list

- Lewis et al. 2020 — Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks — https://arxiv.org/abs/2005.11401
- Yao et al. 2022 — ReAct: Synergizing Reasoning and Acting in Language Models — https://arxiv.org/abs/2210.03629
- Shinn et al. 2023 — Reflexion: Language Agents with Verbal Reinforcement Learning — https://arxiv.org/abs/2303.11366
- Park et al. 2023 — Generative Agents: Interactive Simulacra of Human Behavior — https://arxiv.org/abs/2304.03442
- Zhong et al. 2023 — MemoryBank: Enhancing Large Language Models with Long-Term Memory — https://arxiv.org/abs/2305.10250
- Wang et al. 2023 — Augmenting Language Models with Long-Term Memory / LongMem — https://arxiv.org/abs/2306.07174
- Packer et al. 2023 — MemGPT: Towards LLMs as Operating Systems — https://arxiv.org/abs/2310.08560
- Asai et al. 2023 — Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection — https://arxiv.org/abs/2310.11511
- RAPTOR — Recursive Abstractive Processing for Tree-Organized Retrieval — https://arxiv.org/abs/2401.18059
- Edge et al. 2024 — From Local to Global: A Graph RAG Approach to Query-Focused Summarization — https://arxiv.org/abs/2404.16130
- HippoRAG — Neurobiologically Inspired Long-Term Memory for Large Language Models — https://arxiv.org/abs/2405.14831
- LightRAG — Simple and Fast Retrieval-Augmented Generation — https://arxiv.org/abs/2410.05779
- Basic Memory — https://github.com/basicmachines-co/basic-memory
- Karpathy LLM Wiki pattern — https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
