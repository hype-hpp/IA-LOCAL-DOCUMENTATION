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