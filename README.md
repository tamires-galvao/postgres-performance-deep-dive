# Miniguia de Estudos: Performance e Indexação em PostgreSQL

Este repositório apresenta o resultado de um projeto prático desenvolvido para a plataforma **DIO**. O objetivo foi utilizar o **NotebookLM** como uma ferramenta de aprendizagem ativa para dominar conceitos avançados de otimização de bancos de dados.

## Contexto e Objetivos

O foco deste material é transitar do conhecimento sintático para a **decisão arquitetural**. O estudo explorou como o otimizador do PostgreSQL avalia custos de execução e como equilibrar a performance de leitura com o impacto nas operações de escrita.

## Curadoria de Fontes

A base de conhecimento foi alimentada por uma curadoria estratégica de vídeos e documentação técnica oficial:

### Documentação Técnica
* **EXPLAIN:** [Using EXPLAIN](https://www.postgresql.org/docs/current/using-explain.html)
* **Planner Constants:** [Query Planning Configuration](https://www.postgresql.org/docs/current/runtime-config-query.html)
* **Index Only Scans:** [Index-Only Scans and Covering Indexes](https://www.postgresql.org/docs/current/indexes-index-only-scans.html)
* **Guia Base:** [PostgreSQL Indexing Deep Dive](postgresql-indexing-deep-dive.md)

### Referências em Vídeo
* [PostgreSQL Indexing Explained](https://www.youtube.com/watch?v=YON9PliOYFk)
* [Query Optimization Tips](https://www.youtube.com/watch?v=QLb3nhIy2Lc)
* [Understanding Scan Types](https://www.youtube.com/watch?v=7Hy2mNuqdjk)

Outras fontes recomentadas pela função Deep Research foram usadas.
---

## Engenharia de Prompts e Aprendizado

Durante o desenvolvimento, a interação com a IA foi refinada para extrair detalhes técnicos que fujam do óbvio:

1. **Prompt Inicial:** "O que é um índice B-Tree?" (Resultado genérico sobre estruturas de dados).
2. **Refinamento Crítico:** "Por que o otimizador prefere um Seq Scan mesmo com um índice B-Tree disponível?"
   * **Insights:** Identificou-se que a baixa seletividade, a cardinalidade da coluna e a desorganização física dos dados no heap (correlação) são determinantes para essa escolha.
3. **Troubleshooting:** Para entender o `Bitmap Index Scan`, foi necessário solicitar uma comparação técnica focada na gestão de memória (`work_mem`) e na conversão de I/O aleatório em sequencial para volumes intermediários de dados.

---

## Miniguia de Estudo

### Resumo Estruturado
* **Otimizador:** Baseia-se em estimativas de custo e estatísticas coletadas pelo `ANALYZE`, não garantindo o uso de índices apenas por sua existência.
* **Tipos de Scan:**
    * **Index Scan:** Percorre a B-Tree e acessa o heap linha por linha via ponteiro; ideal para alta seletividade.
    * **Index Only Scan:** Otimização máxima onde o dado é retornado diretamente do índice, sem tocar no heap.
    * **Bitmap Scan:** Organiza a leitura de páginas do heap de forma sequencial após consultar o índice.
* **Índices Compostos:** A ordem das colunas é vital; o índice só é efetivo se a primeira coluna (*leftmost prefix*) for utilizada na cláusula WHERE.
* **Operação em Produção:** Utilização obrigatória de `CREATE INDEX CONCURRENTLY` para evitar o bloqueio de escritas (`ShareLock`) em tabelas ativas.

### Glossário Técnico
* **Seletividade:** A capacidade de um filtro reduzir o número de linhas retornadas; alta seletividade favorece o uso de índices.
* **BRIN (Block Range Index):** Índice minúsculo indicado para tabelas massivas de logs com correlação natural entre valor e posição física.
* **HOT (Heap Only Tuple) Updates:** Recurso que evita a atualização de índices quando a alteração ocorre em colunas não indexadas.

### Prompts de Revisão
* *"Analise este resultado de EXPLAIN ANALYZE e identifique por que o custo de I/O aleatório está penalizando a query."*
* *"Quais os riscos de manter índices redundantes em tabelas com alta taxa de INSERT e UPDATE?"*
* *["Descreva o checklist para diagnosticar e otimizar uma query lenta em produção."](https://github.com/tamires-galvao/postgres-performance-deep-dive/blob/main/respostas/checklist-diagnostico-queries.md)*

## 📁 Materiais Criados pelo NotbookLM 
- [PostgreSQL_Blazing_Fast.pdf](https://github.com/tamires-galvao/postgres-performance-deep-dive/blob/main/docs/PostgreSQL_Blazing_Fast.pdf)

---

**Licença:** [MIT](https://github.com/tamires-galvao/postgres-performance-deep-dive/blob/main/LICENSE)
