# IA Local — Decisions

## DECISION 001 — Local-first
O sistema será local-first. Modelos locais e infraestrutura local são o padrão. Cloud pode ser utilizada como fallback quando trouxer benefício significativo.
Status: DEFINITIVA

---

## DECISION 002 — Agentic architecture
O projeto será construído como um sistema agentic (planner, model router, memory, tools, research controller, verifier).
Status: DEFINITIVA

---

## DECISION 003 — Hybrid retrieval
O retrieval final será híbrido: dense, sparse/BM25, metadata filtering e reranking.
Status: DEFINITIVA

---

## DECISION 004 — PostgreSQL + Qdrant
Usar PostgreSQL para dados estruturados e Qdrant para dados vetoriais.
Status: DEFINITIVA

---

## DECISION 005 — Sandbox
Execução de código ocorrerá em ambiente isolado Docker. Nenhum agente terá acesso direto ao filesystem pessoal do usuário.
Status: DEFINITIVA

---

## DECISION 009 — Crawler adaptativo
O crawler poderá receber parâmetros definidos pelo próprio agente (profundidade, máx páginas, tópicos, exclusões, orçamento).
Status: DEFINITIVA

---

## DECISION 014 — Carregamento e Gerenciamento de Modelos (Sob Demanda)
- GPT-OSS 20B residente na VRAM (100% GPU) como orquestrador.
- Qwen3-Coder 30B e Qwen3.6 27B carregados sob demanda.
- Qwen3-Coder mantido pré-carregado na RAM para evitar cold start.
Status: DEFINITIVA (Substitui antigas 006, 007 e 008)

---

## DECISION 015 — Verificador Universal (Sempre Ativo)
O Verifier rodará em todas as respostas usando um modelo leve (ex: Qwen2.5-7B) exclusivamente na CPU.
Status: DEFINITIVA (Substitui antiga 010)

---

## DECISION 016 — Arquitetura de Duas IAs (Orquestrador + Especialista)
- Orquestrador: GPT-OSS 20B (entende prompt, planeja, interage).
- Especialista de Código: Qwen3-Coder 30B (tratado estritamente como Tool acionada pelo GPT-OSS).
Status: DEFINITIVA

---

## DECISION 017 — Validação e Correção dos Benchmarks
- GPT-OSS 20B corrigido para Média 4.00 (140 tok/s). Escolha definitiva para padrão.
- Qwen3-Coder 30B (Média 4.00, 82 tok/s). Tool de código.
- Qwen3.6 27B (Média 4.50, 8.9 tok/s). Uso excepcional.
Status: DEFINITIVA

---

## DECISION 018 — Sistema de Armazenamento Híbrido (Escopo do Chat vs. Escopo Global)
- Escopo do Chat (Local): `./chats/{chat_id}/` + Qdrant `scope: "chat"`. Apagado ao deletar a conversa.
- Escopo Global (Memória): `./knowledge/` + Qdrant `scope: "global"`. Persistente. Promoção manual via `/save` ou UI.
Status: DEFINITIVA

---

## DECISION 019 — Estratégia de Interface (UI)
- Fases 02 a 07: Interface simples em Gradio ou Streamlit.
- Fase 08+: Expor Agent Core via API OpenAI (`/v1/chat/completions`) e conectar ao Open WebUI.
Status: DEFINITIVA

---

## DECISION 020 — Ambiente de Execução (Docker Sandbox com Pacotes)
Imagem `python:3.11-slim` com `numpy`, `pandas`, `requests`, `beautifulsoup4`, `matplotlib`, `scipy`. Sem `pip install` sem autorização. Sem montagem do `/home`.
Status: DEFINITIVA

---

## DECISION 021 — Pipeline de Percepção Visual
Qwen3-VL 8B extrai OCR e gera descrição textual -> relatório é injetado no contexto do GPT-OSS -> GPT-OSS responde ao usuário.
Status: DEFINITIVA

---

## DECISION 022 — Duas Collections no Qdrant (chat_scope / global_scope)
A separação de escopo definida na Decision 018 é implementada como **duas collections separadas** no Qdrant (`chat_scope` e `global_scope`), não uma única collection com filtro de payload por escopo.

Motivo: isolamento mais simples ao apagar um chat (drop direto dos pontos daquela collection, sem depender de filtro), e menor risco de vazamento entre escopos por filtro esquecido em alguma query.

