# AIET — TSpec-LLM 3GPP RAG — Class Hub

Running notes for the class series: dates, topics discussed, and links shared.
Only things actually covered or handed out in class go here.

**Instructors:** Shreesh & Arjit
**Series started:** 2026-04-20
**Stack:** Qdrant + vLLM + BGE-large + bge-reranker-v2-m3 + gemma-4-E4B-it
**Corpus:** TSpec-LLM (3GPP, ~15,420 markdown files)

---

## Shared pipeline template

_Discussed in Class 3. Kept at the top because every class refers back to this._

Directory layout — what each file is responsible for:

```text
rag/
├── __init__.py                 # package marker (empty)
├── config.py                   # all paths, model IDs, tunables — single source of truth
│
├── ingest/                     # corpus → JSONL → Qdrant (offline, run once)
│   ├── __init__.py
│   ├── chunker.py              # walk *.md corpus, split into clauses + 512-tok chunks,
│   │                             emit chunks.jsonl + clauses.jsonl with metadata
│   ├── glossary.py             # parse TR 21.905 into {acronym → expansion},
│   │                             emit glossary.json for query-time expansion
│   └── indexer.py              # stream chunks.jsonl, embed with BGE,
│                                 build BM25 sparse vectors, upsert to Qdrant
│
├── retrieve/                   # online retrieval service (one FastAPI process)
│   ├── __init__.py
│   ├── service.py              # /retrieve endpoint: classify → dense+sparse → RRF →
│   │                             rerank → parent-doc expand → cross-ref expand
│   └── client.py               # thin sync HTTP client for /retrieve + /healthz
│
├── serve/                      # infra smoke tests (run before demo)
│   ├── smoke_qdrant.py         # verify Qdrant is up, collection exists, returns results
│   ├── smoke_vllm.py           # verify vLLM endpoint responds, streams tokens
│   └── README-serve.md         # how to boot Qdrant + vLLM containers
│
├── chat/                       # user-facing REPL
│   ├── __init__.py
│   ├── agent.py                # tool-calling loop: LLM emits <tool_call>retrieve</tool_call>,
│   │                             hop up to 3x, then cite
│   ├── tui.py                  # terminal UI wrapping agent.py
│   └── README.md               # usage
│
├── eval/                       # offline metrics
│   ├── __init__.py
│   ├── eval_qsmall.py          # run Q-small benchmark, compute recall@k / MRR / citation-F1
│   └── orchestrator.py         # single-question end-to-end runner
│
└── scripts/                    # one-off ops helpers
    └── audit_unknown.py        # scan Qdrant for points with missing/unknown spec metadata
```

Data flow:

```text
corpus/*.md  →  chunker.py  →  chunks.jsonl + clauses.jsonl
                                     ↓
                               indexer.py  →  Qdrant collection
                                                     ↑
user question  →  tui.py  →  agent.py  →  client.py  →  service.py  →  (dense + sparse + rerank)
                              ↓
                          vLLM (gemma-4-E4B)  →  cited answer
```

---

## Class 1 — 2026-04-20

**Topic:** Project intro — what RAG is, why we need it for 3GPP, high-level system walkthrough.

**Links shared:**
- Docling (document parsing, IBM): https://github.com/docling-project/docling
- LiteParse (lightweight document parser, LlamaIndex): https://github.com/run-llama/liteparse

---

## Class 2 — 2026-04-21

**Topic:** Word2Vec and embedding intuition — how words become vectors, cosine similarity, bridge to sentence embedders (BGE).

**Links shared:**
- Word2Vec Explained (Israel Grubman): https://israelg99.github.io/2017-03-23-Word2Vec-Explained/
- Colab notebook: https://colab.research.google.com/drive/1aes3A6AumwokaSmdHL4F477Eb-PBdWLD?usp=sharing

---

## Class 3 — 2026-04-22

**Topic:** Architecture deep dive — what each file in `rag/` does, ingest → retrieve → generate at code level.

**Shared in class:** the directory template above (see "Shared pipeline template" section).

**Links shared:**
- TSpec-LLM dataset (Rasoul Nikbakht, HuggingFace): https://huggingface.co/datasets/rasoul-nikbakht/TSpec-LLM/tree/main

---

## Class 4 onwards — TBD

Add a new section per class as it happens. Format:

```text
## Class N — YYYY-MM-DD

**Topic:** <one-line summary of what was discussed>

**Links shared:**
- <url 1>
- <url 2>
```
