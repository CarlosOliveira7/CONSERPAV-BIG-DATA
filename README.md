# ANALISE DE DADOS

Projeto desenvolvido nas disciplinas de Inteligência Artificial e Big Data e Ciência de Dados
(Análise e Desenvolvimento de Sistemas, UniBalsas Centro Universitário).

**Etapa atual:** TED 01 – Definição do problema de dados e seleção do conjunto de dados.

## Integrantes

- Andrei Pereira Lima
- Carlos Oliveira Lopes
- Kaio Moreira Morais
- Priscila Ferreira Dias Santos
- Ywd Rhavell Ferreira Carvalho

## Descrição do problema

A Conserpav, empresa do setor de obras e pavimentação, controla a frequência e o fechamento
de pagamento dos trabalhadores de forma manual (cadernos, anotações e planilhas). Isso gera
erros de soma, perda de informações, retrabalho no fechamento da folha e dificuldade de
consultar períodos anteriores, além de dificultar a previsão dos custos de mão de obra.

A solução proposta é um sistema web para registro de frequência (diária completa, meia diária
e ausência), cálculo automático de pagamentos, histórico e previsão de custos com IA.

## Conjunto de dados utilizado

**RAIS – Relação Anual de Informações Sociais, microdados de vínculos, ano-base 2023.**

| Item | Descrição |
|---|---|
| Recorte | Maranhão (`sigla_uf = 'MA'`) e CNAE 2.0 divisões 41, 42 e 43 (construção) |
| Registros | 97.971 (de 1.265.961 no Maranhão e 82.966.522 no Brasil) |
| Atributos | 67 |
| Formato | CSV |
| Tabela de origem | `basedosdados.br_me_rais.microdados_vinculos` |

## Fonte dos dados

- **Fonte original (Ministério do Trabalho e Emprego):**
  https://www.gov.br/trabalho-e-emprego/pt-br/assuntos/estatisticas-trabalho/microdados-rais-e-caged
- **Acesso estruturado (Base dos Dados / BigQuery):**
  https://basedosdados.org/dataset/3e7c4d58-96ba-448e-b053-d385a829ef00?table=86b69f96-0bfe-45da-833b-6edc9a0af213
- **Recorte utilizado (Google Drive):**
  https://drive.google.com/file/d/1mK4umhCU6nULcQEdCCYs7DxAyQH38HB2/view

Os microdados da RAIS são dados públicos governamentais, não identificados, de livre acesso
para fins estatísticos e de pesquisa. O tratamento e a disponibilização em tabelas são feitos
pela Base dos Dados, cujas condições de uso constam na página do conjunto.

## Como obter os dados

**Opção 1 – Download direto:** baixe o CSV pelo link do Drive acima e salve em
`data/base_de_dados.csv`.

**Opção 2 – Reproduzir a consulta no BigQuery:**

1. Acesse o console do BigQuery (https://console.cloud.google.com/bigquery) com uma conta Google
   e crie um projeto gratuito.
2. Execute a consulta de `src/consulta.sql`:

```sql
   SELECT *
   FROM `basedosdados.br_me_rais.microdados_vinculos`
   WHERE ano = 2023
     AND sigla_uf = 'MA'
     AND SUBSTR(cnae_2, 1, 2) IN ('41', '42', '43')
```

3. Exporte o resultado em CSV e salve em `data/base_de_dados.csv`.

**Opção 3 – Arquivos originais do MTE:** os microdados brutos também estão disponíveis
no FTP do PDET, pela página da fonte original indicada acima.

## Análise preliminar

O script `src/analise_inicial.py` (ou notebook equivalente) carrega o CSV e calcula:
quantidade de registros e atributos, percentual de nulos por coluna, duplicidades,
estatísticas de remuneração e idade. Para executar:

```bash
pip install pandas
python src/analise_inicial.py
```

Principais achados (detalhados na seção 9 do documento): 58 das 67 colunas sem nulos,
5 colunas totalmente vazias, 542 registros duplicados (0,55%) e 11.840 vínculos
(12,09%) com remuneração média igual a zero.

## Limitações

A base registra vínculos formais e remuneração mensal média declarada anualmente. Ela não
possui identificação de obra ou de trabalhador, frequência diária, meia diária nem valor de
diária. Seus resultados servem como referência do setor no Maranhão, não como retrato das
obras da Conserpav.

## Estrutura do repositório