Ambas as collections usam vetores de dimensão 2560 (Qwen3-Embedding-4B, confirmado empiricamente rodando o modelo real via Ollama), distância COSINE, e índices de payload em `chat_id`, `source` e `content_hash`.
Status: DEFINITIVA

---

## DECISION 023 — Estratégia de Chunking e Deduplicação
Chunking por **palavras** (não caracteres, não tokens), tamanho padrão de 250 palavras com overlap de 40 palavras, via janela deslizante — evita cortar palavras no meio e é mais simples que depender de um tokenizer.

Deduplicação por SHA256 do texto de cada chunk. O ID do ponto no Qdrant é um UUID determinístico (`uuid5`) derivado desse hash — reingerir o mesmo conteúdo sobrescreve o ponto existente em vez de duplicar, sem precisar de lógica extra de dedup no momento da inserção.
Status: DEFINITIVA (tamanho de chunk é revisável após observar qualidade real de retrieval em uso)

---

## DECISION 024 — Hybrid Search via Reciprocal Rank Fusion (RRF)
Busca híbrida implementada como:
- Dense retrieval via Qdrant (cosine similarity)
- Sparse/BM25 em memória (`rank_bm25`), reconstruído a cada busca
- Fusão das duas listas via RRF (k=60)

BM25 em memória foi escolhido em vez de sparse vectors nativos do Qdrant — a alternativa nativa exigiria um modelo adicional (ex: SPLADE) e mais complexidade de setup, sem benefício comprovado para o tamanho de um acervo de conhecimento pessoal. Migrar para sparse vectors nativos fica condicionado a benchmark real mostrando necessidade (regra 5 do projeto).
Status: DEFINITIVA

---

## DECISION 025 — Reranker via GPT-OSS (sem modelo dedicado)
Não existe reranker oficial do Qwen3 disponível no Ollama — apenas builds de terceiros (`dengcao/`, `ggml-org/`, etc.), e o uso correto de um cross-encoder como o Qwen3-Reranker (via logit de "yes/no") é frágil de implementar de forma confiável pela API do Ollama.

Optou-se por usar o próprio orquestrador (GPT-OSS 20B, já residente na VRAM) para pontuar a relevância dos candidatos via prompt, com a resposta forçada a um JSON Schema exato (grammar-constrained decoding do Ollama) em vez de só `"format": "json"` genérico — isso evita ambiguidade de estrutura na resposta do modelo. Fallback automático para a ordem crua do RRF caso o parsing da resposta do LLM falhe.

Trade-off aceito: adiciona uma chamada de LLM (latência) por busca. Nenhum modelo novo precisou ser baixado ou gerenciado.
Status: DEFINITIVA (revisar se a latência adicional se mostrar proibitiva em uso real; ver regra 5)

---

## DECISION 026 — Extração de conteúdo via Crawl4AI (não Playwright puro)
Optou-se por usar Crawl4AI em vez de montar Playwright + uma lib de extração de texto separada (trafilatura/readability). Crawl4AI já usa Playwright internamente e devolve markdown limpo pronto, resolvendo fetch + extração num único componente — evita montar e manter duas peças separadas para o mesmo objetivo.

Trade-off aceito: uma dependência a mais (`crawl4ai`, com setup próprio via `crawl4ai-setup` para baixar o Chromium). Em compensação, elimina a necessidade de escolher e manter uma lib de extração de texto à parte.
Status: DEFINITIVA

---

## DECISION 027 — Escopo padrão das evidências de pesquisa web: chat_scope
Evidências coletadas via pesquisa web (SearXNG + Crawl4AI) são inseridas por padrão no `chat_scope` (Decision 018), não no `global_scope`. Motivo: resultado de pesquisa é, por padrão, contexto específico de uma conversa/tarefa — só deve virar conhecimento permanente se o usuário decidir promover via `/save` (mecanismo da Fase 05, ver Decision 038).

O dedup por hash é escopado por `chat_id` (namespace `uuid5` = `chat_id:content_hash`), permitindo que o mesmo conteúdo exista em chats diferentes sem conflito de ID, mas sem duplicar dentro do mesmo chat.
Status: DEFINITIVA

---

## DECISION 028 — Multi-query via GPT-OSS (mesmo princípio do reranker)
Reaproveitado o mesmo padrão da Decision 025: usar o GPT-OSS (já residente na VRAM) como "worker" para gerar variações da query do usuário via prompt + JSON Schema forçado (grammar-constrained decoding do Ollama), em vez de treinar ou rodar lógica dedicada de expansão de query.

