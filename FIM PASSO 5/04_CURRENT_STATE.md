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

## Coding Agent / Sandbox (Fase 04) — CONCLUÍDA

- [x] Sandbox de execução via Docker (`sandbox/Dockerfile`): `python:3.11-slim` + numpy/pandas/requests/beautifulsoup4/matplotlib/scipy pré-instalados
- [x] Container efêmero por execução (`docker run --rm`), sem rede (`--network none`), limites de memória/CPU/pids, `--cap-drop ALL` + `--security-opt no-new-privileges` (`src/sandbox/executor.py`)
- [x] Tool Qwen3-Coder: geração e correção de código via Ollama (`src/coding/coder_client.py`), tag `qwen3-coder:30b` confirmada empiricamente
- [x] Loop de iteração automática com auto-correção baseada no erro real de execução, limite de tentativas configurável (`src/coding/agent_loop.py`, `DEFAULT_MAX_ATTEMPTS=5`)
- [x] Teste de integração end-to-end cobrindo o pipeline completo (`test_coding_agent_e2e.py`)

Pendente para incremento futuro, fora do escopo mínimo da Fase 04:
- O sandbox ainda não permite escrita de arquivos de saída (ex: gráficos do `matplotlib`) sem que o próprio código gerado contorne isso (ex: usando `tempfile`, que escreve em `/tmp` dentro do container). Isso acontece porque o diretório montado perde a permissão de escrita para o processo root do container sem `CAP_DAC_OVERRIDE` (ver Decision 032). Ajustar apenas se surgir necessidade real de recuperar arquivos gerados pelo sandbox (regra 1 do projeto).
- Não existe ainda um Model Router formal decidindo quando acionar o Qwen3-Coder — hoje ele é chamado diretamente pelos scripts/loop da Fase 04. Fica para a Fase 07 (Agent Core).

## Memory (Fase 05) — CONCLUÍDA

- [x] Schema de metadata de memória em `global_scope` via `memory_type` (`knowledge`/`research`/`manual`), inteiramente no payload do Qdrant, sem PostgreSQL (`src/memory/schema.py`)
- [x] Mecanismo `/save`: promoção explícita `chat_id` + IDs de `chat_scope` para `global_scope`, vetor reaproveitado (sem reembedding), cópia — não move (`src/memory/save.py` + `scripts/save_memory.py`)
- [x] Nota manual avulsa: texto via `--text`, `memory_type="manual"` (`src/memory/add_note.py` + `scripts/add_memory.py`)
- [x] Visualizar/editar/apagar memórias (regra 10 do projeto): listar com filtros, editar tags (mesmo ID) ou texto (ID novo, reembedding), apagar só por ID explícito com confirmação (`src/memory/manage.py` + `scripts/list_memories.py`, `edit_memory.py`, `delete_memory.py`)
- [x] Teste de integração end-to-end cobrindo o pipeline completo da fase (`test_memory_e2e.py`)

Pendente para incremento futuro, fora do escopo mínimo da Fase 05:
- "Memória de conversa" fica para quando existir Agent Core / loop de chat real (Fase 07) — decisão explícita de escopo (Decision 042), não existe hoje um loop de conversa fora de scripts CLI isolados.
- Interface continua CLI — a UI Gradio/Streamlit prevista na Decision 019 não foi construída nesta fase.
- Sem índice de payload dedicado para `tags` em `global_scope` — filtro por tag funciona via full scan do Qdrant, aceitável no volume atual; criar o índice fica como ajuste futuro se a listagem ficar lenta com uso real (regra 5 do projeto).
- Apagar/editar em massa por filtro não existe — só por ID explícito (decisão deliberada de segurança, Decision 040).

## Arquitetura final

- [ ] Open WebUI
- [ ] PostgreSQL
- [x] Qdrant final configurado
- [x] Hybrid retrieval
- [x] Reranker
- [x] Web Search
- [x] Playwright (via Crawl4AI, Decision 026)
- [ ] Adaptive Crawler
- [x] Memory System
- [ ] Agent Core
- [ ] Model Router
- [ ] Vision System
- [x] Coding Sandbox
- [ ] Verifier
- [ ] Observability
- [ ] Continuous Research
- [ ] Final benchmark
