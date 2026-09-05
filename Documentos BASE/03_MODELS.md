# IA Local — Models
## GPT-OSS 20B
- modelo geral
- agente principal
- tarefas rápidas
- fallback

### Benchmark de velocidade

| Contexto | Velocidade |
|---:|---:|
| 2K | 140 tok/s |
| 16K | 140 tok/s |
| 64K | 120 tok/s |

### Offload

- 100% GPU no teste informado
- Começou a estourar o espaço de VRAM somente em contexto maior

### Situação

**Forte candidato a modelo padrão por desempenho/latência.**

---

## Qwen3-Coder 30B
- coding
- debugging
- tarefas pesadas de software
- segunda opção de uso geral pesado

### Benchmark de velocidade

| Contexto | Velocidade |
|---:|---:|
| 2K | 82 tok/s |
| 16K | 72 tok/s |
| 64K | 45 tok/s |

### Arquitetura

- MoE
- ~30B parâmetros totais
- ~3.3B parâmetros ativos

### Offload

- Teste informado: ~27% GPU / 73% CPU

### Situação

**Forte candidato a especialista de código.**

---

## Qwen3.6 27B
- reasoning pesado
- tarefa excepcional em que qualidade pode justificar latência

### Benchmark de velocidade

| Contexto | Velocidade |
|---:|---:|
| 2K | 8.9 tok/s |
| 16K | 7.7 tok/s |
| 64K | 5.5 tok/s |

### Arquitetura

- denso
- ~27.8B parâmetros

### Offload

- teste informado: ~24% GPU / 76% CPU

### Situação

**Não recomendado como modelo padrão neste hardware.**
Uso potencial: tarefas raras de máxima qualidade.

---

## Qwen3-VL 8B

### Candidato a

- visão
- OCR
- análise de screenshots
- percepção visual

### Situação

Especialista visual opcional.

---

## Qwen3-Embedding 4B

### Candidato a

- embeddings de documentos
- embeddings de memória
- retrieval

### Situação

Candidato principal para embedding.

---

## DECISION 016 — Arquitetura de Duas IAs (Orquestrador + Especialista)

### Decisão
O sistema operará com uma arquitetura estrita de **"Orquestrador e Ferramentas"**:

1. **Orquestrador (GPT-OSS 20B)**: Responsável por entender o prompt, planejar, interagir com o usuário, formatar a resposta final e decidir **quando** chamar ferramentas externas.
2. **Especialista de Código (Qwen3-Coder 30B)**: Tratado como uma **ferramenta (Tool)**. O GPT-OSS invoca o Qwen-Coder **apenas** para gerar, depurar ou explicar blocos extensos de código. O Qwen-Coder recebe o prompt técnico, gera o código, e devolve para o GPT-OSS, que então formata a resposta final para o usuário.

### Fluxo
`Usuário -> GPT-OSS (decide) -> [se for código] -> Chama Tool Qwen-Coder -> Retorna código -> GPT-OSS (refina/formata) -> Usuário`

### Justificativa
- Evita conflito de roteamento (ex: perguntas de código com 50% de explicação em texto).
- Aproveita a velocidade do GPT-OSS para a conversação e a especialização (mesmo que marginal) do Qwen-Coder para sintaxe pura.
- Simplifica a lógica do Model Router.