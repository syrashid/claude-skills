# Concept ledger — machine learning

Cross-repo curriculum of transferable ML/AI concepts: retrieval & RAG design, chunking/embedding tradeoffs, model training & evaluation, prompt- and LLM-system engineering. Unchecked = still in retest rotation. See "The concept ledger" in SKILL.md for the rules.

- [ ] Recursive vs. semantic chunking — structure-aware recursive splitting at ~512 tokens reaches 85–90% retrieval recall vs. semantic chunking's 91–92% at ~14× compute cost, and stays deterministic across embedding-model versions; semantic chunking's boundaries shift with the model, which also breaks constraint-based idempotency (introduced 2026-07-11, BookLM chunking rung / NAACL 2025 benchmark)
- [ ] Tokenizer alignment across pipeline stages — a chunker's token budget is denominated in *its* tokenizer's units; if the embedding model tokenizes differently, "512-token chunks" can overflow the embedder's effective window, so pin chunk size to the downstream model's tokenizer (introduced 2026-07-11, chonkie CHUNK_SIZE watch-point)
