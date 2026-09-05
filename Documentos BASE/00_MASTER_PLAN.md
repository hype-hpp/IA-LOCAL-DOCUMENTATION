# IA Local — Master Plan

## Objetivo

Construir um assistente local-first, agentic e multimodal, maximamente capaz dentro do hardware disponível.

### Capacidades desejadas

- Pesquisa profunda na Internet
- Análise de imagens, screenshots, PDFs e outros conteúdos visuais
- Escrita
- Memória persistente controlável pelo usuário
- RAG / base de conhecimento própria
- Busca híbrida e reranking
- Browser automation
- Crawler adaptativo e contínuo
- Coding agent
- Execução e debugging em sandbox
- Verificação de factualidade e contradições
- Model routing entre especialistas
- Uso de serviços cloud como fallback quando necessário

## Meta realista

Não buscar paridade literal com um modelo de fronteira hospedado.

A meta é maximizar a **capacidade prática e confiabilidade do sistema inteiro**, combinando modelos locais, ferramentas, memória, pesquisa, retrieval, sandbox e verificação.

## Hardware

- CPU: Ryzen 7 5800X3D
- GPU: RTX 4070 Ti SUPER
- VRAM: 16 GB
- RAM: 32 GB DDR4
- SSD: Samsung 990 PRO
- Espaço livre informado: ~404 GB
- OS: Arch Linux

## Stack-alvo

- Ollama
- Open WebUI
- Qdrant
- PostgreSQL
- Crawl4AI
- Playwright
- SearXNG
- Docker
- Agent Core próprio
- Model Router
- Verifier

## Modelos candidatos

- GPT-OSS 20B
- Qwen3-Coder 30B
- Qwen3.6 27B

## Regra de desenvolvimento

Nada deve ser implementado apenas por parecer interessante.

Cada fase precisa:
1. Ter objetivo claro.
2. Ter componentes mínimos necessários.
3. Ser testada no hardware real.
4. Terminar em estado funcional antes da próxima fase.
5. Registrar decisões e benchmarks.

## Estado

Este arquivo é a referência geral do projeto.
