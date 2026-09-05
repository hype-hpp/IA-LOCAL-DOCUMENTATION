# IA Local — Roadmap

## Fase 01 — Fundação + Benchmark

Objetivo:
- validar ambiente
- medir modelos
- escolher estratégia de routing

Status: CONCLUIDO

---

## Fase 02 — RAG / Knowledge

Objetivo:
- Implementar separação física e lógica entre escopo do chat (temporário) e escopo global (persistente) no Qdrant e Filesystem
- Qdrant
- embeddings
- parser
- chunking
- metadata
- hybrid search
- reranking

Entregue:
- Qdrant rodando via Docker, collections `chat_scope`/`global_scope` (Decision 022)
- Embeddings via Qwen3-Embedding-4B/Ollama, dimensão 2560 confirmada empiricamente
- Parser (.md/.txt) + chunking por palavras + dedup por hash (Decision 023)
- Hybrid search: dense (Qdrant) + sparse (BM25 em memória) via RRF (Decision 024)
- Reranker via GPT-OSS, sem modelo dedicado (Decision 025)

Status: CONCLUIDO

---

## Fase 03 — Web Search + Browser

Objetivo:
- SearXNG
- Playwright
- pesquisa multi-query
- navegação
- coleta de evidências

Entregue:
- SearXNG rodando via Docker, cliente de busca (`searxng_client.py`) — 3.1
- Fetch + extração de conteúdo via Crawl4AI, que substituiu Playwright puro + lib de extração separada do plano original (`page_fetcher.py`, Decision 026) — 3.2
- Pipeline de evidências: dedup por hash escopado a `chat_id` + chunking reaproveitado da Fase 02 + inserção no `chat_scope` (`evidence.py`, `web_research.py`, Decision 027) — 3.3
- Multi-query via GPT-OSS: geração de variações da query + dedup de resultados por URL (`query_expansion.py`, `multi_query.py`, Decision 028) — 3.4
- Teste de integração end-to-end cobrindo o pipeline completo (`test_web_research_e2e.py`) — 3.5

Status: CONCLUIDO

---

## Fase 04 — Coding Agent + Sandbox

Objetivo:
- Qwen3-Coder
- Docker sandbox
- execução
- testes
- debugging
- iteração automática

Entregue:
- Sandbox de execução via Docker: imagem `python:3.11-slim` com numpy/pandas/requests/beautifulsoup4/matplotlib/scipy pré-instalados, container **efêmero** (`docker run --rm`) por execução, **sem rede** (`--network none`), limites de memória/CPU/pids e hardening de capabilities (`--cap-drop ALL`, `--security-opt no-new-privileges`) (`sandbox/Dockerfile`, `src/sandbox/executor.py`, Decision 031) — 4.1
- Correção de um bug real de permissão (`--cap-drop ALL` removendo `CAP_DAC_OVERRIDE`) descoberto em teste no hardware (Decision 032) — 4.1
- Tool Qwen3-Coder: geração e correção de código via Ollama, mesma função (`generate_code()`) cobrindo os dois casos, sem grammar-constrained JSON Schema (`src/coding/coder_client.py`, Decision 033) — 4.2
- Loop de iteração automática: executa no sandbox → em caso de erro, corrige via Qwen3-Coder com o erro real como contexto → executa de novo, até um limite de tentativas configurável (`src/coding/agent_loop.py`, Decisions 034 e 035) — 4.3
- Teste de integração end-to-end cobrindo o pipeline completo (task → Qwen3-Coder → sandbox → auto-correção) com Ollama e Docker reais (`tests/test_coding_agent_e2e.py`) — 4.4

Status: CONCLUIDO

---

## Fase 05 — Memory

Objetivo:
- Criar mecanismo de promoção manual (/save) para transferir conhecimento do escopo do chat para o escopo global
- memória de conversa
- memória episódica
- memória semântica
- memória de pesquisa
- UI para visualizar/editar/apagar

Status: PENDENTE

---

## Fase 06 — Adaptive Crawler

Objetivo:
- crawler orientado pelo agente
- depth adaptativo
- page budget
- topic filtering
- exclusions
- deduplicação
- hash de conteúdo
- atualização incremental
- limite de armazenamento

Status: PENDENTE

---

## Fase 07 — Agent Core

Objetivo:
- planner
- router
- tool manager
- research controller
- memory manager
- verifier (implementação universal/sempre ativa, rodando modelo 7B na CPU)

Status: PENDENTE

---

## Fase 08 — UI + Integração

Objetivo:
- Open WebUI [
    ignorar inicialmente para a fase de desenvolvimento - decision 019
]
- integração dos serviços
- observabilidade
- status de jobs
- fontes
- memória
- ferramentas

Status: PENDENTE

---

## Fase 09 — Benchmark Final

Objetivo:
comparar:

- modelo puro
- RAG
- web research
- agent
- verifier
- sistema completo

Status: PENDENTE

---

## Fase 10 — Otimização

Objetivo:
- offload
- quantização
- contexto
- cache
- throughput
- armazenamento
- concorrência

Status: PENDENTE
