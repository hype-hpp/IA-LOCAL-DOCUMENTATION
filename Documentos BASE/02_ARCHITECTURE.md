# IA Local — Architecture

## Arquitetura de alto nível

```text
USER
  |
  v
OPEN WEB UI(Deve ser ignorado nas fases iniciais e substituido por alternativas mais simples e faceis como Gradio ou Streamlit)
  |
  v
AGENT CORE
  |- Planner
  |- Model Router
  |- Memory Manager
  |- Research Controller
  |- Tool Manager
  |- Verifier
  |
  +--> Web Search
  +--> Browser
  +--> Adaptive Crawler
  +--> Knowledge / RAG
  +--> Memory
  +--> Coding Sandbox
  |
  v
SPECIALIZED MODELS
  |- General
  |- Coding
  |- Vision
  |- Embedding
  |
  v
FINAL RESPONSE
```

## Dados

### PostgreSQL

Responsável por dados estruturados:

- sessões
- conversas
- memórias
- tarefas de pesquisa
- jobs do crawler
- URLs
- metadata
- logs e traces

### Qdrant

Responsável por dados vetoriais:

- documentos
- chunks
- memória semântica
- memória de pesquisa
- embeddings multimodais quando aplicável

### Filesystem
Separado em dois escopos (DECISION 018):
- Escopo do Chat: `./chats/{chat_id}/` (Temporário, apagado junto com o chat).
- Escopo Global: `./knowledge/` (Persistente, alimentado via promoção manual /save ou UI).
Responsável pelos originais:

- PDFs
- Markdown
- HTML/arquivos processados
- imagens
- código
- cache controlado

## Retrieval

Arquitetura-alvo:

```text
QUERY
  |
  +--> Dense retrieval
  |
  +--> Sparse/BM25
  |
  +--> Metadata filtering
  |
  v
Hybrid fusion
  |
  v
Top-N candidates
  |
  v
Reranker
  |
  v
Top-K context
  |
  v
LLM
```

## Pesquisa web

```text
Agent
  |
  v
Search
  |
  v
Candidate URLs
  |
  +--> Browser / Playwright
  |
  +--> Crawl4AI
  |
  v
Evidence / Documents
  |
  v
Knowledge layer
```

## Crawler adaptativo

A ideia principal é permitir que o próprio agente determine:

- profundidade máxima
- número máximo de páginas
- tópicos prioritários
- assuntos a excluir
- domínios permitidos
- orçamento de armazenamento
- quando a cobertura é suficiente

O crawler deve poder parar quando o agente determinar que já existe evidência suficiente.

## Coding

```text
Agent
  |
  v
Coding Model
  |
  v
Docker Sandbox (
  Com pacotes pre instalado
Imagem base: python:3.11-slim.
Pacotes pré-instalados: numpy, pandas, requests, beautifulsoup4, matplotlib, scipy.
Segurança: Sem montagem do diretório /home do usuário, proibições de pip install sem autorização prévia e execução em volumes temporários descartáveis quando forem necessárias bibliotecas adicionais.
)
  |
  +--> execute
  +--> test
  +--> observe error
  +--> fix
  +--> test again
```

O código não deve receber acesso direto ao filesystem pessoal do usuário.

## Vision
Pipeline perceptual (DECISION 021):
Imagens/PDFs -> Qwen3-VL 8B (OCR + Descrição) -> Texto injetado no prompt do GPT-OSS -> Resposta final gerada pelo GPT-OSS.

O sistema deve suportar:

- imagens
- screenshots
- PDFs
- páginas web visualmente
- vídeos

Um modelo visual especializado pode atuar como camada de percepção para um modelo geral.