Fallback: se o LLM falhar ou não retornar nada parseável, a busca segue apenas com a query original — não é tratado como erro fatal. Resultados de queries diferentes (original + variações) são deduplicados por URL antes do fetch, mantendo a ordem de primeira aparição.
Status: DEFINITIVA (revisar número de variações padrão — atualmente 3 — se o custo de latência não se justificar pelo ganho de cobertura, regra 5 do projeto)

---

## DECISION 029 — Top 10 candidatos por busca (default)
Número padrão de URLs candidatas buscadas por query — e, no caso de multi-query, o teto após dedup entre as variações — definido em 10, replicando o padrão já usado no `candidate_k` do reranker da Fase 02 (`scripts/search.py`). Ajustável via `--max-results` no `web_research.py`.
Status: DEFINITIVA (revisável com uso real)

---

## DECISION 030 — Falha de fetch por página é não-fatal (skip, não abort)
Páginas bloqueadas por proteção anti-bot (Cloudflare, Imperva/Incapsula, etc.) ou que falham por qualquer outro motivo no fetch (`page_fetcher.py`) são simplesmente puladas na montagem de evidências (`build_evidence_chunks`), sem abortar o restante do pipeline. Confirmado em uso real (Fase 03, testes com queries sobre RAG): de 10 candidatos, tipicamente 8-9 são extraídos com sucesso, e isso é suficiente para gerar evidência útil — não há necessidade de retry, proxy rotation ou bypass de anti-bot neste estágio.
Status: DEFINITIVA (revisar se a taxa de falha aumentar a ponto de comprometer a qualidade da evidência coletada)

---

## DECISION 031 — Sandbox de Execução: Container Efêmero, Sem Rede, com Hardening Padrão
Implementação do sandbox de execução de código (Decision 005/020) definida no passo 4.1 da Fase 04:

- **Container efêmero por execução** (`docker run --rm`), não um container persistente com `docker exec`. Prioriza isolamento sobre latência de cold start (~1-2s por chamada) — aceitável, já que o coding agent não é uma rota de conversação de baixa latência.
- **Sem rede por padrão** (`--network none`). Reduz a superfície de risco de executar código gerado e não revisado. Código que precise de rede vai falhar de propósito.
- Hardening adicional (engenharia padrão de sandbox, não decisões em disputa): limite de memória (512m) e CPU (1 core), `--pids-limit 100` (evita fork bomb), `--cap-drop ALL` + `--security-opt no-new-privileges` (reduz superfície de escalonamento de privilégio).
- Único volume montado é um diretório temporário criado e apagado pelo próprio módulo (`src/sandbox/executor.py`) a cada execução — nunca o `/home` do usuário, reforçando a Decision 020.

Status: DEFINITIVA (rede desabilitada é revisável se algum caso de uso real exigir, regra 5 do projeto)

---

## DECISION 032 — Correção de Permissão no Sandbox (chmod do diretório/arquivo temporário)
Bug real encontrado em teste no hardware (passo 4.1): `--cap-drop ALL` (Decision 031) remove a capability `CAP_DAC_OVERRIDE`, que é o que normalmente permite ao root **dentro** do container ler arquivos de qualquer dono. Sem essa capability, o container (rodando como root) não conseguia ler o `script.py` montado, que pertencia ao usuário do host com permissão `0700` (padrão do `tempfile.mkdtemp()`).

Corrigido relaxando a permissão do diretório temporário para `0755` e do arquivo do script para `0644` antes do `docker run` — seguro porque são artefatos efêmeros, sem dado sensível, apagados logo em seguida pelo próprio módulo.

Status: DEFINITIVA

---

