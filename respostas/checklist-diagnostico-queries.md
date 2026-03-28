# Checklist: Diagnóstico e Otimização de Queries Lentas

Ao identificar uma query com performance degradada (ex: execução > 30s), você deve seguir uma abordagem sistemática baseada em evidências, em vez de tentar "adivinhar" o problema. Siga este roteiro técnico:

## Passo 1: Rodar a query com EXPLAIN ANALYZE e BUFFERS
Nunca tente otimizar às cegas. O primeiro passo é prefixar a sua query com o comando:

```sql
EXPLAIN (ANALYZE, BUFFERS) 
SELECT ... -- sua query aqui
```

* **EXPLAIN:** Mostra o plano que o banco pretende seguir.
* **ANALYZE:** Executa a query de fato e exibe o tempo real gasto em cada etapa.
* **BUFFERS:** Mostra o uso de I/O (disco vs. memória), essencial para verificar se a query está lendo dados demais do disco (`read`) ou aproveitando o cache (`shared hit`).

## Passo 2: Ler o plano de execução (De baixo para cima)
O PostgreSQL executa o plano como uma árvore invertida:

* **Nós Internos:** Comece a ler a partir dos nós mais recuados (maior indentação), pois o nó pai só termina quando os filhos entregam os dados.
* **Trilha do Tempo:** Procure onde o `actual time` salta de milissegundos para segundos.
* **Loops:** O tempo em `actual time` é a média por loop. Multiplique sempre o tempo pelo número de loops para saber o gasto total naquele nó.

## Passo 3: Identificar "Red Flags" (Sinais de Alerta)
Analise as métricas buscando os seguintes padrões críticos:

* **Seq Scan em tabelas grandes:** Indica que o banco está lendo a tabela inteira linha por linha; geralmente falta um índice adequado.
* **Rows Removed by Filter:** Se o banco lê milhões de linhas para retornar apenas 5, há um desperdício massivo de CPU e disco.
* **Divergência de Estimativas:** Se `rows=10` mas `actual rows=10000`, as estatísticas estão desatualizadas, levando o banco a escolher estratégias ruins.
* **Sort Method: external merge Disk:** Significa que a operação estourou a memória RAM disponível (`work_mem`) e precisou usar o disco, destruindo a performance.

## Passo 4: Revisar a Estrutura da Query
Práticas de código que inviabilizam a performance:

* **Evite SELECT *:** Retorne apenas as colunas necessárias para reduzir o overhead de I/O.
* **SARGability (Filtros Otimizáveis):** Evite funções no `WHERE` (ex: `WHERE UPPER(nome)`). Isso invalida índices comuns, a menos que você use um **Índice de Expressão**.
* **Filtre cedo:** Reduza o volume de dados antes de realizar JOINs complexos ou agregações.

## Passo 5: Otimizar através de Índices
Se a lentidão ocorre por falta de acesso direto aos dados:

* **Índices Compostos:** Se usa múltiplas colunas no filtro, crie um índice composto. Lembre-se da regra do **leftmost prefix**: a ordem das colunas importa.
* **Alta Cardinalidade:** Coloque as colunas com maior diversidade de valores primeiro no índice.
* **Índices Parciais:** Se o filtro é previsível (ex: `WHERE ativo = true`), índices parciais são menores e muito mais rápidos.

## Passo 6: Manutenção e Tunning
* **Estatísticas:** Rode `VACUUM ANALYZE nome_da_tabela` para atualizar a contagem de registros e melhorar o planejamento do banco.
* **Memória:** Se houver paginação para disco, aumente o parâmetro `work_mem` para a sessão.
* **Criação em Produção:** Para novos índices, use sempre `CREATE INDEX CONCURRENTLY` para não bloquear as escritas na tabela. 
Após aplicar correções (seja criando um índice ou alterando o SQL), volte ao Passo 1. Execute novamente o EXPLAIN ANALYZE e compare os BUFFERS matematicamente (menos leituras no disco = sucesso) para garantir que a lentidão foi resolvida
