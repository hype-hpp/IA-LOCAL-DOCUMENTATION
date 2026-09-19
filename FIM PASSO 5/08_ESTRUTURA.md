# IA Local — Estrutura de Pastas

Referência da estrutura real do projeto em `~/IA-LOCAL/`, como ficou ao final da Fase 05.

```
IA-LOCAL/
│
├── docker-compose.yml       # sobe os serviços de infraestrutura (Qdrant, SearXNG)
├── requirements.txt          # dependências Python do projeto
├── README.md                  # referência rápida: estrutura + comandos de setup
├── .gitignore                  # gerado por scripts/setup_dirs.sh, expandido na Fase 03
│
├── docs/
│   ├── STATUS.md                # progresso por passo/fase, tabela curta
│   └── TUTORIAL.md              # tutorial da ÚLTIMA entrega de arquivos (é substituído a cada passo, não acumula histórico)
│
├── sandbox/                    # Fase 04
│   └── Dockerfile                 # imagem do sandbox: python:3.11-slim + numpy/pandas/requests/beautifulsoup4/matplotlib/scipy
│
├── scripts/                    # ferramentas administrativas — rodar manualmente, sob demanda
│   ├── setup_dirs.sh              # cria chats/, knowledge/, data/
│   ├── create_collections.py      # cria as collections do Qdrant (idempotente) + índices de payload (inclui memory_type, Fase 05)
│   ├── ingest_document.py         # ingesta um documento real no global_scope (memory_type="knowledge", Fase 05)
│   ├── search.py                  # CLI de busca (hybrid search + rerank) no global_scope
│   ├── search_web.py              # CLI de busca isolada via SearXNG (Fase 03)
│   ├── fetch_page.py              # CLI de fetch + extração isolada via Crawl4AI (Fase 03)
│   ├── web_research.py            # orquestrador end-to-end: multi-query -> busca -> fetch -> evidências (Fase 03)
│   ├── build_sandbox.sh           # Fase 04 — builda a imagem do sandbox
│   ├── generate_code.py           # Fase 04 — CLI: gera código via Qwen3-Coder (com --run para já executar no sandbox)
│   ├── solve_task.py              # Fase 04 — CLI: loop completo (gera -> executa -> corrige -> executa de novo)
│   ├── save_memory.py             # NOVO (Fase 05) — CLI do /save: promove chat_scope -> global_scope (5.2)
│   ├── add_memory.py              # NOVO (Fase 05) — CLI da nota manual avulsa, memory_type="manual" (5.3)
│   ├── list_memories.py           # NOVO (Fase 05) — CLI: lista memórias (filtros) ou vê detalhe de uma por --id (5.4)
│   ├── edit_memory.py             # NOVO (Fase 05) — CLI: edita tags (mesmo ID) ou texto (ID novo) de uma memória (5.4)
│   └── delete_memory.py           # NOVO (Fase 05) — CLI: apaga memórias por ID, com confirmação (5.4)
│
├── src/                          # código de produção, importável pelos scripts
│   ├── __init__.py
│   ├── ingestion/
│   │   ├── __init__.py
│   │   ├── embedding_client.py       # embed_text() / embed_texts() via Ollama
│   │   ├── parser.py                  # leitura de .md/.txt
│   │   └── chunking.py                # chunking por palavras com overlap
│   ├── retrieval/
│   │   ├── __init__.py
│   │   ├── fusion.py                  # Reciprocal Rank Fusion (RRF)
│   │   ├── bm25_index.py              # índice sparse (BM25) em memória
│   │   ├── hybrid_search.py           # orquestra dense (Qdrant) + sparse (BM25)
│   │   └── llm_reranker.py            # reranking via GPT-OSS (prompt + JSON Schema)
│   ├── search/
│   │   ├── __init__.py
│   │   ├── searxng_client.py          # cliente de busca via SearXNG (3.1)
│   │   ├── page_fetcher.py            # fetch + extração de conteúdo via Crawl4AI (3.2)
│   │   ├── evidence.py                # dedup + chunking + inserção no chat_scope (3.3)
│   │   ├── query_expansion.py         # geração de variações de query via GPT-OSS (3.4)
│   │   └── multi_query.py             # orquestra busca multi-query + dedup por URL (3.4)
│   ├── sandbox/                        # Fase 04
│   │   ├── __init__.py
│   │   └── executor.py                # run_code(): executa código em container Docker efêmero e isolado (4.1)
│   ├── coding/                         # Fase 04
│   │   ├── __init__.py
│   │   ├── coder_client.py            # generate_code(): gera/corrige código via Qwen3-Coder/Ollama (4.2)
│   │   └── agent_loop.py              # solve_task(): loop de iteração gera -> executa -> corrige (4.3)
│   └── memory/                         # NOVO (Fase 05)
│       ├── __init__.py
│       ├── schema.py                  # memory_point_id(), build_memory_payload(), is_already_saved() — schema comum aos 3 tipos de memória (5.1)
│       ├── save.py                    # save_memory(): lógica do /save, chat_scope -> global_scope (5.2)
│       ├── add_note.py                # add_note(): lógica da nota manual avulsa, memory_type="manual" (5.3)
│       └── manage.py                  # list_memories(), get_memory(), update_tags(), update_text(), delete_memories() (5.4)
│
├── tests/                        # testes de validação — rodar após qualquer mudança no pipeline
│   ├── test_embedding_dim.py        # confirma dimensão do embedding (precisa Ollama)
│   ├── test_end_to_end.py           # embed -> insert -> search no Qdrant (precisa Ollama + Qdrant)
│   ├── test_chunking.py             # lógica de chunking, sem rede
│   ├── test_rrf.py                  # lógica de fusão RRF, sem rede
│   ├── test_llm_reranker_parsing.py # parsing da resposta do reranker, sem rede
│   ├── test_searxng_client.py       # cliente de busca SearXNG (precisa SearXNG)
│   ├── test_page_fetcher.py         # fetch + extração via Crawl4AI (precisa rede + Chromium)
│   ├── test_evidence_dedup.py       # lógica de montagem de evidência, sem rede
│   ├── test_evidence_index.py       # indexação no chat_scope (precisa Ollama + Qdrant)
│   ├── test_query_expansion_parsing.py  # parsing das variações de query, sem rede
│   ├── test_multi_query_dedup.py    # merge de queries + dedup por URL, sem rede
│   ├── test_web_research_e2e.py     # pipeline completo da Fase 03 de ponta a ponta (precisa tudo no ar)
│   ├── test_sandbox_executor.py     # Fase 04 — executor do sandbox: sucesso, erro, timeout, sem rede, limpeza (precisa Docker)
│   ├── test_coder_client_parsing.py # Fase 04 — extract_code()/build_prompt(), sem rede
│   ├── test_agent_loop.py           # Fase 04 — controle do loop de iteração, com fakes (sem Ollama/Docker)
│   ├── test_coding_agent_e2e.py     # Fase 04 — pipeline completo da Fase 04 de ponta a ponta (precisa Ollama + Docker)
│   ├── test_memory_schema.py        # NOVO (Fase 05) — memory_point_id()/build_memory_payload(), sem rede (5.1)
│   ├── test_save_memory.py          # NOVO (Fase 05) — save_memory() com Qdrant fake, sem rede (5.2)
│   ├── test_add_note.py             # NOVO (Fase 05) — add_note() com Qdrant fake + embed_text fake via monkeypatch, sem rede (5.3)
│   ├── test_manage_memory.py        # NOVO (Fase 05) — list/get/edit/delete de memórias com Qdrant fake, sem rede (5.4)
│   └── test_memory_e2e.py           # NOVO (Fase 05) — pipeline completo da Fase 05 de ponta a ponta (precisa Ollama + Qdrant) (5.5)
│
├── chats/                        # ESCOPO TEMPORÁRIO (Decision 018)
│   └── {chat_id}/                    # uma pasta por conversa; apagada junto com o chat
│
├── knowledge/                    # ESCOPO GLOBAL / PERSISTENTE (Decision 018)
│   ├── documents/                    # documentos originais promovidos via ingestão
│   └── cache/
│
├── searxng/                      # config do SearXNG gerada pelo container (settings.yml, gitignored)
│
└── data/
    └── qdrant/                       # storage do Qdrant em disco (volume do docker-compose)
```

