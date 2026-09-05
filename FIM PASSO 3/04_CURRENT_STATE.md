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

## Web Search / Browser (Fase 03) — CONCLUÍDA

- [x] SearXNG rodando via Docker, cliente de busca (`searxng_client.py`)
- [x] Fetch + extração de conteúdo via Crawl4AI (`page_fetcher.py`) — substitui Playwright puro + lib de extração separada do plano original
- [x] Pipeline de evidências: dedup por hash escopado a `chat_id` + chunking reaproveitado + inserção no `chat_scope`
- [x] Multi-query via GPT-OSS (geração de variações + dedup de resultados por URL)
- [x] Teste de integração end-to-end cobrindo o pipeline completo

Pendente para incremento futuro, fora do escopo mínimo da Fase 03: navegação interativa multi-página (clicar, preencher formulário) — o que foi implementado é fetch de página única por URL candidata, suficiente para o objetivo de coleta de evidência. Automação de navegação mais complexa fica para quando houver necessidade real (regra 1 do projeto).

## Arquitetura final

- [ ] Open WebUI
- [ ] PostgreSQL
- [x] Qdrant final configurado
- [x] Hybrid retrieval
- [x] Reranker
- [x] Web Search
- [x] Playwright (via Crawl4AI, Decision 026)
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
