# CONSERPAV BIG DATA

Projeto desenvolvido nas disciplinas de Inteligência Artificial e Big Data e Ciência de Dados
(Análise e Desenvolvimento de Sistemas, UniBalsas Centro Universitário).

**Etapa atual:** TED 03 – Modelagem preditiva e Machine Learning (Documento Técnico v3.0).

| Etapa | Tema | Material |
|---|---|---|
| TED 01 | Prospecção, diagnóstico e seleção dos dados | `projeto/docs/TED01-BIG-DATA.pdf`, `projeto/docs/ted01-materia-IA.pdf`, `projeto/src/analise01.ipynb` |
| TED 02 | Limpeza, saneamento e análise exploratória | `projeto/docs/TED-02-BIG-DATA.pdf` |
| TED 03 | Modelagem preditiva e Machine Learning | `projeto/src/TED03.ipynb` e gráficos em `projeto/data/` |

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
| Atributos | 67 (55 após a limpeza da TED 02) |
| Formato | CSV (`projeto/data/base_de_dados.csv`, codificação latin-1) |
| Tabela de origem | `basedosdados.br_me_rais.microdados_vinculos` |

O arquivo do repositório já é o recorte do Maranhão. A limpeza da TED 02 (remoção de 12 colunas)
é reproduzida no início do notebook da TED 03, então não há um segundo CSV tratado.

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

**Opção 1 – Arquivo do repositório:** `projeto/data/base_de_dados.csv` já acompanha o projeto.

**Opção 2 – Download direto:** baixe o CSV pelo link do Drive acima e salve em
`projeto/data/base_de_dados.csv`.

**Opção 3 – Reproduzir a consulta no BigQuery:**

1. Acesse o console do BigQuery (https://console.cloud.google.com/bigquery) com uma conta Google
   e crie um projeto gratuito.
2. Execute a consulta abaixo:

```sql
SELECT *
FROM `basedosdados.br_me_rais.microdados_vinculos`
WHERE ano = 2023
  AND sigla_uf = 'MA'
  AND SUBSTR(cnae_2, 1, 2) IN ('41', '42', '43')
```

3. Exporte o resultado em CSV e salve em `projeto/data/base_de_dados.csv`.

**Opção 4 – Arquivos originais do MTE:** os microdados brutos também estão disponíveis
no FTP do PDET, pela página da fonte original indicada acima.

## Como executar

Requer Python 3.10 ou superior (testado com 3.13).

```bash
cd projeto
pip install -r requirements.txt
cd src
jupyter notebook TED03.ipynb
```

O notebook lê `../data/base_de_dados.csv`, treina os modelos e regrava os três gráficos em
`projeto/data/`. A execução completa leva cerca de 1 a 2 minutos. Todas as sementes
aleatórias usam o valor 42, então os resultados são reproduzíveis.

O notebook `projeto/src/analise01.ipynb` reproduz a análise preliminar da TED 01
(registros, nulos, duplicidades e estatísticas de remuneração e idade).

## Principais achados

**TED 01/02 (análise e limpeza):** 58 das 67 colunas sem nulos, 5 colunas totalmente vazias,
542 registros duplicados (0,55%; 543 após a remoção das colunas) e 11.840 vínculos (12,09%)
com remuneração média igual a zero. A TED 02 foi conservadora: removeu 12 colunas e nenhuma
linha. A remuneração é assimétrica à direita e varia muito por ocupação (mediana de R$ 1.394,80
para servente de obras a R$ 3.418,13 para mestre de construção civil) e por divisão CNAE.

**TED 03 (modelagem):** regressão para estimar `valor_remuneracao_media` com 8 variáveis
(ocupação, setor, sexo, porte do estabelecimento, tipo de vínculo, idade, horas contratadas e
tempo de emprego). Para o modelo, duplicidades e remunerações iguais a zero foram excluídas,
restando 85.755 registros (68.604 para treino e 17.151 para teste).

| Modelo | MAE (R$) | RMSE (R$) | R² |
|---|---|---|---|
| DummyRegressor (baseline) | 995,09 | 1.687,01 | 0,000 |
| LinearRegression | 634,03 | 1.222,34 | 0,475 |

A validação cruzada de 5 partes deu R² médio de 0,482 (desvio de 0,017), sem sinal de
sobreajuste relevante. O modelo erra mais nos salários altos (acima de R$ 5.000 o MAE é de
R$ 2.967,36). A análise completa está no notebook e no Documento Técnico v3.0.

## Limitações

A base registra vínculos formais e remuneração mensal média declarada anualmente. Ela não
possui identificação de obra ou de trabalhador, frequência diária, meia diária nem valor de
diária. Seus resultados servem como referência do setor no Maranhão, não como retrato das
obras da Conserpav. Por isso, o modelo da TED 03 é uma referência salarial por função e perfil,
e não uma previsão do custo de mão de obra da empresa, que dependerá do histórico gerado pelo
próprio sistema.

## Estrutura do repositório

```
CONSERPAV-BIG-DATA/
├── README.md
└── projeto/
    ├── requirements.txt
    ├── data/
    │   ├── base_de_dados.csv
    │   ├── grafico_comparacao_metricas.png
    │   ├── grafico_real_vs_previsto.png
    │   └── grafico_distribuicao_residuos.png
    ├── docs/
    │   ├── TED01-BIG-DATA.pdf
    │   ├── ted01-materia-IA.pdf
    │   └── TED-02-BIG-DATA.pdf
    └── src/
        ├── analise01.ipynb
        └── TED03.ipynb
```
