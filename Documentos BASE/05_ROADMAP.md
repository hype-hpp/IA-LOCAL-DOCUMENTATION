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

Status: PENDENTE

---

## Fase 03 — Web Search + Browser

Objetivo:
- SearXNG
- Playwright
- pesquisa multi-query
- navegação
- coleta de evidências

Status: PENDENTE

---

## Fase 04 — Coding Agent + Sandbox

Objetivo:
- Qwen3-Coder
- Docker sandbox
- execução
- testes
- debugging
- iteração automática

Status: PENDENTE

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
