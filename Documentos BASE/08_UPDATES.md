NAO VAI PARA O PROJETO FINAL

# IA Local — Novas Decisões (Pós-Revisão)

## Precedência

**Este documento tem preferência e sobrescreve quaisquer decisões conflitantes presentes nos arquivos `00` a `08`.**

As decisões abaixo foram tomadas após a revisão crítica da arquitetura, dos benchmarks (com os dados corrigidos) e da viabilidade prática no hardware declarado. Elas substituem orientações anteriores sobre roteamento de modelos, verificação, armazenamento, interface e pipelines de visão.

---

## DECISION 014 — Carregamento e Gerenciamento de Modelos (Sob Demanda)

### Decisão
O **GPT-OSS 20B** ficará **a maior parte do tempo residente na VRAM** (100% GPU), servindo como modelo orquestrador principal.

O **Qwen3-Coder 30B** e o **Qwen3.6 27B** **não** ficarão residentes na VRAM simultaneamente. Eles serão carregados **sob demanda** quando a tarefa específica assim exigir.

### Regras práticas
- O Qwen3-Coder será mantido **pré-carregado na RAM** (em segundo plano) para evitar o *cold start* (leitura do SSD) quando for ativado. A ativação move os pesos da RAM para a VRAM.
- O Qwen3.6 27B só será carregado se o usuário explicitamente solicitar "raciocínio profundo" ou se o agente identificar uma tarefa de altíssima complexidade.
- Ao final da tarefa especializada, o modelo é descarregado da VRAM (mantendo-se apenas na RAM, se possível) para liberar espaço para o GPT-OSS.

### Justificativa
- O GPT-OSS é o mais rápido (140 tok/s) e possui qualidade 4.0, sendo ideal para 95% das interações.
- Manter modelos pesados na VRAM o tempo todo causaria *thrashing* e degradaria a performance geral.
- A RAM de 32 GB é suficiente para manter o Qwen-Coder (MoE ~22 GB) residente em segundo plano, enquanto o GPT-OSS ocupa apenas a VRAM.

### Status
**DEFINITIVA** (Substitui parcialmente as DECISIONS 006, 007 e 008 no que tange à convivência simultânea).

---

## DECISION 015 — Verificador Universal (Sempre Ativo)

### Decisão
O Verifier será executado **em todas as respostas**, sem exceção.

### Implementação
- Não será utilizado o GPT-OSS ou qualquer modelo pesado para verificar.
- Será utilizado um **modelo pequeno e leve** (ex: `Qwen2.5-7B-Instruct` ou similar) rodando **exclusivamente na CPU**.
- Função do Verifier: classificar a resposta gerada em uma das categorias: **"VERDADEIRO"**, **"DUVIDOSO"** ou **"FALSO"**, com base no contexto disponível (histórico e documentos recuperados).

### Justificativa
- A verificação condicional (antiga DECISION 010) criava um paradoxo: para saber se a resposta é incerta, precisamos verificá-la.
- Rodar um modelo 7B na CPU adiciona latência marginal (já que o gargalo é a VRAM/GPU) e não compete por recursos com o GPT-OSS.
- Aumenta drasticamente a confiabilidade do sistema para o usuário final.

### Status
**DEFINITIVA** (Substitui a DECISION 010).

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

### Status
**DEFINITIVA** (Substitui a lógica de roteamento da DECISION 007).

---

## DECISION 017 — Validação e Correção dos Benchmarks

### Decisão
O benchmark de qualidade do **GPT-OSS 20B** é oficialmente corrigido para **média 4.00** (notas [4.0, 4.0]), e não 2.50.

### Critérios Definitivos para Escolha de Modelo
- **GPT-OSS 20B**: Qualidade 4.00 | Velocidade 140 tok/s | Uso 100% VRAM. → **Escolha definitiva para padrão.**
- **Qwen3-Coder 30B**: Qualidade 4.00 | Velocidade 82 tok/s | Uso 27% GPU / 73% CPU. → **Tool de código.**
- **Qwen3.6 27B**: Qualidade 4.50 | Velocidade 8.9 tok/s. → **Uso excepcional (pesquisa profunda).**

### Justificativa
Com a correção, o GPT-OSS não tem desvantagem qualitativa frente ao Qwen-Coder, mas possui **vantagem esmagadora em velocidade e uso de hardware**. Isso torna a decisão de torná-lo padrão matematicamente indiscutível.

### Status
**DEFINITIVA** (Corrige os dados da DECISION 006 e do arquivo `07_BENCHMARKS.md`).

---

## DECISION 018 — Sistema de Armazenamento Híbrido (Escopo do Chat vs. Escopo Global)

### Decisão
O sistema de arquivos e vetores será dividido em **dois níveis estanques** para otimizar espaço e respeitar a privacidade do usuário:

| Nível | Localização | Escopo no Qdrant | Ciclo de Vida |
| :--- | :--- | :--- | :--- |
| **Escopo do Chat (Local)** | `./chats/{chat_id}/` | `scope: "chat"` + `chat_id` | **Apagado** automaticamente quando o usuário deleta o chat. Utilizado para contexto imediato (evidências de pesquisa, PDFs da conversa, cache do crawler). |
| **Escopo Global (Memória)** | `./knowledge/` | `scope: "global"` | **Persistente** até que o usuário edite ou delete manualmente pela UI. Utilizado para RAG entre diferentes conversas e memória de longo prazo. |