## DECISION 033 — Geração de Código via Qwen3-Coder Sem JSON Schema Forçado
Ao contrário do reranker (Decision 025) e do query expansion (Decision 028), a geração de código no passo 4.2 **não** usa grammar-constrained JSON Schema. O Qwen3-Coder responde com um bloco de código cercado por ```` ```python ````, extraído via regex (`extract_code()` em `src/coding/coder_client.py`).

Motivo: forçar código Python inteiro dentro de uma string JSON exigiria escapar quebras de linha, aspas, etc., sem benefício real sobre pedir um bloco cercado e extrair via regex — abordagem padrão para geração de código com LLMs (regra 1 do projeto: não adicionar complexidade sem necessidade).

Uma única função `generate_code(task, previous_code=None, error=None)` cobre tanto geração de código novo quanto correção de código existente (dado o erro de execução), reaproveitada sem duplicação de lógica de prompt pelo loop de iteração (Decision 034).

Status: DEFINITIVA

---

## DECISION 034 — Loop de Iteração: Falha de Infraestrutura na Geração Não Conta como Tentativa
No loop de iteração (`src/coding/agent_loop.py`, passo 4.3), uma falha de **infraestrutura ao gerar código** (`CoderError`, ex: Ollama fora do ar) interrompe o loop imediatamente, sem incrementar o contador de tentativas (`attempts_used` permanece em 0 nesse caso).

Motivo: não faz sentido tratar "não deu nem pra gerar código" da mesma forma que "o código gerado falhou na execução" — só o segundo caso é elegível a correção automática via retry (feedback do erro real de execução para o Qwen3-Coder).

Status: DEFINITIVA

---

## DECISION 035 — DEFAULT_MAX_ATTEMPTS Ajustado de 3 para 5
Valor padrão de tentativas do loop de iteração (`src/coding/agent_loop.py`) ajustado de 3 para 5, após uso real no hardware (passo 4.3): 3 tentativas se mostrou pouco para tarefas que exigem mais de uma correção — exemplo real observado: uma tarefa de leitura de um CSV inexistente levou exatamente 3 tentativas até o Qwen3-Coder contornar sozinho (usando `tempfile` em vez de tentar escrever em `/sandbox`, que não tem permissão de escrita sem `CAP_DAC_OVERRIDE` — ver Decision 032).

Status: DEFINITIVA (revisão futura para 10 possível, condicionada a mais uso real — regra 5 do projeto)

---

## DECISION 036 — Metadata de Memória Só no Payload do Qdrant (Sem PostgreSQL)
A metadata estruturada das memórias (tipo, tags, quando foi salva, origem) fica inteiramente no payload dos pontos do Qdrant em `global_scope`, sem introduzir um componente novo — a Decision 004 previa PostgreSQL para dados estruturados em geral, mas isso nunca chegou a ser implementado. Reaproveita a mesma collection/estrutura já validada nas Fases 02/03.

Motivo: regra 1 do projeto (não criar componente sem necessidade real) — o volume de um acervo de memória pessoal não justifica, por ora, a complexidade operacional de rodar e manter um PostgreSQL à parte só para isso.

Status: DEFINITIVA (revisável se o volume ou a complexidade de consulta crescerem a ponto de o payload do Qdrant não ser suficiente — regra 5 do projeto)

---

## DECISION 037 — Três Tipos de Memória em global_scope via campo memory_type
Toda memória em `global_scope` carrega um campo `memory_type`, com três valores possíveis:
- `"knowledge"`: chunk de documento ingerido diretamente (`scripts/ingest_document.py`, Fase 02).
- `"research"`: evidência de pesquisa web promovida do `chat_scope` via `/save` (Fase 05, 5.2, ver Decision 038).
- `"manual"`: nota livre adicionada diretamente pelo usuário, sem busca prévia (Fase 05, 5.3, ver Decision 039).

Os três tipos compartilham o mesmo formato-base de payload (`text`, `content_hash`, `source`, `memory_type`, `tags`, `saved_at`), centralizado em `src/memory/schema.py`, para poderem ser listados/filtrados de forma uniforme (5.4, ver Decision 040). O campo `ingested_at` da Fase 02 foi renomeado para `saved_at` para manter esse formato comum entre os três tipos — sem dado real a migrar no momento da mudança.

Status: DEFINITIVA

---

## DECISION 038 — /save: Seleção Explícita por chat_id + IDs, Cópia (não Move), Vetor Reaproveitado
O mecanismo `/save` (promoção de `chat_scope` para `global_scope`, Decision 018/027) foi implementado com três escolhas deliberadas de hp:
- **Seleção explícita**: `chat_id` + lista de IDs específicos (`--ids`) — não existe modo de "promover tudo do chat de uma vez". `/save` é curadoria deliberada, não dump em massa.
- **Cópia, não move**: o ponto original permanece em `chat_scope` depois de promovido — some só quando o chat inteiro for apagado (comportamento padrão do escopo temporário, Decision 018).
- **Reaproveitamento de vetor**: a promoção usa o vetor já existente no ponto do `chat_scope` (`with_vectors=True`), sem chamar o Ollama de novo para reembeddar o mesmo conteúdo.

Implementado em `src/memory/save.py` + `scripts/save_memory.py`, testado com Qdrant fake (`tests/test_save_memory.py`) e validado com promoção real no hardware.

Status: DEFINITIVA

---

## DECISION 039 — Nota Manual Avulsa: Texto via --text, source Fixo em "manual"
A nota manual (`memory_type="manual"`, Fase 05, 5.3) recebe o texto como argumento direto de linha de comando (`--text`), sem abrir um editor externo (tipo `git commit`). O campo `source` é fixo em `"manual"` para todo ponto deste tipo — sem um rótulo por nota; a categorização de notas manuais entre si fica por conta das tags (já suportadas pelo schema desde a Decision 037).

Diferente do `/save`, aqui o texto é novo (sem vetor pré-existente), então precisa ser embeddado via Ollama — mas só depois de confirmar que o conteúdo ainda não existe em `global_scope` (dedup por `content_hash`), para não gastar uma chamada de embedding à toa.

Implementado em `src/memory/add_note.py` + `scripts/add_memory.py`.

Status: DEFINITIVA (revisável se um rótulo por nota fizer falta na prática — regra 5 do projeto)

---

## DECISION 040 — Editar Memória: Texto Gera ID Novo, Apagar Só por ID Explícito
Visualizar/editar/apagar memórias (regra 10 do projeto, Fase 05, 5.4) foi implementado em `src/memory/manage.py` com duas escolhas deliberadas de hp:
- Editar **TEXTO** reembedda via Ollama e necessariamente gera um **ID NOVO**, já que o ID do ponto em `global_scope` é derivado do `content_hash` do texto (Decision 023). O ponto antigo é apagado e o novo inserido no lugar. Editar só as **TAGS** é uma troca simples de payload (via `set_payload`), sem reembedding e sem mudar o ID.
- Apagar é só por **ID explícito** (um ou vários) — sem suporte a apagar em massa por filtro (`memory_type`/tag/`chat_id`), para reduzir o risco de apagar mais do que o pretendido.

Proteção adicional (não pedida explicitamente, mas coerente com a cautela da regra 11 do projeto para ferramentas que destroem dado): `scripts/delete_memory.py` pede confirmação interativa antes de apagar de verdade, pulável com `--yes` para uso em automação. `update_text()` também recusa editar um texto que colidiria (mesmo `content_hash`) com uma memória já existente, para não sobrescrevê-la silenciosamente.

Status: DEFINITIVA

---

## DECISION 041 — Bug de Log Corrigido em create_collections.py (Idempotência Real do Qdrant)
Achado real em teste no hardware durante o passo 5.1: a versão inicial de `scripts/create_collections.py` assumia que `client.create_payload_index()` do Qdrant levantaria uma exceção se o índice de payload já existisse, usando isso para decidir entre logar "[ok] criado" ou "[skip] já existe". Na prática, essa chamada é idempotente no servidor — nunca levanta exceção, sempre retorna sucesso — então o script sempre logava "criado", mesmo rodando pela terceira vez seguida sem nada novo para criar. Nada quebrou funcionalmente (o índice não duplicava), mas o log mentia sobre o que de fato aconteceu.

Corrigido checando o `payload_schema` da collection antes de decidir se precisa criar o índice, em vez de confiar numa exceção que nunca chegava a ser levantada.

Status: DEFINITIVA

---

## DECISION 042 — Escopo da Fase 05: "Memória de Conversa" Fora, Interface Continua CLI
Duas decisões de escopo tomadas por hp antes de iniciar a Fase 05:
- "Memória de conversa" (um dos itens do objetivo original da fase, ver `05_ROADMAP.md`) ficou fora do escopo mínimo: não existe Agent Core/loop de chat real ainda (só chega na Fase 07), então não faz sentido construir memória de uma conversa que ainda não acontece de verdade fora de scripts CLI isolados. A fase focou em `/save` + memória semântica (`knowledge`) + memória de pesquisa (`research`) + nota manual (`manual`).
- A interface continua CLI nesta fase, não a UI Gradio/Streamlit prevista na Decision 019 — fica para quando houver necessidade real ou a Fase 08 (UI + Integração).

Status: DEFINITIVA (revisável quando a Fase 07 - Agent Core - existir de fato)
