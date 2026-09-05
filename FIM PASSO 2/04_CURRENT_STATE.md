# IA Local — Current State

## Infraestrutura

- [x] Arch Linux
- [x] Ollama instalado
- [x] Python virtual environment `~/crawler-ai`

## Benchmark de modelos

- [x] Benchmark de velocidade realizado
- [x] Benchmark de qualidade concluído

## RAG / Knowledge (Fase 02) — CONCLUÍDA

- [x] Estrutura `chats/` (temporário) vs `knowledge/` (persistente)
- [x] Qdrant rodando via Docker, collections `chat_scope`/`global_scope`
- [x] Pipeline de embedding (Qwen3-Embedding-4B via Ollama), dimensão 2560 confirmada
- [x] Parser (.md/.txt)
- [x] Chunking por palavras (250/overlap 40) + dedup por hash
- [x] Hybrid search (dense + BM25 em memória, fusão via RRF)
- [x] Reranker (via GPT-OSS, sem modelo dedicado)

Pendente para incremento futuro, fora do escopo mínimo da Fase 02: suporte a PDF no parser.

## Arquitetura final

- [ ] Open WebUI
- [ ] PostgreSQL
- [x] Qdrant final configurado
- [x] Hybrid retrieval
- [x] Reranker
- [ ] Web Search
- [ ] Playwright
- [ ] Adaptive Crawler
- [ ] Memory System
- [ ] Agent Core
- [ ] Model Router
- [ ] Vision System
- [ ] Coding Sandbox
- [ ] Verifier
- [ ] Observability
- [ ] Continuous Research
- [ ] Final benchmark
