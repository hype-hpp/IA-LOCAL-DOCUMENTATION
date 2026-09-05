# IA Local — Benchmarks

## Benchmark de velocidade

### GPT-OSS 20B

| Contexto | tok/s |
|---:|---:|
| 2K | 140 |
| 16K | 140 |
| 64K | 120 |

Offload no teste:
- 100% GPU nos contextos menores testados

---

### Qwen3-Coder 30B

| Contexto | tok/s |
|---:|---:|
| 2K | 82 |
| 16K | 72 |
| 64K | 45 |

Offload no teste:
- ~27% GPU
- ~73% CPU

---

### Qwen3.6 27B

| Contexto | tok/s |
|---:|---:|
| 2K | 8.9 |
| 16K | 7.7 |
| 64K | 5.5 |

Offload no teste:
- ~24% GPU
- ~76% CPU

## Interpretação provisória

O benchmark confirma uma diferença prática importante entre o modelo denso e o MoE no hardware disponível.

O GPT-OSS 20B é o candidato mais forte para interações rápidas.

O Qwen3-Coder 30B é muito mais lento que o GPT-OSS, mas continua em uma faixa bastante utilizável para tarefas pesadas/código.

O Qwen3.6 27B é muito mais lento devido ao custo de offload do modelo denso e deve ser reservado para tarefas em que a qualidade esperada compense a latência.

## Benchmark de qualidade

qwen3.6:27b        média 4.50   notas: [4.0, 5.0]
qwen3-coder:30b    média 4.00   notas: [3.0, 5.0]
gpt-oss:20b        média 4.00   notas: [4.0, 4.0]

### Categorias planejadas

- instruções
- raciocínio
- coding
- pesquisa factual
- escrita
- uso de ferramentas
- análise multimodal

### Critérios

- correção
- raciocínio
- completude
- precisão técnica
- clareza

## Regra

Velocidade nunca deve ser o único critério para escolher o modelo.

---

## Fase 02 — RAG / Knowledge

### Confirmado

- Dimensão de embedding do Qwen3-Embedding-4B: **2560** (validado rodando o modelo real via Ollama, não só documentação).

### Pendente (registrar quando houver uso real com volume)

- Latência de `hybrid_search()` isolada (dense + BM25 + RRF, sem rerank).
- Latência adicional do reranker via GPT-OSS por busca (depende do número de candidatos enviados no prompt).
- Comportamento do BM25 em memória conforme o `global_scope` cresce (é reconstruído a cada busca — ponto a observar, ver Decision 024).

Regra 7 do projeto pede benchmarks reais registrados; os números acima ainda não foram medidos formalmente e não devem ser assumidos — só preenchidos quando houver medição real.
