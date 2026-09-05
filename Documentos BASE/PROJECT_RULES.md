# IA Local — Project Rules

## Desenvolvimento

1. Não criar componentes sem necessidade.
2. Não criar scripts temporários quando um componente existente puder ser reutilizado.
3. Não assumir que uma instalação funcionou; testar.
4. Não avançar enquanto a fase atual estiver quebrada.
5. Não trocar a arquitetura por melhorias marginais sem benchmark.
6. Registrar decisões importantes.
7. Registrar benchmarks reais.
8. Preferir ferramentas open-source e local-first.
9. Cloud é fallback opcional, não requisito.
10. O usuário deve poder visualizar, editar e apagar memórias.
11. Toda ferramenta potencialmente destrutiva deve operar em sandbox.
12. O sistema deve ter observabilidade suficiente para diagnosticar falhas.
13. Limitar armazenamento, cache e crawling.
14. Evitar indexação de conteúdo irrelevante.
15. Não confundir capacidade do modelo com capacidade do sistema.
16. Não usar contexto gigantesco como substituto para retrieval.
17. O Verifier deve ser executado em todas as respostas (universal), rodando um modelo leve na CPU.
18. Toda mudança importante deve ser testada no hardware real.

## Estrutura de chats

Cada fase deve ter objetivo fechado e terminar com:
- estado funcional
- testes executados
- decisões registradas
- próximos passos claramente definidos