## Convenção de pastas

| Pasta | Regra |
|---|---|
| `scripts/` | Só scripts que **você** roda manualmente. Nunca é importado por outro código. |
| `src/` | Código de produção, importável. Cresce por área (`ingestion/`, `retrieval/`, `search/`, `sandbox/`, `coding/`, `memory/`, e futuramente `agent/`, `crawler/`, etc., conforme novas fases). |
| `tests/` | Um teste por unidade de lógica testada. Os que não dependem de rede/Docker devem rodar sempre, rápido, sem serviço externo no ar. |
| `chats/` vs `knowledge/` | Nunca misturar. `chats/` é descartável por definição; `knowledge/` é o que persiste. |
| `sandbox/` | Só a definição da imagem Docker (`Dockerfile`) — não é código Python importável, por isso fica fora de `src/`. |

## Papel dos arquivos de documentação

O projeto tem três arquivos de documentação com papéis bem diferentes — não são intercambiáveis:

| Arquivo | Papel | Frequência de mudança |
|---|---|---|
| `README.md` | Referência estável: estrutura do projeto, comandos de setup do zero, regra geral de onde colocar arquivos novos. Não cresce a cada passo. | Raramente — só quando o fluxo geral de uso muda de verdade. |
| `docs/STATUS.md` | Tabela curta de progresso por passo/fase (o que já foi validado, o que falta) + decisões-chave resumidas. | A cada passo concluído, mas só a tabela/linhas — não vira texto corrido. |
| `docs/TUTORIAL.md` | Explica **só os arquivos entregues no passo mais recente**: onde colocar cada um, como rodar, checklist de validação daquele passo específico. | **Substituído por completo** a cada nova entrega — não acumula histórico de passos antigos. |

Essa separação existe porque, no início da Fase 02, tudo isso estava misturado dentro do `README.md`, que foi ficando grande e confuso a cada passo novo. Separar por papel (estável vs. progresso vs. instrução pontual) resolveu isso.

## Como este arquivo é mantido

Diferente do `docs/TUTORIAL.md` (que é substituído a cada passo), este arquivo é **cumulativo** — atualizado sempre que uma nova fase adicionar pastas/módulos novos à estrutura real do projeto. Na Fase 03, isso aconteceu com a adição de `src/search/` e dos scripts/testes associados. Na Fase 04, com a adição de `sandbox/`, `src/sandbox/` e `src/coding/` — e também com a correção dos `__init__.py` que faltavam em `src/`, `src/ingestion/` e `src/retrieval/`. Na Fase 05, com a adição de `src/memory/` (schema, save, add_note, manage) e dos scripts/testes de memória associados — nenhuma pasta nova de infraestrutura foi necessária (Decision 036: metadata de memória fica no payload do Qdrant, sem componente novo).