### Mecanismo de Promoção
- O usuário deve clicar em um botão **"Salvar na memória"** ou digitar um comando (`/save`) para promover um conhecimento do escopo do chat para o escopo global.
- O agente pode *sugerir* a promoção, mas nunca a executa sem confirmação explícita.

### Justificativa
- Resolve o "furo" onde apagar arquivos imediatamente quebrava a memória futura.
- Garante que o SSD (~404 GB) não seja entupido por caches de chats antigos.
- Dá ao usuário controle total sobre o que vira "conhecimento permanente".

### Status
**DEFINITIVA** (Nova decisão, complementa as fases 02 e 05).

---

## DECISION 019 — Estratégia de Interface (UI) — Gradio/Streamlit Primeiro

### Decisão
O **Open WebUI** será **ignorado** durante as fases iniciais de desenvolvimento (Fases 02 a 07).

### Plano
1. **Fase de Desenvolvimento/Teste**: Utilizar **Gradio** ou **Streamlit** (Python puro) para criar uma interface minimalista de chat. Isso permite iterar rapidamente sobre o Agent Core, ferramentas e pipelines sem a complexidade de integração com o Open WebUI.
2. **Fase de Produção (Fase 08+)**: O Agent Core será exposto como uma **API no formato OpenAI** (`/v1/chat/completions`). Neste momento, faremos a integração com o Open WebUI via "Conexão OpenAI" nativa ou via Pipelines.

### Justificativa
- Velocidade de desenvolvimento: 30 linhas de Gradio resolvem o teste.
- Evita dor de cabeça com configuração de ferramentas visuais (Tool Calling) no Open WebUI durante a fase de prototipagem.
- O usuário declarou não ter domínio sobre a integração profunda do Open WebUI agora, então isso adia essa complexidade para o momento certo.

### Status
**DEFINITIVA** (Substitui a exigência imediata do Open WebUI na Fase 08, postergando-a).

---

## DECISION 020 — Ambiente de Execução (Docker Sandbox com Pacotes Pré-instalados)

### Decisão
O sandbox Docker utilizado para execução de código terá um ambiente **pré-configurado** com as bibliotecas mais comuns.

### Pacotes inclusos na imagem base
- `python:3.11-slim` como base.
- Bibliotecas: `numpy`, `pandas`, `requests`, `beautifulsoup4`, `matplotlib`, `scipy`.

### Regras
- O agente **não** pode executar `pip install` sem autorização explícita do usuário (para evitar ataques de supply chain ou pacotes gigantescos).
- Caso o código exija uma biblioteca exótica, o agente deve pedir permissão e, se autorizado, executar a instalação em um volume temporário que será descartado após a execução.
- Isolamento total: sem acesso ao `/home` do usuário, sem montagem de diretórios sensíveis.

### Justificativa
- Acelera a execução de scripts típicos (análise de dados, web scraping, automação).
- Evita falhas por falta de dependências básicas.

### Status
**DEFINITIVA** (Reforça a DECISION 005).

---

## DECISION 021 — Pipeline de Percepção Visual (Visão Antes do Texto)

### Decisão
O modelo visual **nunca** conversará diretamente com o usuário. Ele atuará estritamente como uma **camada de pré-processamento perceptual**.

### Fluxo Obrigatório
1. Usuário envia uma imagem, screenshot ou PDF (com imagens).
2. O arquivo é enviado para o **Qwen3-VL 8B** (ou OCR alternativo).
3. O modelo visual gera uma **descrição textual detalhada** do conteúdo e extrai todo o texto legível (OCR).
4. Este relatório textual é **injetado no prompt de sistema/contexto** do GPT-OSS.
5. O GPT-OSS processa o texto e responde ao usuário.

### Justificativa
- O GPT-OSS 20B **não é multimodal** nativamente. Tentar enviar imagens para ele resultaria em erro ou perda de informação.
- Centralizar a "percepção" em um modelo especializado (8B) e a "cognição" no GPT-OSS é a arquitetura mais eficiente para hardware local.

### Status
**DEFINITIVA** (Especifica a implementação da camada de Visão na arquitetura).

---

## Resumo do Impacto nos Documentos Anteriores

| Arquivo Afetado | Item Sobrescrito/Corrigido |
| :--- | :--- |
| `06_DECISIONS.md` | DECISION 006 (agora DEFINITIVA), DECISION 007 (agora TOOL), DECISION 010 (substituída pela 015). |
| `07_BENCHMARKS.md` | Média do GPT-OSS corrigida para 4.00. |
| `02_ARCHITECTURE.md` | Pipeline de visão, armazenamento em dois níveis e verifier na CPU são agora obrigatórios. |
| `05_ROADMAP.md` | Fase 08 ajustada para priorizar Gradio/Streamlit; Open WebUI fica para integração final. |
| `03_MODELS.md` | Roteamento definitivo (GPT-OSS orquestrador, Qwen-Coder tool). |