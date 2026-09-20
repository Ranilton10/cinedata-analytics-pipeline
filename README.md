# CineData Analytics

Projeto desenvolvido para a atividade de Engenharia de Dados do Rocket Lab 2026, utilizando Databricks, PySpark e SQL.

O objetivo foi criar um pipeline para organizar uma base de filmes do TMDB/IMDb, desde a leitura dos dados brutos até a criação de tabelas para análises. Para isso, utilizei a arquitetura Medalhão, dividida nas camadas Bronze, Silver e Gold.

## 1. Bronze — Ingestão dos dados

No notebook `Landing_to_Bronze.ipynb`, fiz a leitura dos cinco arquivos CSV disponibilizados na atividade e salvei os dados em tabelas Delta, utilizando o modo Append.

Mantive os dados no formato original e adicionei a coluna `ingestion_datetime` para registrar o momento da ingestão.

Também consultei a API do Banco Central para obter a cotação do dólar e salvei os dados na tabela `tb_cotacao_dolar`.

## 2. Silver — Tratamento dos dados

No notebook `Bronze_to_Silver.ipynb`, tratei os dados da Bronze para deixá-los prontos para as análises.

Os principais tratamentos foram:

- Padronização dos nomes das colunas, datas e status dos filmes.
- Remoção de registros duplicados e tratamento de valores ausentes ou inválidos.
- Conversão de orçamento e receita para valores numéricos, com cálculo de lucro e margem de lucro.
- Conversão dos valores financeiros de USD para BRL, utilizando a cotação mais recente disponível como referência.
- Tratamento das métricas de popularidade, notas e avaliações dos usuários.
- Separação dos gêneros, atores, diretores, roteiristas e produtoras em registros individuais.
- Preenchimento dos dias sem cotação com o último valor disponível.

Ao final, salvei os dados tratados em sete tabelas Delta na camada Silver.

## 3. Gold — Modelagem e análises

No notebook `Silver_to_Gold.ipynb`, organizei os dados em uma modelagem dimensional, utilizando tabelas fato, dimensões e tabelas-ponte.

A tabela `fact_movies_performance` reúne as informações financeiras e as métricas dos filmes. As dimensões armazenam os dados de filmes, gêneros, pessoas, produtoras e avaliações dos usuários. As tabelas-ponte relacionam os filmes aos seus gêneros, participantes e produtoras.

Também desenvolvi as seis análises solicitadas na atividade:

1. Receita total dos filmes em reais.
2. Cinco filmes com maior popularidade.
3. Quantidade de filmes por gênero.
4. Ranking dos dez filmes com maior receita em USD e BRL.
5. Ator com mais participações nos últimos dois anos.
6. Produtora com maior lucro nos últimos cinco anos.

Para as duas últimas análises, utilizei como referência a data de lançamento mais recente da base, desconsiderando filmes futuros e não lançados.

### Tabela para IA

Criei a tabela `gold_genai_movies_context`, que reúne título, ano de lançamento, receita, orçamento, atores, diretor e sinopse de cada filme em um texto.

Essa tabela foi preparada para servir como contexto para um assistente de IA. Nos casos em que alguma informação não estava disponível, utilizei textos substitutos para não perder o documento inteiro.

## 4. Workflow

Criei um Job no Databricks para executar os notebooks na seguinte ordem:

`Landing_to_Bronze` → `Bronze_to_Silver` → `Silver_to_Gold`

Configurei as dependências entre as tarefas e um agendamento diário. Após os testes, deixei o agendamento pausado para evitar execuções automáticas desnecessárias.

Executei o Workflow manualmente e as três tarefas terminaram com sucesso. A configuração do Job está no arquivo `job.yaml`.

### Execução do Workflow

![Execução bem-sucedida do Workflow](job_execucao_sucesso.png)

## 5. Arquivos do repositório

- `Landing_to_Bronze.ipynb`: ingestão dos arquivos CSV e da cotação do dólar.
- `Bronze_to_Silver.ipynb`: limpeza e tratamento dos dados.
- `Silver_to_Gold.ipynb`: modelagem dimensional, análises e tabela para IA.
- `job.yaml`: configuração do Workflow no Databricks.
- `job_execucao_sucesso.png`: imagem da execução bem-sucedida do Workflow.

## Observação

O agendamento diário do Workflow foi pausado após a execução bem-sucedida dos testes para evitar novas execuções automáticas desnecessárias e o consumo de recursos do ambiente Databricks.

O Job permanece configurado e pode ser executado manualmente. O arquivo `job.yaml` foi mantido no repositório para documentar sua configuração.

