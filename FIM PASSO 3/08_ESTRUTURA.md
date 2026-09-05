# IA Local — Estrutura de Pastas

Referência da estrutura real do projeto em `~/IA-LOCAL/`, como ficou ao final da Fase 03.

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
├── scripts/                    # ferramentas administrativas — rodar manualmente, sob demanda
│   ├── setup_dirs.sh              # cria chats/, knowledge/, data/
│   ├── create_collections.py      # cria as collections do Qdrant (idempotente)
│   ├── ingest_document.py         # ingesta um documento real no global_scope
│   ├── search.py                  # CLI de busca (hybrid search + rerank) no global_scope
│   ├── search_web.py              # CLI de busca isolada via SearXNG (Fase 03)
│   ├── fetch_page.py              # CLI de fetch + extração isolada via Crawl4AI (Fase 03)
│   └── web_research.py            # orquestrador end-to-end: multi-query -> busca -> fetch -> evidências (Fase 03)
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
│   └── search/                        # NOVO (Fase 03)
│       ├── __init__.py
│       ├── searxng_client.py          # cliente de busca via SearXNG (3.1)
│       ├── page_fetcher.py            # fetch + extração de conteúdo via Crawl4AI (3.2)
│       ├── evidence.py                # dedup + chunking + inserção no chat_scope (3.3)
│       ├── query_expansion.py         # geração de variações de query via GPT-OSS (3.4)
│       └── multi_query.py             # orquestra busca multi-query + dedup por URL (3.4)
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
│   └── test_web_research_e2e.py     # pipeline completo da Fase 03 de ponta a ponta (precisa tudo no ar)
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
| `src/` | Código de produção, importável. Cresce por área (`ingestion/`, `retrieval/`, `search/`, e futuramente `agent/`, `crawler/`, etc., conforme novas fases). |
| `tests/` | Um teste por unidade de lógica testada. Os que não dependem de rede devem rodar sempre, rápido, sem serviço externo no ar. |
| `chats/` vs `knowledge/` | Nunca misturar. `chats/` é descartável por definição; `knowledge/` é o que persiste. |

## Papel dos arquivos de documentação

O projeto tem três arquivos de documentação com papéis bem diferentes — não são intercambiáveis:

| Arquivo | Papel | Frequência de mudança |
|---|---|---|
| `README.md` | Referência estável: estrutura do projeto, comandos de setup do zero, regra geral de onde colocar arquivos novos. Não cresce a cada passo. | Raramente — só quando o fluxo geral de uso muda de verdade. |
| `docs/STATUS.md` | Tabela curta de progresso por passo/fase (o que já foi validado, o que falta) + decisões-chave resumidas. | A cada passo concluído, mas só a tabela/linhas — não vira texto corrido. |
| `docs/TUTORIAL.md` | Explica **só os arquivos entregues no passo mais recente**: onde colocar cada um, como rodar, checklist de validação daquele passo específico. | **Substituído por completo** a cada nova entrega — não acumula histórico de passos antigos. |

Essa separação existe porque, no início da Fase 02, tudo isso estava misturado dentro do `README.md`, que foi ficando grande e confuso a cada passo novo. Separar por papel (estável vs. progresso vs. instrução pontual) resolveu isso.

## Como este arquivo é mantido

Diferente do `docs/TUTORIAL.md` (que é substituído a cada passo), este arquivo é **cumulativo** — atualizado sempre que uma nova fase adicionar pastas/módulos novos à estrutura real do projeto. Na Fase 03, isso aconteceu com a adição de `src/search/` e dos scripts/testes associados.
